Monitoring the system:

Basic command line:

pstree Processes and parent-child relationships
top Show top processes
iostat Report CPU statistics and input/output statistics for devices and partitions.
ps -auxw process status
uname -a print system information
cat /proc/version Display Linux kernel version in use.
cat /etc/redhat-release Display Red Hat Linux Release. (also /etc/issue)
uptime Tell how long the system has been running. Also number of users and system's load average.
w Show who is logged on and what they are doing.
/sbin/lsmod List all currently loaded kernel modules.
Same as cat /proc/modules
/sbin/runlevel Displays the system's current runlevel.
hostname Displays/changes the system's node name. (Must also manually change hostname setting in /etc/sysconfig/network. Command will change entry in /etc/hosts)
service Red Hat/Fedora command to display status of system services.
Example: service --status-all
Help: service --help

Process Management:

The basic Linux monitoring commands such as pstree and ps -auxw and top will inform you of the processes running on your system. Sometimes a process must be terminated. To terminate a process:

1. Identify the process:
* pstree -p
OR
* ps -auxw
OR
* top
2. Kill the process:
* kill 
* killall

This will perform an orderly shutdown of the process. If it hangs give a stronger signal with: kill -9 . This method is not as sanitary and thus less preferred.

A signal may be given to the process. The program must be programmed to handle the given signal. See /usr/include/bits/signum.h for a full list. For example, to restart a process after updating it's configuration file, issue the command kill -HUP

In the previous example, the HUP signal was sent to the process. The software was written to trap for the signal so that it could respond to it. If the software (command) is not written to respond to a particular signal, then the sending of the signal to the process is futile.

Identify all known signals: fuser -l

lsof - Processes attached to open files or open network ports:

The command lsof shows a list of processes attached to open files or network ports.

* List processes attached to a given file: lsof filename:

[root@test DIR]# lsof path to file

The process attached to an open file can be killed using the command fuser -ki filename

* List all open files on system: lsof
(Long list)

* List all files opened by user: lsof -u user-id

* The commands netstat -punta and socklist will list open network connections.
Use the command lsof -i TCP:port-number to see the processes attached to the port.
Example:

[root@test DIR]# lsof -i TCP:389

Restricting user resources:

* ulimit: (bash shell command)

Shell and process resources may be controlled and reported using the ulimit command. Display the limits of a shell using the bash command "ulimit -a". Limits can be set for the number of open files and processes, memory and virtual memory etc.

* See limits assigned in /etc/security

* Modify process scheduling priority: Range goes from -20 (highest priority) to 19 (lowest).
o Lower scheduling priority (runs slower and less likely to slow you down.)
nice -n 19 program-to-launch
Default for "nice -n" is 10
o Show default for any process: nice executable
Shows nice value to be used if run.
Memory Usage:

Linux Commands to Monitor Memory Usage:

vmstat Monitor virtual memory
free Display amount of free and used memory in the system. (Also: cat /proc/meminfo)
pmap Display/examine memory map and libraries (so). Usage: pmap pid
top Show top processes
sar -B Show statistics on page swapping.
time -v date Show system page size, page faults, etc of a process during execution. Note you must fully qualify the command as "/usr/bin/time" to avoid using the bash shell command "time".
cat /proc/sys/vm/freepages Display virtual memory "free pages".
One may increase/decrease this limit: echo 300 400 500 > /proc/sys/vm/freepages
cat /proc/meminfo Show memory size and usage

System crash and disk check upon boot:

Pertains to Red Hat 7.1 EXT2 filesystems and earlier which require an integrity check. (RH 7.2+ uses EXT3 which is a journaled file system which maintains file system integrity even with a crash.)

If the system crashes (due to power outage etc...) then upon boot the system will check if the disk was unmounted cleanly. If not you may get the following message:

Unexpected inconsistency; Run fsck Manually
...
*** An error occurred during the file system check.
*** Dropping you to a shell; the system will reboot
...
.
Give root password for maintenance
(or type Control-D for normal startup):

At this point enter the root password then run fsck:

(repair file system) 1# fsck -A -y
...
..
.
***** FILE SYSTEM WAS MODIFIED *****
...
..
.
(repair file system) 2# exit

The system will hopefully reboot properly at this point.
fsck man page
Note that fsck is NOT run against mounted file systems.

User Info:

Commands:

who Displays currently logged in users.
Use who -uH for idle time and terminal info.
users Show all users logged in.
w Displays currently logged in users and processes they are running.
whoami Displays user id.
groups Display groups you are part of.
Use groups user-id to display groups for a given user.
set Display all environment variables in your current environment.
id Display user and all group ids.
Use id user-id to display info for another user id.
last Listing of most recent logins by users. Show where from, date and time of login (ftp, ssh, ...) Also see lastlog command.
Show last 100 logins: last -100
history Shell command to display previously entered commands.

RPM - Redhat Package Manager:

The rpm command is used to manage software applications and system modules for Red Hat, Fedora, CentOS, Suse and many other Linux distributions.

Step One: Import Red Hat and Fedora GPG signature keys:

View your public key: (RHEL5)

[root@yoserver2 ~]# rpm -qa gpg-pubkey
gpg-pubkey-ed555983-3457f7f3
[root@yoserver2 ~]# rpm -qi gpg-pubkey-ed555983-3457f7f3
...
...

(Thus already installed. Your system is ready.)

Remove your public key: (RHEL5)

[root@yoserver2 ~]# rpm -e gpg-pubkey-ed555983-3457f7f3

Install public key: (Red Hat package up2date - now depricated. Use YUM.)

[root@yoserver2 ~]# rpm --import /usr/share/rhn/RPM-GPG-KEY
[root@yoserver2 ~]# rpm --import /usr/share/rhn/RPM-GPG-KEY-fedora

Do this once to configure RPM so that you won't constantly get the warning message that the signature is "NOKEY".
The purpose is to protect you from using a corrupt or hacked RPM.
Once these command are performed, you are ready to use the RPM command. (This is also required for the YUM commands below.)

YUM config file: /etc/yum.conf (Fedora Core 3)

[main]
cachedir=/var/cache/yum
debuglevel=2
logfile=/var/log/yum.log
pkgpolicy=newest
distroverpkg=redhat-release
tolerant=1
exactarch=1
retries=20
obsoletes=1
gpgcheck=1
exclude=firefox mozplugger gftp

# PUT YOUR REPOS HERE OR IN separate files named file.repo
# in /etc/yum.repos.d

You may list packages you wish NOT to update (Space delimited list. '*' wildcards allowed.): exclude=package-name
(i.e. On x86_64 I do not update firefox or mozplugger with the 64 bit version, I use the 32 bit version so that 32 bit plugins will work.)
Set "gpgcheck=0" to avoid the signature check.
For the option "gpgcheck=1" to work, use the "rpm --import GPG-KEY commands as detailed above in section one of the RPM tutorial.

[root@server2 ~]# rpm --import /usr/share/rhn/RPM-GPG-KEY
[root@server2 ~]# rpm --import /usr/share/rhn/RPM-GPG-KEY-fedora

File: /etc/yum.repos.d/fedora.repo (Fedora Core 3)

[base]
name=Fedora Core $releasever - $basearch - Base
#baseurl=http://download.fedora.redhat.com/pub/fedora/linux/core/$releasever/$basearch/os/
mirrorlist=http://fedora.redhat.com/download/mirrors/fedora-core-$releasever
enabled=1
gpgcheck=1

Mirror site/sites which contain Fedora base configuration RPM's.
Other protocols such as ftp can be used as well as http.

File: /etc/yum.repos.d/fedora-updates.repo (Fedora Core 3)

[updates-released]
name=Fedora Core $releasever - $basearch - Released Updates
#baseurl=http://download.fedora.redhat.com/pub/fedora/linux/core/updates/$releasever/$basearch/
mirrorlist=http://fedora.redhat.com/download/mirrors/updates-released-fc$releasever
enabled=1
gpgcheck=1

Mirror site which contain Fedora updated RPM's.

List of mirrors: http://fedora.redhat.com/download/mirrors.html

Terms:

* releasever: Release Version - current version of Fedora.
* basearch: Base Architecture - system hardware architecture i.e. i386

Add other repositories: (not included in default install)

* Fedora Extras:
Create file: /etc/yum.repos.d/extras.repo

[extras]
name=Fedora Extras $releasever - $basearch
baseurl=http://mirrors.kernel.org/fedora/extras/$releasever/$basearch/
http://www.mirrorservice.org/sites/download.fedora.redhat.com/pub/fedora/linux/extras/$releasever/$basearch/
http://fr2.rpmfind.net/linux/fedora/extras/$releasever/$basearch/ 
gpgcheck=1

Adding FreshRPM GPG signature key:

[root@server2 ~]# rpm --import http://download.fedora.redhat.com/pub/fedora/linux/extras/RPM-GPG-KEY-Fedora-Extras

* Add FreshRPM repository site to your list for downloads of non-standard Fedora software.
(Software not released by Red Hat like DVD players, audio encoders/rippers, etc)
Create file: /etc/yum.repos.d/freshrpms.repo

[freshrpms]
name=Fedora Linux $releasever - $basearch - freshrpms
baseurl=http://ayo.freshrpms.net/fedora/linux/$releasever/$basearch/freshrpms
enabled=0
gpgcheck=1

To directly enable a particular repository which is currently disabled (enabled=0): yum -y --enablerepo=freshrpms install kino

Adding FreshRPM GPG signature key:

[root@server2 ~]# rpm --import http://freshrpms.net/packages/builds/yum/RPM-GPG-KEY.freshrpms

More examples of FreshRPMs yum.conf

* Add: dag.wieers.com
Create file: /etc/yum.repos.d/dag.repo

[dag]
name=Dag APT Repository
baseurl=http://dag.freshrpms.net/fedora/$releasever/en/$basearch/dag/
http://dag.atrpms.net/fedora/$releasever/en/$basearch/dag/
http://ftp.heanet.ie/pub/freshrpms/pub/dag/fedora/$releasever/en/$basearch/dag/
enabled=0
gpgcheck=1

Directly enable repository: yum -y --enablerepo=dag install fortune-oneliners Adding dag GPG signature key:

[root@server2 ~]# rpm --import http://dag.wieers.com/packages/RPM-GPG-KEY.dag.txt

* Add: Macromedia.mplug.org
Create file: /etc/yum.repos.d/flash.repo

[flash]
name=Macromedia Flash plugin
baseurl=http://macromedia.mplug.org/apt/fedora/$releasever
http://sluglug.ucsc.edu/macromedia/apt/fedora/$releasever
http://ruslug.rutgers.edu/macromedia/apt/fedora/$releasever
http://macromedia.rediris.es/apt/fedora/$releasever
enabled=0
#gpgcheck=1

To directly enable a particular repository which is currently disabled (enabled=0): yum -y --enablerepo=flash install flash-plugin

Using YUM and YUM examples:

* Update:
o List packages which will be updated: yum check-update
(Does not perform an update)
o Update all packages on your system: yum update
o Update a package: yum update package-name
o Update all with same prefix: yum update package-name-prefix*
This command will update your system. It will interactively ask permission. i.e. "Is this ok [y/N]:"
o To avoid the prompt/questions use the command: yum -y
# To install a single package: yum -y install package-name
This will also resolve package dependencies.
# Remove a package: yum remove package-name
# Info:

* List available packages, version and state (base, installed, updates-released): yum list
* List the packages installed which are not available in repository listed in config file: yum list extras
* List packages which are obsoleted by packages in yum repository: yum list obsoletes

# Clean local cache of headers and RPM's: yum clean all
(See: /var/cache/yum/)

Form of command: find path operators

Examples:

* Search and list all files from current directory and down for the string ABC:
find ./ -name "*" -exec grep -H ABC {} ;
find ./ -type f -print | xargs grep -H "ABC" /dev/null
egrep -r ABC *
* Find all files of a given type from current directory on down:
find ./ -name "*.conf" -print
* Find all user files larger than 5Mb:
find /home -size +5000000c -print
* Find all files owned by a user (defined by user id number. see /etc/passwd) on the system: (could take a very long time)
find / -user 501 -print
* Find all files created or updated in the last five minutes: (Great for finding effects of make install)
find / -cmin -5
* Find all users in group 20 and change them to group 102: (execute as root)
find / -group 20 -exec chown :102 {} ;
* Find all suid and setgid executables:
find / ( -perm -4000 -o -perm -2000 ) -type f -exec ls -ldb {} ;
find / -type f -perm +6000 -ls

Note: suid executable binaries are programs which switch to root privileges to perform their tasks. These are created by applying a "sticky" bit: chmod +s. These programs should be watched as they are often the first point of entry for hackers. Thus it is prudent to run this command and remove the "sticky" bits from executables which either won't be used or are not required by users. chmod -s filename
* Find all world writable directories:
find / -perm -0002 -type d -print
* Find all world writable files:
find / -perm -0002 -type f -print
find / -perm -2 ! -type l -ls
* Find files with no user:
find / -nouser -o -nogroup -print
* Find files modified in the last two days:
find / -mtime 2 -o -ctime 2
* Compare two drives to see if all files are identical:
find / -path /proc -prune -o -path /new-disk -prune -o -xtype f -exec cmp {} /new-disk{} ;

Finding/Locating files:

locate/slocate Find location/list of files which contain a given partial name
which Find executable file location of command given. Command must be in path.
whereis Find executable file location of command given and related files
rpm -qf file Display name of RPM package from which the file was installed.

Note: The script /etc/cron.daily/updatedb.cron generates the index for the locate command. It will generate the database /var/lib/locatedb

File Information/Status/Ownership/Security:

ls List directory contents. List file information
chmod Change file access permissions
chmod ugo+rwx file-name :Change file security so that the user, group and all others have read, write and execute privileges.
chmod go-wx file-name :Remove file access so that the group and all others have write and execute privileges revoked/removed.
chown Change file owner and group
chown root.root file-name :Make file owned by root. Group assignment is also root.
fuser Identify processes using files or sockets
If you ever get the message: error: cannot get exclusive lock
then you may need to kill a process that has the file locked. Either terminate the process through the application interface or using the fuser command: fuser -k file-name
file Identify file type.
file file-name
Uses /usr/share/magic, /usr/share/magic.mime for file signatures to identify file type. The file extension is NOT used.

CRON - Scheduling a re-occurring task:

Add shell script to have run hourly, daily, weekly or monthly into the appropriate directory:

* /etc/cron.hourly/
* /etc/cron.daily/
* /etc/cron.weekly/
* /etc/cron.monthly/

These are preconfigured schedules. To assign a very specific schedule add a line to the /etc/crontab file. Cron entries may also be added to a crontab formatted file located in the directory /var/spool/cron/.
