Monit and Config Server Firewall (CSF) are both simple and easy to use software for linux vps servers.
Monit not only sends you, email and jabber alerts but also provides you with a web interface from which you can store and graphically represent your data.

Monit Monitoring for linux vps hosting

Monit Monitoring for linux vps hosting

Installating and Configuring of Monit 5.5.

Ensure you have atomic and epel repositories installed to download the latest software versions.

The following steps are performed on a Centos 6 64bit OS with linux kernel version 2.6.32.

Step 1: Installation of Monit on Centos via yum


yum install monit
Step 2 : Configuration of Monit
Open the Monit configuration file


vim /etc/monit/monit.conf
Set the time interval to perform checks on services


set daemon  300  
Tell monit to send data to the mmonit server, replace "10.10.10.10" with your server IP address in the below config.


set eventqueue basedir /var/monit slots 1000
set mmonit http://admin:swordfish@10.10.10.10.com:8080/collector
set httpd port 2812 and use address localhost
allow localhost
allow 10.10.10.10
allow admin:swordfish
Set your email address where you would like to receive alerts and your sending email server, leave localhost to send from the same server.


set alert you@yourdomain.com
set mailserver localhost
The parameters are simple "if" statements, for eg "if loadavg (1min) > 4 then alert" means if load average for 1min is greater than 4 then send an alert, similarly for memory, swap and cpu. Set values as per your server resources.


  check system yourhostname.com
    if loadavg (1min) > 4 then alert
    if loadavg (5min) > 2 then alert
    if memory usage > 75% then alert
    if swap usage > 25% then alert
    if cpu usage (user) > 70% then alert
    if cpu usage (system) > 30% then alert
    if cpu usage (wait) > 20% then alert

Step 4 : Configure Services you want to monitor.

Monit monitor's services by checking the process id file, most pid files are in /var/run.

Mariadb or MySQL monit configuration


# File: /etc/monit.d/mysql
check process mysqld with pidfile /var/run/mysqld/mysqld.pid
group mysql
start program = "/etc/init.d/mysqld start"
stop program = "/etc/init.d/mysqld stop"
if failed host 127.0.0.1 port 3306 then restart
if 5 restarts within 5 cycles then timeout
Postfix monit configuration


# File: /etc/monit.d/postfix
check process postfix with pidfile /var/spool/postfix/pid/master.pid
   group mail
   start program = "/etc/init.d/postfix start"
   stop  program = "/etc/init.d/postfix stop"
   if failed port 25 protocol smtp then restart
   if 5 restarts within 5 cycles then timeout
Apache monit configuration


# File: /etc/monit.d/httpd
 check process apache with pidfile /var/run/httpd.pid
   group www
   start program = "/etc/init.d/httpd start"
   stop  program = "/etc/init.d/httpd stop"
   if failed port 80 then restart
   if 5 restarts within 5 cycles then timeout

PHP-FPM monit configuration

￼
check process php-fpm with pidfile /var/run/php-fpm/php-fpm.pid
group mysql
start program = "/etc/init.d/php-fpm start"
stop program = "/etc/init.d/php-fpm stop"
if failed host 127.0.0.1 port 9000 then restart
if 5 restarts within 5 cycles then timeout
Step 5 : Start Monit


service monit start
Installation and Configuration of M/Monit 3.2

Step 1 :Download and install M/Monit from http://mmonit.com/download/


cd /var/www/html
wget http://mmonit.com/dist/mmonit-3.2-linux-x64.tar.gz
tar -xzvf mmonit-3.2-linux-x64.tar.gz
cd mmonit-3.2
Step 2 : Configure M/Monit
Configuration files are stored in the conf folder and the main config file is server.xml. The default configuration should work fine without any further modification, however if you want a mariadb or mysql database to store data then modify the following parameters with your database details.

vim conf/server.xml
Realm url="mysql://dbusername:dbpassword@127.0.0.1:3306/dbname"
minConnections="5"
maxConnections="25"
reapConnections="300"
Optional : Incase you want to change the port number


Connector address="*" port="80" processors="10" 

Step 3 : Start the M/Monit Server


./bin/mmonit
Ensure you open port 8080 in your firewall.

Goto your hostname:8080 and type in admin for the username and swordfish for the password.

Monit should automatically send data to M/Monit no further configuration is required.

For troubleshooting M/Monit logs to logs/mmonit.log and logs/error.log.

You may also like to read about Config Server Firewall

http://www.iredmail.org/forum/topic565-iredmail-support-faq-how-to-monitor-mail-related-service-with-monit.html
