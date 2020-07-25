In my previous post we looked at a simple bash backup script example, which would sync data between servers using "rsync" and then send us an email.

The only problem with this script is if "rsync" fails, we would still get a successful email.

Now I am sure you don't want to find out on the day you really need them that you don't have them :) .

To avoid such a catastrophe, we need our bash script to do the following

1. Test if "rsync" was successful.
2. If it was successful then send us a successful email.
3. Else send us an email informing us that the backup failed.

Lets add these conditions in our code.

[user@linux-vps ~]$vim backup
#!/bin/bash
#Author Leo G
rsync -avz -e "ssh " /path/to/yourfile user@backupserver.com:/backup/
if test $? -eq 0 
then
echo "backup for $(date) "| mail -s "backup complete" user@youremail.com
else
 echo "backup for $(date) "| mail -s "backup failed" user@youremail.com
fi

"if" bash keyword
Bash has a few built-in keywords to help you run commands based on conditions. "if" is one of them and it's syntax is as follows.

if condition 
then
do something
else
do something else
fi

Test command
"test" is a bash built-in command which will test our expression and return True or False

Exit Status of a command
"$?" is a special variable which stores the exit status of a command. If your command ran successfully then it will return "0" and this will be stored in "$?". The exit status ranges from 0 to 255, you can read more about it here

Bash Comparison Operators.
"-eq" is a comparison operator and compares the value of "$?" to 0.

We can use the following operators for numeric comparison.

Operator Description
-eq Is Equal To
-ne Is Not Equal To
-gt Is Greater Than
-ge Is Greater Than or Equal To
-lt Is Less Than
-le Is Less Than or Equal To

And the following for String or word comparison

== Is Equal To
!= Is Not Equal To
< Is Less Than
<= Is Less Than Or Equal To
> Is Greater Than if
>= Is Greater Than Or Equal To

Bash Scripting tutorial

BASH double brackets
I used the "test" command in this example along with the "if" statement for explanation purposes. The "if" statement in bash itself will check the exit status, so you do not need to use "test".

[user@linux-vps ~]$vim backup
#!/bin/bash
#Author Leo G
if rsync -avz -e "ssh " /path/to/yourfile user@backupserver.com:/backup/
then
echo "backup for $(date) "| mail -s "backup complete" user@youremail.com
else
 echo "backup for $(date) "| mail -s "backup failed" user@youremail.com
fi

You can also use "[" instead of test, "[[" will be the extended form which will allow you yo use logical operators like &&. You can read about it in detail here

Now, we may not be around when the backup fails, but we would still like to know why it did. So how do we tell our script to send us the error message?

In order for our bash script to tell us what went wrong, we need to redirect the error to a file and then attach the file or it's content in our email.

I/O redirection
In Linux everything is a file, so INPUT (STDIN), OUTPUT (STDOUT) and ERROR (STDERR) are also files and can be accessed via File Descriptors. File Descriptors are numeric(integer) values of these files.
The numeric value for STDIN is 0, STDOUT is 1 and STDERR is 2.

Let's add this redirection to our bash script.

[user@linux-vps ~]$vim backup
#!/bin/bash
#Author Leo G
if rsync -avz -e "ssh " /path/to/yourfile user@backupserver.com:/backup/ 2> /tmp/error.txt
then
echo "backup for $(date) "| mail -s "backup complete" user@youremail.com
else
 mail -s "backup for $(date) failed" user@youremail.com < /tmp/error.txt
fi

Bash scripting tutorials
There you go, a much more intelligent script, That will help us debug much faster in case of a failure.

For more advanced bash scripting I recommend your read text printing with the AWK command
