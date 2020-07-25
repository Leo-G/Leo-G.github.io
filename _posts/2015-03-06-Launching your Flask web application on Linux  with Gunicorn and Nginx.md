Flask is a web framework for python, Flask tackles Routing, HTML template rendering, Sessions etc. If you are building web applications on Linux then I highly recommend using Flask, here's how quickly you can build an app with it on Linux.
The inbuilt server with Flask is good for development, but sooner or later you will want to serve hundreds or even thousands of request, hence in this post I will describe how you can deploy your flask application on a production Linux servver with Gunicorn and Nginx.

Gunicorn is a WSGI HTTP server written in Python and used to serve Python files, it uses a pre-fork worker model in which it forks worker processes to handle requests. Although Gunicorn can serve as a standalone server it is recommended to use Nginx as a frontend and reverse proxy Python requests to Gunicorn.

Software versions

Ubuntu 14.04

Python 2.7.6

Linux kernel  3.13.0-24-generic

Flask 0.10.1

Gunicorn 19.2.1

Nginx  1.4.6

Installing  Nginx and Gunicorn 

leo@flask:~$ sudo  apt-get install python-pip python-virtualenv nginx supervisor
leo@flask:~$ mkdir myapp
leo@flask:~$ cd myapp
leo@flask:~/myapp$ virtualenv venv
leo@flask:~/myapp$ source venv/bin/activate
(venv)leo@flask:~/myapp$ pip install flask gunicorn
I have installed Gunicorn in a Virtual Environment via “pip” so I can use the latest version, you can install it via “apt” as well but it does not have the latest version.

Write some application code

(venv)leo@flask:~/myapp$ vim app.py
from flask import Flask
app = Flask(__name__)
@app.route('/')
def index():
 return 'Houston we have lift off'
if __name__ == '__main__':
app.run()

Start Gunicorn

(venv)leo@flask:~/myapp$ gunicorn app:app -b localhost:5000
[2015-03-06 09:59:28 +0000] [12794] [INFO] Starting gunicorn 19.2.1
[2015-03-06 09:59:28 +0000] [12794] [INFO] Listening at: http://127.0.0.1:5000 (12794)
[2015-03-06 09:59:28 +0000] [12794] [INFO] Using worker: sync
[2015-03-06 09:59:28 +0000] [12799] [INFO] Booting worker with pid: 12799

You should see “Houston we have lift off”  when you resolve http://localhost:5000
install gunicorn on ubuntu

Daemon-izing Gunicorn with Supervisor

Supervisor allows you to control and monitor  processes on  Linux. With Supervisor you can start/stop/restart Gunicorn via the command line just like an init process and view it’s logs in “/var/log/supervisor/”. One of the key benefits of Supervisor is it's simple configuration.

#Add the Gunicorn configuration to Supervisor
(venv)leo@flask:~/myapp$ sudo vim /etc/supervisor/conf.d/myapp
#name of the process
[program:myapp]
#command to run
command = /complete/path/to/venv/bin/gunicorn app:app -b localhost:5000
#complete path to your application directory
directory = /path/to/myapp/
#User to run the process with
user = username
#Start process at system boot
autostart=true

Ensure you add the complete path to the “gunicorn” command else you will get a “Error: File not found” message.

(venv)leo@flask:~/myapp$ sudo supervisorctl reread
(venv)leo@flask:~/myapp$ sudo supervisorctl update
(venv)leo@flask:~/myapp$ sudo supervisorctl start myapp

To stop or  restart Gunicorn you can use

(venv)leo@flask:~/myapp$ sudo supervisorctl stop myapp
(venv)leo@flask:~/myapp$ sudo supervisorctl restart myapp

Configure  and Start Nginx

(venv)leo@flask:~/myapp$ mv /etc/nginx/sites-enabled/default /tmp/nginx.bak
(venv)leo@flask:~/myapp$ vim /etc/nginx/sites-enabled/myapp
server {
location / {
proxy_pass http://127.0.0.1:5000;
}
}
(venv)leo@flask:~/myapp$ sudo service nginx start

You should be able to successfully resolve localhost and view “Houston we have lift off” if everything went ok.

Optimize

One way to speed up your app is to serve static files like Javascript and Images directly through Nginx.

(venv)leo@flask:~/myapp$ sudo vim /etc/nginx/sites-enabled/myapp
server {
location / {
proxy_pass http://127.0.0.1:5000;
}
#Serve all static files including js and css directly
location ~\.(jpg|png|gif|js|css)${
root /myapp/static/;
}
}
(venv)leo@flask:~/myapp$ sudo service nginx restart

Create a directory static and add an image file to it.

(venv)leo@flask:~/myapp$ mkdir static
(venv)leo@flask:~/myapp/static$ll
-rw-r--r-- 1 leo_g leo_g 130868 Mar  6 18:24 myimage.jpg
Add some HTML code to your app to serve images

(venv)leo@flask:~/myapp$ vim app.py
from flask import Flask
app = Flask(__name__)
@app.route('/')
def index():
 return '''<html> <body>
 <h2>Houston we have lift off</h2>
 <img src="myimage.jpg" height="700">
 </body></html>'''
if __name__ == '__main__':
app.run()

Now restart Gunicorn

(venv)leo@flask:~/myapp$ sudo supervisorctl restart myapp

deploy flask with Gunicorn

Have fun
