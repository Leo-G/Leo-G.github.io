This Configuration is for CentOS 6, it should also work on Centos 5 although I have not tried it. This guide is for those who have basic linux command line knowledge.

I use atomic repo, as they have the latest updates, you can use epel, the choice is yours. But if there is any vulnerability then you will have to manually compile the patch

Step 1: Add Atomic Repo

wget -q -O - http://www.atomicorp.com/installers/atomic | sh
yum update

Step 2: Install Nginx

yum install nginx

Step3: Configure Nginx

Open Nginx conf file vim /etc/nginx/nginx.conf
Modify the Below lines, the number of processes depends on your CPU cores so if you have 2 then it should be 2 and the number of connections depend on your RAM, For a 2 core cpu with 750MB ram you can add the below config

worker_processes 2;
worker_connections 1024;
keepalive_timeout 5;

Keep alive should be as less as possible so that connections do not take up memory.

Next add the below line

include /etc/nginx/sites-enabled/*.conf;

Step 4: make 2 directories in /etc/nginx/

mkdir sites-available
mkdir sites-enabled

Create a file with named yourdomain.com.conf in sites-available and add the below config to it

server {
listen *:80;
server_name leog.in www.leog.in blog.leog.in;
error_log /var/log/nginx/leog.in-error.log;
access_log /var/log/nginx/leog.in-access.log main;
root /home/sites/leog.in;
location / {
index index.php index.html index.htm;
}
#Permalink redirect http://nginxlibrary.com/wordpress-permalinks
location /wordpress {
index index.php index.html index.htm;
try_files $uri $uri/ /wordpress/index.php?$args;
}
# force to use leog.in witout 'www'
if ($host = 'www.leog.in' ) {
rewrite ^/(.*)$ http://leog.in/$1 permanent;
}
if ($host = 'blog.leog.in' ) {
rewrite ^/(.*)$ http://leog.in/wordpress permanent;
}
#er_name wordpress.leog.in;
# return 301 "http://leog.in/wordpress${uri}";
# get friendly url links working
# if (!-e $request_filename)
# {
# rewrite ^(.+)$ /index.php?q=$1 last;
#}
# pass all PHP files through php-fpm
location ~ .php$ {
#root /home/sites/leog.in;
fastcgi_pass 127.0.0.1:9000;
fastcgi_index index.php;
fastcgi_param SCRIPT_FILENAME /home/sites/leog.in$fastcgi_script_name;
include /etc/nginx/fastcgi_params;
}
# deny all apache .htaccess or .htpasswd files
location ~ /.ht
{
deny all;
}

Replace leog.in with your domain where ever applicable if your using vim then you can use the below code



:%s/leog.in/yourdomain.com/g

Make simlink to sites-enabled



ln -sf /etc/nginx/sites-available/leog.in.conf /etc/nginx/sites-enabled/

Step 5: Install php-fpm

yum install php php-fpm php-gd php-mysql php-mbstring php-xml php-mcrypt

Add a user for your site and goto /etc/php-fpm.d/, open www.conf and add the change the user to yourusername

; RPM: apache Choosed to be able to access some dir as httpd
user = leo
; RPM: Keep a group allowed to write in log dir.
group = leo

Now restart Nginx and php-fpm

service nginx restart
service php-fpm restart

php-fpm uses a lot of memory

max children-- This is the maximum number of child processes, this should be kept low and should be the same as max spare servers.
The more processes the more memory used, hence this should be low

max requests : This should neither be too low or too high as too low means that the process needs to be recreated very often thus cpu usage and too high means that memory used will be more as to handle more connections memory is used.

You should also read a speed up php with opcode cache

For optimization please check http://www.if-not-true-then-false.com/2011/nginx-and-php-fpm-configuration-and-optimizing-tips-and-tricks/

http://nls.io/optimize-nginx-and-php-fpm-max_children/
