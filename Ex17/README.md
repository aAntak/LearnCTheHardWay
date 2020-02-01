### Exercise 17: Heap And Stack Memory Allocation
#### Extra Credit

1. The die function needs to be augmented to let you pass the conn variable so it can close it and clean up.

```
	void die(const char *message, struct Connection *conn)
	{
		if(errno) {
			perror(message);
		} else {
			printf("ERROR: %s \n",message);
		}


		Database_close(conn);	
		exit(1);
	}
````
2. Change the code to accept parameters for MAX_DATA and MAX_ROWS , store them in the Database struct, and write that to the file, thus creating a database that can be arbitrarily sized.

````
#include <stdio.h>
#include <assert.h>
#include <stdlib.h>
#include <errno.h>
#include <string.h>


struct Address {
	int id;
	int set;
	char *name;
	char *email;
};

struct Database {
	int MAX_DATA2;
	int MAX_ROWS2;
	struct Address **rows;
};

struct Connection {
	FILE *file;
	struct Database *db;
};

void Database_close(struct Connection *conn);

void die(const char *message, struct Connection *conn)
{
	if(errno) {
		perror(message);
	} else {
		printf("ERROR: %s \n",message);
	}


	Database_close(conn);	
	exit(1);
}

void  Address_print(struct Address *addr)
{
	printf("%d %s %s \n",
			addr->id,addr->name,addr->email);
}

void Database_load(struct Connection *conn)
{
	assert(conn->db && conn->file);
	if(!(conn->db && conn->file))
		die("Database load : Invalid Connection info", conn);
        if(fread(&conn->db->MAX_DATA2, sizeof(conn->db->MAX_DATA2), 1, conn->file) != 1)
                die("Database read : failed to read MAX_DATA",conn);
        if(fread(&conn->db->MAX_ROWS2, sizeof(conn->db->MAX_ROWS2), 1, conn->file) !=1)
                die("Database read: failed to read MAX_ROWS",conn);
	
	conn->db->rows = (struct Address**)malloc(sizeof(struct Address *) * conn->db->MAX_ROWS2);
        int i = 0;
        for(i = 0; i < conn->db->MAX_ROWS2; i++) {
		
                conn->db->rows[i] = (struct Address*)malloc(sizeof(struct Address));
		struct Address *address = conn->db->rows[i];
		address->name = malloc(conn->db->MAX_DATA2);
		address->email = malloc(conn->db->MAX_DATA2);
                if(fread(&address->id, sizeof(address->id), 1, conn->file) != 1)
                        die("Database load : failed to read id",conn);
                if(fread(&address->set, sizeof(address->set), 1, conn->file) != 1)
                        die("Database load: failed to read set value", conn);
                if(fread(address->name, conn->db->MAX_DATA2, 1, conn->file) != 1)
			die("Database load: failed to read name", conn);
                if(fread(address->email, conn->db->MAX_DATA2, 1, conn->file) != 1)
                        die("Database load: failed to read email", conn);
		//printf("%s \n", conn->db->rows[i]->name);

        }
}

struct Connection *Database_open(const char *filename, char mode)
{
	struct Connection *conn = malloc(sizeof(struct Connection));
	if(!conn) die("Memory error",conn);

	conn->db = malloc(sizeof(struct Database));
	if(!conn->db) die("Memory error",conn);

	if(mode == 'c') {
		conn->file = fopen(filename, "w");
	} else {
		conn->file = fopen(filename, "r+");

		if(conn->file) {
			Database_load(conn);
		}
	}
	if(!conn->file) die("Failed to open the file",conn);

	return conn;
}

void Database_close(struct Connection *conn)
{
	if(conn) {
		if(conn->file) fclose(conn->file);
		if(conn->db) free(conn->db);
		free(conn);
	}
}

void Database_write(struct Connection *conn)
{
	rewind(conn->file);
	if(fwrite(&conn->db->MAX_DATA2, sizeof(conn->db->MAX_DATA2), 1, conn->file) != 1)
		die("Database write : failed to write MAX_DATA",conn);
	if(fwrite(&conn->db->MAX_ROWS2, sizeof(conn->db->MAX_ROWS2), 1, conn->file) !=1)
		die("Database write: failed to write MAX_ROWS",conn);

	printf("%d rows in write method \n",conn->db->MAX_ROWS2);	
	int i = 0;
	for(i = 0; i < conn->db->MAX_ROWS2; i++) {
		struct Address *address = conn->db->rows[i];
		
		if(fwrite(&address->id, sizeof(address->id), 1, conn->file) != 1)
			die("Database write : failed to write id",conn);
		if(fwrite(&address->set, sizeof(address->set), 1, conn->file) != 1)
			die("Database write: failed to wrte set value", conn);
		if(fwrite(address->name, conn->db->MAX_DATA2, 1, conn->file) != 1)
			die("Database write: failed to write name", conn);
		if(fwrite(address->email, conn->db->MAX_DATA2, 1, conn->file) != 1)
			die("Database write: failed to write email", conn);
	//	printf("%d ",address->id);

	}


	//int rc = fwrite(conn->db, sizeof(struct Database), 1, conn->file);
	//if(rc!=1) die("Failed to write database",conn);

	int rc = fflush(conn->file);
	if(rc == -1) die("Cannot flush database.",conn);
}

void Database_create(struct Connection *conn)
{
	int i = 0;
	conn->db->rows = (struct Address**)malloc(sizeof(struct Address*) * conn->db->MAX_ROWS2);
	printf("%d Max rows in create method\n", conn->db->MAX_ROWS2);
	//printf("%ld \n", sizeof(struct Address));

	for(i=0; i < conn->db->MAX_ROWS2; i++) {
	//make a prototype to initialize it
		conn->db->rows[i]=  malloc(sizeof(struct Address));
		assert(conn->db->rows[i] != NULL);
		conn->db->rows[i]->id = i;
		conn->db->rows[i]->set = 0;
		conn->db->rows[i]->name = (char *)malloc(conn->db->MAX_DATA2);
		conn->db->rows[i]->name = (char *)memset(conn->db->rows[i]->name, ' ', conn->db->MAX_DATA2);
		conn->db->rows[i]->email =(char *)malloc(conn->db->MAX_DATA2);
		conn->db->rows[i]->email = (char *)memset(conn->db->rows[i]->email, ' ', conn->db->MAX_DATA2);
	//then just assign it
		//conn->db->rows[i] = addr;
	}
}

void Database_set(struct Connection *conn, int id, const char *name, const char *email)
{
	struct Address *addr = conn->db->rows[id];
	if(addr->set) die("Already set, delete it first",conn);

	addr->set = 1;
	//Warning: bug, read the "How to break it and fix this
	char *res = strncpy(addr->name, name, conn->db->MAX_DATA2);
	printf("Name in set %s", addr->name);
	//demonstrate the strncpy bug
	addr->name[conn->db->MAX_DATA2-1] = '\0';
	addr->name[conn->db->MAX_DATA2-1] = '\0';
	if(!res) die("Name copy failed",conn);

	res = strncpy(addr->email, email, conn->db->MAX_DATA2);
	addr->email[conn->db->MAX_DATA2-1] = '\0';
	if(!res) die("Email copy failed",conn);
}

void Database_get(struct Connection *conn, int id)
{
	struct Address *addr = conn->db->rows[id];
	
	if(addr->set) {
		Address_print(addr);
	} else {
		die("ID is not set",conn);
	}
}

void Database_delete(struct Connection *conn, int id)
{
	struct Address *addr = conn->db->rows[id];

	addr->set = 0;
	memset(addr->name, ' ', conn->db->MAX_DATA2);
	addr->name[conn->db->MAX_DATA2-1] = '\0';
	memset(addr->email, ' ', conn->db->MAX_DATA2);
	addr->email[conn->db->MAX_DATA2-1] = '\0';

}

void Database_list(struct Connection *conn)
{
	int i = 0;
	struct Database *db = conn->db;

	for(i=0; i < conn->db->MAX_DATA2; i++){
		struct Address *cur = db->rows[i];
		if(cur->set){
			Address_print(cur);
		}
	}
}


int main(int argc, char *argv[])
{
	
	if(argc < 3) { printf("USAGE: ex17 <dbfile> <action> [action params] \n"); exit(1); }

	char *filename = argv[1];
	char action = argv[2][0];
	struct Connection *conn = Database_open(filename, action);
	int id = 0;
	if(argv[2][0]=='c') {
		if(argc!=5) {
			die("INVALID. Required format <C> <MAX_ROWS> <MAX_DATA> \n", conn);
		}
		conn->db->MAX_ROWS2 = atoi(argv[3]);
		conn->db->MAX_DATA2 = atoi(argv[4]);
		printf("MAX ROWS %d, MAX_DATA : %d\n", conn->db->MAX_ROWS2, conn->db->MAX_DATA2);
	}
	else {
	if(argc > 3) id = atoi(argv[3]);
	if(id >= conn->db->MAX_ROWS2) die("There's not that many records.",conn);
	}
	switch(action) {
		case 'c':
			Database_create(conn);
			Database_write(conn);
			break;
		
		case 'g':
			if(argc!=4) die("Need an id to get",conn);
			Database_get(conn,id);
			break;
		
		case 's':
			if(argc != 6) die("Need id, name, email to set",conn);
			Database_set(conn, id, argv[4], argv[5]);
			Database_write(conn);
			break;
		
		case 'd':
			if(argc!=4) die("Need id to delete",conn);
			Database_delete(conn,id);
			Database_write(conn);
			break;
		
		case 'l':
			Database_list(conn);
			break;	
		default:
			die("Invalid action, only: c=create, g=get, s=set, d=del, l=list",conn);		
	}

	Database_close(conn);

	return 0;
}
````
3. Add more operations you can do on the database, like find .
4. Read about how C does it's struct packing, and then try to see why your file is the size it is. See if you can calculate a new size after adding more fields.
5. Add some more fields to the Address and make them searchable.
6. Write a shell script that will do your testing automatically for you by running commands in the right order. Hint: Use set -e at the top of a bash to make it abort the whole script if any
command has an error.
7. Try reworking the program to use a single global for the database connection. How does this new version of the program compare to the other one?
8. Go research "stack data structure" and write one in your favorite
language, then try to do it in C.

``` 
CODE
````
