In this Django Tutorial I will describe how you can move a PHP based wordpress blog to a  Python 3 based Django blog via the Linux command line.

Reasons to Switch to Django.

Built with Python
Better Security
Several modules built into the core system, as compared to externally vulnerable wordpress plugins
Software Versions

Django 1.10.8 (Dependent on Mezzanine)
Mezzanine 4.2.3
Debian 8
Python 3.4
Postgres 9.5
Step 1. Install Mezzanine a Django based CMS.

Mezzanine is a Django based project which has several blogging features inbuilt including an option to import wordpress posts. It has many themes available and Unlike many other platforms that make extensive use of modules or reusable applications, Mezzanine provides most of its functionality by default. This approach yields a more integrated and efficient platform. For a complete list of features visit https://goo.gl/2UWyyL.



 pyvenv-3.5 venv-3.5
 source venv-3.5/bin/activate
 pip install mezzanine
 mezzanine-project blog
 sudo -u postgres createdb techarena51
# Modify the below settings in blog/local_settings.py
vim blog/local_settings.py
#modify secret key
DATABASES = {
    "default": {
       # Ends with "postgresql_psycopg2", "mysql", "sqlite3" or "oracle".
        "ENGINE": "django.db.backends.postgresql_psycopg2",
        # DB name or path to database file if using sqlite3.
        "NAME": "techarena51",
        # Not used with sqlite3.
        "USER": "postgres",
       ALLOWED_HOSTS = ["yourhostname.com”]]

#create the Postgresql Database
  python manage.py createdb --noinput
  pip install psycopg2
  python manage.py createdb --noinput
  python manage.py runserver 0.0.0.0:8000
Verify that you have the default mezzanine page up and running at yourhost.com:8000.

Step 2: Migrate Wordpress Posts to Mezzanine

Mezzanine comes with a tool to migrate wordpress posts. You will need to export a xml file with all posts and relevant data from wordpress and then run the below command



python manage.py import_wordpress --mezzanine-user=leog --url=/path/to/wordpress.-backup.xml
Note: You may get a lot of ThreadedComment.user_name exceeds its maximum length of 50 chars Errors,  Just increase the limit and move on

In order to view your posts on the home page you need to  add/uncomment the following  from blog/urls.py and comment the default homepage



import mezzanine.blog.views
url("^$", mezzanine.blog.views.blog_post_list, {"template": "index.html"}, name="home"),
#url("^$", mezzanine.pages.views.page, {"slug": "/"}, name="home"),


Step 3 : Creating your own Mezzanine theme

Several Mezzanine themes can be found on Github, below is an example of how you can configure them. 

    git clone https://github.com/thecodinghouse/mezzanine-themes
    cd mezzanine-themes/
    mezzanine-project lucid
    cd lucid/
    python manage.py startapp theme
   python manage.py collecttemplates -t base.html
   python manage.py collecttemplates -t pages/menus/dropdown.html
   mv templates/pages/ theme/templates/
   mv templates/pages/ static/theme/templates/
   mv blog/templates/pages lucid/theme/template
   python manage.py  makemigrations
   python manage.py migrate theme
You need to manually move images, css and js from your wordpress blog to mezzanine

mv -r /your/wordpressblog/css js lucid/static/
mv -r /your/wordpressblog/uploads/ lucid/static/media/uploads/
To use featured images add the following in settings.py

BLOG_USE_FEATURED_IMAGE = true 
You need to Manually Add Keywords and to embed  a youtube video you need to change High to Low for Rich Text in settings.py

Step 4 : Install uwsgi and nginx web servers to serve requests to your Django Mezzanine Blog

  pip install uwsgi
  Add the following code in a file called lucid.wsgi and test uwsgi
  import os
from django.core.wsgi import get_wsgi_application
from mezzanine.utils.conf import real_project_name
os.environ.setdefault("DJANGO_SETTINGS_MODULE",
 "%s.settings" % real_project_name("lucid"))
application = get_wsgi_application()
   uwsgi --http :8000 --module lucid.wsgi
Next add the below config in nginx 

sudo vim /etc/nginx/nginx.conf
#Ensure that the below line is uncommented
 include /etc/nginx/sites-enabled/*.conf;

#To redirect URL's from index.php you need to add the below code to nginx.conf inside the server
# block
rewrite ^/index.php/category(.*) http://$server_name/blog/tag/$1 permanent;
rewrite ^/index.php(.*) http://$server_name/blog
sudo vim  /etc/nginx/sites-enabled/your-django-site.conf 
#Add the following
server {
 listen 80;
 server_name techarena51.com www.techarena51.com;
 return 301 https://$host$request_uri;
}

server {
listen 443 ssl default_server;
 server_name techarena51.com www.techarena51.com;
ssl_certificate /etc/letsencrypt/live/techarena51.com/fullchain.pem;
 ssl_certificate_key /etc/letsencrypt/live/techarena51.com/privkey.pem;

 # listen 80;
 # server_name techarena51.com;
 # for temp replace permanent with redirect
 rewrite ^/index.php(.*) http://$server_name/blog$1 permanent;
large_client_header_buffers 4 32k;
 client_max_body_size 50M;
 charset utf-8;
access_log /var/log/nginx/techarena51.com-access.log;
 error_log /var/log/nginx/techarena51.com-error.log;
# Static files
 location /static/ {
 alias /sites/techarena51.com/lucid/static/;
 }
location ^~ /.well-known/acme-challenge/ {
allow all;
 root /sites/techarena51.com/;
 }
# Backend
 location / {
uwsgi_pass 127.0.0.1:8002;
 include uwsgi_params;
 }
sudo systemctl restart nginx

Now create a uwsgi service 

sudo mkdir -p /etc/uwsgi/vassals
 sudo vim /etc/uwsgi/vassals/uwsgi.ini
#http://uwsgi-docs.readthedocs.io/en/latest/tutorials/Django_and_nginx.html
# mysite_uwsgi.ini file
[uwsgi]
# Django-related settings
# the base directory (full path)
chdir = /path/to/blog
# Django's wsgi file
module = lucid.wsgi
# the virtualenv (full path)
home = /path/to/venv-3.5
# process-related settings
# master
master = true
# maximum number of worker processes
processes = 10
# the socket (use the full path to be safe
socket = 127.0.0.1:8002
# ... with appropriate permissions - may be needed
# chmod-socket = 664
# clear environment on exit
vacuum = true
 sudo vim /etc/uwsgi/emperor.ini
#http://uwsgi-docs.readthedocs.io/en/latest/Systemd.html
[uwsgi]
emperor = /etc/uwsgi/vassals
uid = username
gid = username

 sudo vim /etc/systemd/system/emperor.uwsgi.service
[Unit]
Description=uWSGI Emperor
After=syslog.target
[Service]
ExecStart=/path/to/venv-3.5/bin/uwsgi --ini /etc/uwsgi/emperor.ini
# Requires systemd version 211 or newer
RuntimeDirectory=uwsgi
Restart=always
KillSignal=SIGQUIT
Type=notify
StandardError=syslog
NotifyAccess=all

[Install]
WantedBy=multi-user.target

sudo systemctl daemon-reload 
sudo systemctl enable emperor
sudo systemctl start emperor.uwsgi
sudo systemctl restart nginx
# For troubleshooting check the logs below
 sudo tail -f /var/log/syslog 
Notes:

You will need to goto http://yourdomain.com/admin/sites/site/ and modify 127.0.0.1 to your domain name

Categories are tags, you can find them at http://yourdomain.com/blog/tag/your-tag

The rss  feed url for mezzanine is at http://yourdomain.com/blog/feed/rss



