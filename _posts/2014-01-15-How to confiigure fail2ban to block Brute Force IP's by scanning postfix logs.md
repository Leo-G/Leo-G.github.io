Recently, I was looking at my postfix logs and found more than 13,000 Auth fail attempts and I was shocked. We had recently changed our user password due to a security breach, so I thought that some apps might have still not implemented the change, However after doing a reverse dig on the IP I saw it did not belong to us at all.
After digging around for solutions, I came across Fail2Ban. Fail2BAN is written in python and does log scanning to BLock IP's which match a particular IP address.

Installation is pretty easy you need to have epel repo, if you don't please add it via my script here.

yum install fail2ban
The main configuration is stored in "/etc/fail2ban/jail.conf" and the filters use to scan logs are stored in the "/etc/fail2ban/filter.d" directory.

Open jail.conf file and add or modify the below lines.

[postfix-tcpwrapper]
enabled = true
filter = postfix
action = hostsdeny[file=/etc/hosts.deny]
 sendmail[name=Postfix, dest=you@yourdomain.com]
logpath = /var/log/postfix.log
bantime = -1
Below are the parameters and their meaning.
Enabled : Enable the filter via boolean True or false
Action : What action is to be taken on regex match, here the IP will be blocked via tcpwrappers and an mail will be sent to me

Logpath: path to the log file
BanTime: the amount of time you want to ban the IP, the value should be in seconds like 300, negative value means permanent ban.

You also need to change the below values in the jail.conf file

ignoreip = 127.0.0.1/8
findtime  = 300
maxretry = 5
Ignoreip: space separated list of ip's to ignore, ensure you add your own :)
FindTime: The amount of time in which the connections are made post which the IP gets blocked.
Maxretry: Once 5 failed attempts are made the connection is blocked
Now add the below filter in /etc/fail2ban/filter.d/postfix.conf below any current filters

failregex = warning: (.*)[]: SASL LOGIN authentication failed:
This tells fail2ban to check this particular line and ban the host IP.

that's it

chkconfig on
servive fail2ban start
You should also read How to install Config Server Firewall and Login Failure daemon to stop ssh bruteforce
