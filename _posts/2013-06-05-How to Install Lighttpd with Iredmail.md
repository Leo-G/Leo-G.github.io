Lately, I have been trying different web servers with the following Goals in Mind.

The response should be fast (as in resolve quick in a 2G network or a score of 90 n above in page speed)

Should be resource friendly, utilize less memory as possible.

With these two Goals in mind I have been experimenting with Nginx and Lighttpd .

I decided to try Lighttpd, partly because it is recommended by roundcube., and nginx you can only install modules at compile time.
http://trac.roundcube.net/wiki/Howto_Requirements

In order to Install lighttpd on iRedMAil you need to have the epel repo



rpm -Uvh http://apt.sw.be/redhat/el6/en/i386/rpmforge/RPMS/rpmforge-release-0.5.3-1.el6.rf.i686.rpm
yum -y install lighthttpd lighhttpd-fastcgi php-fpm
This post assumes that you have iRedmail installed, if not then please install it first

It is less than 1MB and has no dependencies

open the config file



vim /etc/lighttpd/lighttpd.conf
Check the following directives and either add or change them as follows



##
var.server_root = "/var/www"
server.use-ipv6 = "disable"
include "modules.conf"
include_shell "cat /etc/lighttpd/vhosts.d/*.conf"
include "conf.d/fastcgi.conf"
Next goto the vhost directory and setup your site conf file



cd /etc/lighttpd/vhosts.d/
vim mail.yoursite.com.conf
$HTTP["host"] == "mail.yoursite.com" {
var.server_name = "mail.yoursite.com"
server.name = server_name
server.document-root = "/var/www/roundcubemail/"
##
## use a seperate access log file
## At the moment you cant have different error log files.
##
accesslog.filename = "/var/log/lighttpd/mail.yoursite.com/access.log"
}
$SERVER["socket"] == ":443" {
server.name = "mail.yoursite.conf"
ssl.pemfile = "/etc/lighttpd/ssl/mail.yoursite.com/yoursite.pem"
ssl.engine = "enable"
server.document-root = "/var/www/roundcubemail/"
}
You need to add your private key and certificate in one file called pem.
The iredmail ssl keys and cet are in the follow directries
/etc/pki/tls/certs/iRedMail_CA.pem
/etc/pki/tls/private/iRedMail.key
cat iRedMail.key iRedMail_CA.pem > yoursite.pem


The sequence is not important.

create a ssl folder



cd /etc/lighttpd/
mkdir -p ssl/mail.yoursite.com/
mv yoursite.pem /etc/lighttpd/ssl/mail.yoursite.com/
chown -R lighttpd:lighttpd ssl
FastCGI module and config

You can configure modules in modules.conf in /etc/lighttpd/modules.conf, uncommet the module in server.modules parameter

But fastcgi it is better to use a separate file in conf.d directory



vim /etc/lighttpd/conf.d/fastcgi.conf
add
server.modules += ( "mod_fastcgi" )
fastcgi.server = ( "php-local" =>
((
"host" => "127.0.0.1",
"port" => "9000",
"broken-scriptfilename" => "enable",
)
))
That's it, now start php-fpm and lighttpd


service php-fpm start
service lighttpd start

If you get the below error then disable seliniux

root@testserver /]# /etc/init.d/lighttpd start
Starting lighttpd: 2011-07-30 06:52:41: (server.c.722) couldn't set 'max filedescriptors' Permission denied
[FAILED]

Disable selinux in



vim /etc/selinux


If you have any issues disable iptables and check as well. 

source
http://redmine.lighttpd.net/projects/lighttpd/wiki/Docs_PerformanceFastCGI

http://www.tecmint.com/install-lighttpd-with-mysql5-and-php5-php-fpm-support-on-rhel-centos-6-3-5-8/

 

 
