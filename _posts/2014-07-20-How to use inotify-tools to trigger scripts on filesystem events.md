In my last example of Bash if statements we created a backup script which would run "rsync" as per the time set in a cron.

But wouldn't it be nice if we could run our "rsync" script only if there was a change to the files in our directory instead of running a cron every hour despite no change? With inotify-tools you can do just that.

What is inotify-tools?
Inotify tools are a set of command line programs based on inotify a Linux kernel (2.6.13 or later) feature which can be used to monitor filesystem events.

Installing inotify-tools
Software versions :

inotify-tools.x86_64 3.14-1.el6
CentOS 6.5
Linux kernel 2.6.32-042stab085.20

#Add the epel repo
[leo@linux-vps ~]$sudo rpm -Uvh http://download.fedoraproject.org/pub/epel/6/i386/epel-release-6-8.noarch.rpm
#Install inotify-tools
[leo_g@vps ~]$ sudo yum install inotify-tools

Inotify-tools has two commands

Inotifywait
This command simply blocks for inotify events, making it appropriate for use in shell scripts. It can watch any set of files and directories, and can recursively watch entire directory trees.

inotifywatch
inotifywatch collects filesystem usage statistics and outputs counts of each inotify event.

We will use inotifywait, since we do not need statistics.

inotifywait syntax

inotifywait filename

inotifywait example

[leo@linux-vps ~]$ inotifywait /tmp
Setting up watches.
Watches established.
/tmp/ MODIFY test

As you can see in the above example an event (in this case a "modify action" was performed on a file named "test" inside /tmp) triggered the output.

Now "inotifywait" by default checks for all events including if a file was opened but not written to, Since we only want "rsync" to trigger on change events like when a file is modified, we will need to specify the "-e" flag along with the list of events we want to be notified about.

[leo@linux-vps ~]$ inotifywait -m -r -e \ modify,attrib,close_write,move,create,delete /tmp
Setting up watches.  Beware: since -r was given, this may take a while!
Watches established.
/tmp/ MODIFY a
/tmp/ CLOSE_WRITE,CLOSE a

The -m flag is for continuous monitoring as by default inotifywait will exit on the first event and -r means recursively or check through sub-directories as well.

List of inotifywait events 
access
A watched file or a file within a watched directory was read from.
modify
A watched file or a file within a watched directory was written to.
attrib
The metadata of a watched file or a file within a watched directory was modified. This includes timestamps, file permissions, extended attributes etc.
close_write
A watched file or a file within a watched directory was closed, after being opened in writeable mode. This does not necessarily imply the file was written to.
close_nowrite
A watched file or a file within a watched directory was closed, after being opened in read-only mode.
close
A watched file or a file within a watched directory was closed, regardless of how it was opened. Note that this is actually implemented simply by listening for both close_write and close_nowrite, hence all close events received will be output as one of these, not CLOSE.
open
A watched file or a file within a watched directory was opened.
moved_to
A file or directory was moved into a watched directory. This event occurs even if the file is simply moved from and to the same directory.
moved_from
A file or directory was moved from a watched directory. This event occurs even if the file is simply moved from and to the same directory.
move
A file or directory was moved from or to a watched directory. Note that this is actually implemented simply by listening for both moved_to and moved_from, hence all close events received will be output as one or both of these, not MOVE.
move_self
A watched file or directory was moved. After this event, the file or directory is no longer being watched.
create
A file or directory was created within a watched directory.
delete
A file or directory within a watched directory was deleted.
delete_self
A watched file or directory was deleted. After this event the file or directory is no longer being watched. Note that this event can occur even if it is not explicitly being listened for.
unmount
The filesystem on which a watched file or directory resides was unmounted. After this event the file or directory is no longer being watched. Note that this event can occur even if it is not explicitly being listened to.

Now let's use inotifywait with our script.

[leo@linux-vps~]$ vim inotify-example
while true #run indefinitely
do 
inotifywait -r -e modify,attrib,close_write,move,create,delete /dir && /bin/bash backup-script
done

Since we want to continuously monitor changes, we use an infinite while loop and the Logic "&&" operator will ensure that our backup script is only triggered on a successful completion of the inotifywait event

[leo_g@vps scripts]$bash inotify-example 
+ true
+ inotifywait -r -e modify,attrib,close_write,move,create,delete /
Setting up watches.  Beware: since -r was given, this may take a while!
Watches established.
/dir DELETE a
+ /bin/bash backup-script 
+ rsync -avz  -e “ssh ” /path/to/yourfile user@backupserver.com:/backup/ 2> \ /tmp/error.txt
+ mail -s “backup complete” user@youremail.com
+ echo “backup for $(date) “
+ true
+ inotifywait -r -e modify,attrib,close_write,move,create,delete /dir
Setting up watches.  Beware: since -r was given, this may take a while!
Watches established.

If you have a suggestion feel free to let me know. The complete source code to the file syncing daemon with rsync and inotify can be found at https://github.com/Leo-g/backup-bash

You mat also like How to display a changing output like top

source https://github.com/rvoicilas/inotify-tools/wiki
