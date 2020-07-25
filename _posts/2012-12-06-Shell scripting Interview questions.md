Here are some examples of Shell Scripting interview questions and also shell scripting tips that may help you when you write shell scripts

How will you save the output of the TOP command in a file?

Use  Top in batch mode

 top -b -n1 > filename.txt
The -b option is for batch mode and the -n option is for the number of iterations in this case 1.

You can use the -d to option to to specify the time between two iterations

 top -b -n1 -d10 > filename.txt 
The delay is 10 seconds

How to print the nth line of a file?

ls -l | sed -n 2p 
will print the second line

ls -l | sed -n 2,3p
will print second and 3rd line

ls -l | sed -n -e '2,4p;20,30p'
can be used for several ranges.

How will you print n lines before and after a string matched by grep?

 grep -A 4 -B4 string filename
"-A" will print after the string
"-B" will print before the string.

Will keep adding to the post as I get the time, if you have anything to share then do let me know in the comments
