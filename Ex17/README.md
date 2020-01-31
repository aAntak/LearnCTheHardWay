### Exercise 17: Heap And Stack Memory Allocation
#### Extra Credit

1. The die function needs to be augmented to let you pass the conn variable so it can close it and clean up.
2. Change the code to accept parameters for MAX_DATA and MAX_ROWS , store them in the Database struct, and write that to the file, thus creating a database that can be arbitrarily sized.
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
