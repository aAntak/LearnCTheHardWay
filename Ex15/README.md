### Exercise 15: Pointers Dreaded Pointers
#### Extra Credit

##### Rewrite all the array usage in this program so that it's pointers.
##### Rewrite all the pointer usage so they're arrays.
##### Go back to some of the other programs that use arrays and try to use pointers instead.
##### Process command line arguments using just pointers similar to how you did names in this one.
##### Play with combinations of getting the value of and the address of things.
##### Add another for-loop at the end that prints out the addresses these pointers are using. You'll need the %p format for printf .
##### Rewrite this program to use a function for each of the ways you're printing out things. Try to pass pointers to these functions so they work on the data. Remember you can declare a function to accept a pointer, but just use it like an array.
##### Change the for-loops to while-loops and see what works better for which kind of pointer usage.

``` #include <stdio.h>

int main(int argc, char *argv[])
{


	char **ptr;

	for(ptr=argv;(ptr-argv)<argc;ptr++) {
		printf("%p : %s\n",ptr,*ptr);
	}
	int ages[] = {23, 43, 12, 89, 2};
	char *names[] = {
		"Alan", "Frank",
		"Mary", "John", "Lisa"
	};

	int count = sizeof(ages) / sizeof(int);
	int i = 0;
        
	for(i = 0; i < count; i++) {
		printf("%s has %d years alive. \n",
				names[i], ages[i]);
	}

	printf("---\n");

	int *cur_age = ages;
	char **cur_name = names;

	for(i = 0; i < count; i++) {
		printf("%p : %s is %d years old. \n",
				cur_name+i,*(cur_name+i), *(cur_age + i));
	}

	printf("---\n");

	for(i = 0; i < count; i++) {
		printf("%s is %d years old again. \n",
				cur_name[i], cur_age[i]);
	}
	printf("---\n");

	for(cur_name = names, cur_age = ages; (cur_age - ages) < count; cur_name++,cur_age++) {
	printf("%s lived %d years so far. \n", *cur_name, *cur_age);
	}

	return 0;
}
````
