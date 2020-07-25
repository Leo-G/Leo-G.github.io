To improve the performance of my 512MB Linux VPS, I decided to use Lighttpd for my  web server instead of Apache. The reason for this change was that Lighttpd consumed less resources, was faster and supported Caching.

Before you start the install please ensure you download and install the repos via this shell script. Versions Centos 6 64 bit Lighttpd 1.4.35.1 Mariadb 5.5.36 PHP 5.4.27 Let's start

yum install lighttpd lighttpd-fastcgi php php-fpm mariadb-server php-mysql postfix -y

Open lighttpd configuration file

vim /etc/lighttpd/lighttpd.conf
 
$HTTP["host"] =~ “(^|.)leog.in” {
server.document-root = “/home/leog.in”
server.errorlog = “/var/log/lighttpd/leog.in-error.log”
accesslog.filename = “/var/log/lighttpd/leog.in-access.log”
server.error-handler-404 = “/404.php”
}

where, $HTTP["host"] =~ “(^|.)leog.in” :- will match for both www.leog.in and leog.in server.document-root = “/home/leog.in” :- Is the directory that contains your web files, ensure you have an index.html file in this directory with some test code. server.errorlog = “/var/log/lighttpd/leog.in-error.log” :- Site error Log file accesslog.filename = “/var/log/lighttpd/leog.in-access.log” :- Site access log file server.error-handler-404 = “/404.php” :- page not found customised page if you like Also make the following changes to avoid errors.

server.use-ipv6 = "disable"
server.max-fds = 2048

Next open up the modules file

vim /etc/lighttpd/modules.conf
And uncomment this line:
##
include "conf.d/fastcgi.conf"
##

Open fastcgi.conf

vim etc/lighttpd/conf.d/fastcgi.conf
and add the following


fastcgi.server += ( ".php" =>
 ((
 "host" => "127.0.0.1",
 "port" => "9000",
 "broken-scriptfilename" => "enable"
 ))
)
Create a user

useradd leog.in
This will also create the home directory /home/leog.in. The reason it is better to use a separate user for your site is to avoid a server wide compromise incase of a hack. open the php-fpm configuration file and add the user created.

vim /etc/php-fpm.conf
user = leog.in
; RPM: Keep a group allowed to write in log dir.
group = leog.in

service lighttpd start service php-fpm start create phpinfo page to test out the installation.

phpinfo (); ?> Mariadb Run the mysql_secure_installation to create a root password and disable remote root login as well as remove the test database.

service mysqld start
/usr/bin/mysql_secure_installation

Tweaking with PHP APC opcode cache and MySQL Query cache PHP APC install

yum install php-pecl-apc

This will install a apc config file at /etc/php.d/apc.ini, You can learn more about the params at http://www.php.net/manual/en/book.apc.php

apc.shm_size=64M

The above parameter is responsible for the size of the cache. You can increase or decrease it as you like it.I am currently leaving it at default. Copy APC cache file to your web directory

cp /usr/share/doc/php-pecl-apc-3.1.15/apc.php /home/leog.in/
service php-fpm restart 


and resolve http://leog.in/apc.php Lighttp php configuration Hits mean, the request has been served from the cache and misses mean it has not. MySQL Query cache. Add the below code in /etc/my.cnf

query-cache-type = 1    #0=off, 1=on, 2=on demand
query_cache_size = 20M  #total amount of memory to use for the cache, depending on the amount of memory your server has
query_cache_limit = 1M  #the max size of any one query result which will be cached, prevents a single massive query filling up the cache
service mysqld restart

This installation is fine for a small Wordpress blog or a small site which has no more than a 1000 to 2000 pageviews a month.

For a large Wordpress Blog or site I recommend you use Nginx, you can read about it at  How to Install  Nginx and PHP-FPM  for WordPress 
