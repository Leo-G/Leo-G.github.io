In my  last Shell Scripting tutorial, I used "inotifywait" along with rsync to create a real time file syncing script called backup.

This script will run till I manually kill it or till I close my terminal, if I run it as a background process. Now I cannot keep my terminal open forever, but I want my script to run as long as possible so that my files are always in sync.

One way to let the script run even after you close your terminal is by using the “nohup “ (no hangup) command as follows:

nohup ./backup-script &

You still need to find the process id to stop the script when you log back in though. Instead of going through the hassle, we can create an init script and start or stop our backup daemon whenever we need to.

A skeleton Sysv init script is available at “/usr/share/doc/initscripts-9.03.40/sysvinitfiles” in CentOS 6.5

#!/bin/sh
#
#  
#
# chkconfig:     
# description: <description, split multiple lines with \
#              a backslash>
### BEGIN INIT INFO
# Provides: 
# Required-Start: 
# Required-Stop: 
# Should-Start: 
# Should-Stop: 
# Default-Start: 
# Default-Stop: 
# Short-Description: 
# Description: 
### END INIT INFO
# Source function library.
. /etc/rc.d/init.d/functions
exec="/path/to/"
prog=""
config=""
[ -e /etc/sysconfig/$prog ] && . /etc/sysconfig/$prog
lockfile=/var/lock/subsys/$prog
start() {
 [ -x $exec ] || exit 5
 [ -f $config ] || exit 6
 echo -n $"Starting $prog: "
 # if not running, start it up here, usually something like "daemon $exec"
 retval=$?
 echo
 [ $retval -eq 0 ] && touch $lockfile
 return $retval
}
stop() {
 echo -n $"Stopping $prog: "
 # stop it here, often "killproc $prog"
 retval=$?
 echo
 [ $retval -eq 0 ] && rm -f $lockfile
 return $retval
}
restart() {
 stop
 start
}
reload() {
 restart
}
force_reload() {
 restart
}
rh_status() {
 # run checks to determine if the service is running or use generic status
 status $prog
}
rh_status_q() {
 rh_status >/dev/null 2>&1
}
case "$1" in
 start)
 rh_status_q && exit 0
 $1
 ;;
 stop)
 rh_status_q || exit 0
 $1
 ;;
 restart)
 $1
 ;;
 reload)
 rh_status_q || exit 7
 $1
 ;;
 force-reload)
 force_reload
 ;;
 status)
 rh_status
 ;;
 condrestart|try-restart)
 rh_status_q || exit 0
 restart
 ;;
 *)
 echo $"Usage: $0 {start|stop|status|restart|condrestart|try-restart|reload|force-reload}"
 exit 2
esac
exit $?

Let’s start by defining our variables:

#path to our backup script
exec="/path/to/backup"
#name of our script
prog="backup"

The file etc/init.d/functions has a set of functions like daemon and killproc which we will use in our case statement to start and stop our script.

You need to define your start and stop functions, which will basically start your script(here you can use the daemon function) and kill it (killproc). A status function is already available, all you need to do is call the function with the name of your script ($prog).

#!/bin/bash
#Author Leo G
# backup daemon
# chkconfig: 2345 20 80
#The above indicates that the script should be started in levels 2, 3, 4, and 5, #that its start priority should be 20, and that its stop priority should be 80. 
# description: syncs two files
# Source function library.
. /etc/rc.d/init.d/functions
#Define variables
exec="/path/to/backup"
prog="backup"
start() {
 [ -x $exec ] || exit 5
 echo -n $"Starting $prog: "
 # use daemon to start the service 
 daemon $exec &
 retval=$?
 echo
 return $retval
}
stop() {
 echo -n $"Stopping $prog: "
 # use kill proc to stop the service
 killproc $prog -TERM
 retval=$?
 echo
 [ $retval -eq 0 ] && rm -f $lockfile
 return $retval
}
restart() {
 stop
 start
}
reload() {
 restart
}
force_reload() {
force_reload() {
 restart
}
rh_status() {
 # run checks to determine if the service is running or use generic status
 status $prog
}
rh_status_q() {
 rh_status >/dev/null 2>&1
}
case "$1" in
 start)
 #Only if not running, start
 rh_status_q && echo "process already started" && exit 0
 $1
 ;;
 stop)
 # Only if service is running stopt
 rh_status_q || exit 0
 $1
 ;;
 restart)
 $1
 ;;
 reload)
 rh_status_q || exit 7
 $1
 ;;
 force-reload)
 force_reload
 ;;
 status)
 rh_status
 ;;
 condrestart|try-restart)
 rh_status_q || exit 0
 restart
 ;;
 *)
 echo $"Usage: $0 {start|stop|status|restart|condrestart|try-restart|reload|force-reload}"
 exit 2
esac
exit $?
Test the script:

[leo@linux-vps]$ ./initscript.bash start
Starting backup:  OK
Check the status of our backup daemon.

[leo@linux-vps]$ ./initscript.bash status
backup (pid 2023) is running...
Alright, lets stop it.

[leo@linux-vps]$ ./initscript.bash stop
Stopping backup:    OK
You can move it to /etc/ini.d/yourscriptname, and use it like any other initscript along with service.

Now an even better way to do this with lesser code is Systemd. Systemd is the new system and service manager for Linux, compatible with SysV init scripts. Besides less code it also has many other notable features like starting processes in parallel ( currently they start one after the other ) which speeds up boot time, you can read about at http://www.freedesktop.org/wiki/Software/systemd/.

Systemd is available on CentOS 7.

You need to add the following service script to "/etc/systemd/system/backup.service"

[Unit]
Description=Backup daemon
[Service]
Type=simple
ExecStart=/path/to/backup
[Install]
WantedBy=multi-user.target
In the Unit section, I have added the description for the script.

In the service section, I have defined the executable i want to run with "ExecStart" and used "type=simple" since my process does not fork another service.

In the install section I added "WantedBy=multi-user.target", This indicates the runlevel, multi-user.target is runlevel 3.

I recommend you read http://0pointer.de/blog/projects/systemd-for-admins-3.html for a more detailed information, and any suggestions are welcome.

Reload systemd

[leo@linux-vps]$ systemctl daemon-reload
Activate the service

[leo@linux-vps]$ systemctl start backup.service
Check the status

[leo@linux-vps]$systemctl status backup.service. 
stop the service via systemctl

[leo@linux-vps]$ systemctl stop backup.service. 
Enable service at boot

 [leo@linux-vps]$systemctl enable backup.service.
I have also used trap as follows in the backup script itself to ensure that all child processes are killed. You may want to modify your script accordingly.

trap 'kill -HUP 0' EXIT

If you are a system administrator then I recommend you to read the Devops Wiki at https://github.com/Leo-G/DevopsWiki for a list of Linux programming and system administration guides.

Another good article that will help you on CentOS is How to Build an RPM.

As always feedback and suggestions are welcome.

source http://fedoraproject.org/wiki/Packaging:SysVInitScript
http://0pointer.de/blog/projects/systemd-for-admins-3.html
images are not mine
