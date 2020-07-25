 Nginx with pagespeed


I moved to Nginx from Apache about 6 months ago and havn't looked back since. My Goal was to install a web server for my wordpress sites which is fast, light and easy to configure.

The only draw back of  Nginx is that it does not support DSO or Loading Modules dynamically like Apache.

This is not a problem if you want to install and run only Wordpress, but it is a problem if you want to install external modules like Google's pagespeed without having to recompile nginx.

The Solution I propose was to use Lighthttpd, which was event driven, similar to Nginx and lighter but instead I came across Tengine on the Google pagespeed documentation page.

What is Tengine?

Tengine is a fork of Nginx by Taobao , the largest e-commerce website in Asia as per their website  http://tengine.taobao.org/. Alibaba also uses this server for their site

If you know how to install and configure Nginx then you should have no problem configuring Tengine.

How do I install Tengine on my Centos or Red Hat machine?

Well you would need to compile it, yes I know no rpm or yum but it is not that difficult.
First Download and uninstall Tengine


wget http://tengine.taobao.org/download/tengine-1.5.1.tar.gz
tar -xzvf tengine-1.5.1.tar.gz
cd tengine-1.5.1
sudo ./configure
sudo make
sudo make install
The default installation location is /usr/local/nginx/ but you can modify the same by using --prefix option along with ./configure to set a location.

More ./configure options are here http://tengine.taobao.org/document/install.html

The path to conf file and module file is as follows

/usr/local/nginx/conf

/usr/local/nginx/modules

The nginx and dso executable binaries are in /usr/local/nginx/sbin.

I am not going into nginx configuration, you should know that by now if not you can check the article here.

Below is the startup script you need to add in /etc/init.d/nginx


-----------------------------------------------
#!/bin/bash
# nginx Startup script for the Nginx HTTP Server
# it is v.0.0.2 version.
# chkconfig: - 85 15
# description: Nginx is a high-performance web and proxy server.
# It has a lot of features, but it's not for everyone.
# processname: nginx
# pidfile: /var/run/nginx.pid
# config: /usr/local/nginx/conf/nginx.conf
nginxd=/usr/local/nginx/sbin/nginx
nginx_config=/usr/local/nginx/conf/nginx.conf
nginx_pid=/usr/local/nginx/logs/nginx.pid
RETVAL=0
prog="nginx"
# Source function library.
. /etc/rc.d/init.d/functions
# Source networking configuration.
. /etc/sysconfig/network
# Check that networking is up.
[ ${NETWORKING} = "no" ] && exit 0
[ -x $nginxd ] || exit 0
# Start nginx daemons functions.
start() {
if [ -e $nginx_pid ];then
echo "nginx already running...."
exit 1
fi
echo -n $"Starting $prog: "
daemon $nginxd -c ${nginx_config}
RETVAL=$?
echo
[ $RETVAL = 0 ] && touch /var/lock/subsys/nginx
return $RETVAL
}
# Stop nginx daemons functions.
stop() {
echo -n $"Stopping $prog: "
killproc $nginxd
RETVAL=$?
echo
[ $RETVAL = 0 ] && rm -f /var/lock/subsys/nginx /usr/local/nginx/logs/nginx.pid
}
reload() {
echo -n $"Reloading $prog: "
#kill -HUP `cat ${nginx_pid}`
"nginx" [readonly] 74L, 1453C
#kill -HUP `cat ${nginx_pid}`
killproc $nginxd -HUP
RETVAL=$?
echo
}
configtest() {
$nginxd -t -c $nginx_config
}
# See how we were called.
case "$1" in
start)
start
;;
stop)
stop
;;
reload)
reload
;;
restart)
stop
start
;;
configtest)
configtest
;;
status)
status $prog
RETVAL=$?
;;
*)
echo $"Usage: $prog {start|stop|restart|reload|status|help}"
exit 1
esac
exit $RETVAL
----------------------------------------

sudo chmod 755 nginx
sudo service nginx start
sudo chkconfig nginx on


Everything is fine till here, then you can go ahead and configure the page speed module.

First install the required Dependencies

sudo yum install gcc-c++ pcre-dev pcre-devel zlib-devel make
Then download and install


wget https://github.com/pagespeed/ngx_pagespeed/archive/release-1.6.29.5-beta.zip
unzip release-1.6.29.5-beta.zip
cd ngx_pagespeed-release-1.6.29.5-beta/
wget https://dl.google.com/dl/page-speed/psol/1.6.29.5.tar.gz
tar -xzvf 1.6.29.5.tar.gz # expands to psol/
cd /usr/local/nginx/sbin
sudo ./dso_tool --add-module=/path/to/ngx_pagespeed-release-1.6.29.5-beta
This will prepare a dynamically loadable module out of ngx_pagespeed. To check that it worked you can verify that "/path/to/tengine/modules/" contains an ngx_pagespeed.so .

You need to tell tengine to load this module

Add the below line in your nginx.conf file at the top


dso {
load ngx_pagespeed.so;
}
Then in your server block add the following lines


pagespeed on;
pagespeed FileCachePath /var/ngx_pagespeed_cache;
location ~ ".pagespeed.([a-z].)?[a-z]{2}.[^.]{10}.[^.]+" { add_header "" ""; }
location ~ "^/ngx_pagespeed_static/" { }
location ~ "^/ngx_pagespeed_beacon$" { }
location /ngx_pagespeed_statistics { allow 127.0.0.1; deny all; }
location /ngx_pagespeed_message { allow 127.0.0.1; deny all; }
location /pagespeed_console { allow 127.0.0.1; deny all; }
Restart nginx



sudo service nginx restart
Check if it is installed with the following command



$ curl -I 'http://localhost:8050/some_page/' | grep X-Page-Speed
X-Page-Speed: 1.6.29.5-...


Images are not mine and are found on the net
