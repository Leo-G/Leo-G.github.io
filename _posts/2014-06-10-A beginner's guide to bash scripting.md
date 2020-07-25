A Brief Introduction
Bash or Bourne again shell is a replacement to the original unix shell written by Stephen Bourne at Bell Labs.

It offers vast improvements over the original shell, which include
Integer arithmetic,
Indexed arrays,
Command line editing,
Unlimited command history.

Bash scripts are available by default on most Linux distributions.

To find out which version of bash you are running type the following command.

[leo@bash101 ~]$ bash --version
bash --version
GNU bash, version 4.2.45(1)-release (x86_64-redhat-linux-gnu)
Copyright (C) 2011 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software; you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
In this bash tutorial you will learn how to write a simple bash script and then proceed to some intermediate bash scripting examples.

Getting Started with Bash scripting

Here's a simple bash script which will tell you the disk usage in the current directory.

[leo@bash101 ~]$vim disk_usage
#!/bin/bash
#Author Leo G
du -sh *
Assign execute permissions, using "chmod".

[leo@bash101 ~]chmod +x disk_usage
[leo@bash101 ~] bash disk_usage
8.0K    File1
4.0K    disk_usage
4.0k    Directory
Explanation of the disk usage bash script

Let's take each line of the script and understand how it work's.

#!/bin/bash : This line is called shebang and it indicates that the bash interpreter should be used to run the script. The bash interpreter will execute commands in our script by searching for them in the directories which contain executables.

To find the path to your bash interpreter type

[leo@bash101 ~]which bash
/usr/bin/bash
To find which PATHS your interpreter will search in, type

[leo@bash101 ~]echo $PATH
/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/s                                                                                        bin:/usr/local/rvm/bin:/home/leo_g/.local/bin:/home/leo_g/bin

#Author Leo G : "#" symbol indicates a comment. The exception to this is the first line "#!/bin/bash".

du -sh * : This line as you might have guessed contains the disk usage command for the current directories.

That's it, you can replace "du -sh *" with any command you like.

Debugging a bash script

Use "-x" to debug your bash scripts

[leo@linux-vps ~]$ bash -x disk_usage
+ du -sh File1 File2 Dire
8.0K    File1
4.0K    disk_usage
4.0k    Directory
I added a non existent bash command to the script, Now when you run the disk usage script with "-x" option you get:

[leolinux-vps ~]$ bash -x disk_usage
+ du -sh 1 disk_usage rails
8.0K    File1
4.0K    disk_usage
4.0k    Directory
+ nocommand
disk_usage: line 6: nocommand: command not found
Running multiple bash commands

You can run multiple commands by adding them on the next line

#!/bin/bash
#Author Leo G
du -sh *
ls -l
echo " This script is done executing and you now know how to write a basic bash script"

You need to ensure that you add them on the next line, else you will need to use a semi colon after each command.

Bash Script to read user input

Before we go into user input, we need to learn about variables.

A small brief on Bash variables
Bash variables are used to store data and then perform an action on this data.
There are different types of variables, Local variables, Global variables, String variables,Integer variables,Constant variables, Array variables.
Global variables or environment variables are available in all shells while local variables are available only in the current shell.

Type "env" to see environment variables.

There are pre-defined bash variables as well.

Examples of Pre-defined Bash Variables
To find your user id

[leo@linux-vps ~]$ echo $UID
1000

To find the path to your home directory

[leo_g@linux ~]$ echo $HOME
/home/leo_g

Here's something you should know about variables
"Variables are case sensitive and capitalized by default. Giving local variables a lowercase name is a
convention which is sometimes applied. However, you are free to use the names you want or to mix cases.
Variables can also contain digits, but a name starting with a digit is not allowed"

 
[leo@linux-vps ~]$vim disk_usage
#!/bin/bash
#Author Leo G
echo " Enter your directory: "
read x
du -sh "$x"
This script will prompt you for an input, enter the path to a file in the format "\path\file" and type enter.

In the above script the "read" command will store your input in the variable "x". Since we need to find the disk usage of the value stored inside x and not of x, we need to prefix x with a dollar sign.

[leo@linux-vps ~]$bash disk_usage
 Enter your directory:
/home/leo_g/
90M     /home/leo_g/

Note: I have enclosed "x" in double quotes so that the script does not throw an error when a file name has a space.

Let's take another practical Bash script example

Backup Bash Script.

In this example we will take a daily backup of our files and notify the user when the backup was successful.

[leo@linux-vps ~]$vim backup
#!/bin/bash
#Author Leo G
rsync -avz -e "ssh " /path/to/yourfile user@backupserver.com:/backup/
echo "backup for $(date) "| mail -s "backup complete" user@youremail.com

When this script will run, "rsync" will sync files to a remote backup server and then send an email to us.

Running scripts daily with cronjob

[leo@linux-vps ~]$crontab -e
00 1 * * * /path/to/backup
Note: There is no error check in the script and I know a common question that may have come to your mind, What if rsync fails?. Well we will tackle that issue with error codes in my next post.

I like to keep posts simple, with less content and more practical examples, if you have any suggestions feel free to comment.

Here are some other Bash Scripts and Guides you may find useful

Bash Real time file syncing daemon with inotify tools

https://github.com/Leo-G/backup-bash

How to use inotify-tools to trigger scripts on filesystem events

Creating Init/Systemd Scripts

How to create an init script

Building an RPM on CentOS

Blog post: Here is How I Built my First RPM

Bash If statements, Exit Status and Comparison Operators

 

For a list of Guides and Tutorials see https://github.com/Leo-G/DevopsWiki

source (http://www.tldp.org/LDP/Bash-Beginners-Guide/Bash-Beginners-Guide.pdf)
