In this tutorial I will describe how you can run asynchronous tasks on Linux using Celery an asynchronous task queue manager.

While running scripts on Linux some tasks which take time to complete can be done asynchronously. For example a System Update. With Celery you can run such tasks asynchronously in the background and then fetch the results once the task is complete.

You can use celery in your python script and run it from the command line as well but in this tutorial I will be using Flask a Web framework for Python to show you how you can achieve this through a web application.

Before we start it’s good if you have some familiarity with Flask if not you can quickly read my earlier tutorial on building Web Applications on Linux with Flask before you proceed.

This tutorial is for Python 3.4, Flask 0.10, Celery 3.1.23 and rabbitmq-server 3.2.4-1

To make it easier for you I have generated all the code required for the web interface using Flask-Scaffold and
uploaded it at https://github.com/Leo-G/Flask-Celery-Linux. You will just need to clone the code and proceed with the installation and configuration as follows:

Installation

As described above the first step is to clone the code on your Linux server and install the requirements

git clone https://github.com/Leo-G/Flask-Celery-Linux
cd Flask-Celery-Linux
virtualenv -p python3.4 venv-3.4
source venv-3.4/bin/activate
pip install -r requirements.txt
sudo apt-get install rabbitmq-server

Most of the requirements including Flask and Celery will be installed using 'pip' however we will need to install RabbitMQ via 'apt-get' or your distros default package manager.

What is RabbitMQ?
RabbitMQ is a message broker. Celery uses a message broker like RabbitMQ to mediate between clients and workers. To initiate a task, a client adds a message to the queue, which the broker then delivers to a worker. There are other message brokers as well but RabbitMQ is the recommended broker for Celery.

Configuration

Configurations are stored in the config.py file. There are two configurations that you will need to add,
One is your database details where the state and results of your tasks will be stored and two is
the RabbitMQ message broker URL for Celery.

vim config.py

#You can add either a Postgres or MySQL Database 
#I am using MySQL for this tutorial
#https://github.com/Leo-G/Flask-Celery-Linux/blob/master/config.py

mysql_db_username = 'youruser'
mysql_db_password = 'yourpass'
mysql_db_name = 'flask_celery_linux'
mysql_db_hostname = 'localhost'
SQLALCHEMY_DATABASE_URI = "mysql+pymysql://{DB_USER}:DB_PASS}@{DB_ADDR}/{DB_NAME}".format(DB_USER=mysql_db_username, DB_PASS=mysql_db_password, DB_ADDR=mysql_db_hostname, DB_NAME=mysql_db_name)

#Celery Message Broker Configuration

CELERY_BROKER_URL = 'amqp://guest@localhost//'
CELERY_RESULT_BACKEND = "db+{}".format(SQLALCHEMY_DATABASE_URI)
Database Migrations

Run the db.py script to create the database tables

python db.py db init
python db.py db migrate
python db.py db upgrade

And finally run the in built web server with

python run.py

You should be able to see the Web Interface at http://localhost:5000

Python 3 Asynchronous example

You will need to create a username and password by clicking on sign up, after which you can login.

Starting the Celery Worker Process

In a new window/terminal activate the virtual environment and start the celery worker process

cd Flask-Celery-Linux
source venv-3.4/bin/activate
celery worker -A celery_worker.celery --loglevel=debug

Now go back to the Web interface and click on Commands --> New. Here you can type in any Linux command and see it run asynchronously.

The video below will show you a live demonstration





Working

To integrate Celery into your Python script or Web application you first need to create an instance of
celery with your application name and the message broker URL.


#full code at https://github.com/Leo-G/Flask-Celery-Linux/blob/master/app/__init__.py
from celery import Celery
from config import CELERY_BROKER_URL

celery = Celery(__name__, broker=CELERY_BROKER_URL)
Any task that has to run asynchronously then needs to be wrapped by a Celery decorator


#complete code at https://github.com/Leo-G/Flask-Celery-Linux/blob/master/app/commands/views.py

from app import celery

@celery.task
def run_command(command):
 cmd = subprocess.Popen(command,stdout=subprocess.PIPE,stderr=subprocess.PIPE)
 stdout,error = cmd.communicate()
 return {"result":stdout, "error":error}
You can then call the task in your python scripts using the 'delay' or 'apply_async' method as follows:


#https://github.com/Leo-G/Flask-Celery-Linux/blob/master/app/commands/views.py

task = run_command.delay(command)


The difference between the 'delay' and the 'apply_async()' method is that the latter allows you to specify a time post which the task will be executed.


run_command.apply_async(args=[command], countdown=30)
The above command will be executed on Linux after a 30 second delay.

In order to obtain the task status and result you will need the task id.


#https://github.com/Leo-G/Flask-Celery-Linux/blob/master/app/commands/views.py
#https://github.com/Leo-G/Flask-Celery-Linux/blob/master/app/commands/models.py

 task = run_command.delay(cmd.split()) 
 task_id = task.id
 task_status = run_command.AsyncResult(task_id)
 task_state = task_status.state
 result = str(task_status.info)
 #Store results in the database using SQlAlchemy
 from models import Commands
 command = Commands(request_dict['name'], task_id, task_state, result)
 command.add(command)
Tasks can have different states. Pre-defined states include PENDING, FAILURE and SUCCESS. You can
define custom states as well.

Incase a task takes a long time to execute or you want to terminate a task pre-maturely you have to use the 'revoke' method.


#https://github.com/Leo-G/Flask-Celery-Linux/blob/master/app/commands/views.py
run_command.AsyncResult(task_id).revoke(terminate=True)
Just be sure to pass the terminate flag to it else it will be respawned when a celery worker process restarts.

Finally If you are using Flask Application Factories you will need to instantiate Celery when you create your Flask application.


#full code at https://github.com/Leo-G/Flask-Celery-Linux/blob/master/app/__init__.py
def create_app(config_filename):
 app = Flask(__name__, static_folder='templates/static')
 app.config.from_object(config_filename)
# Init Flask-SQLAlchemy
 from app.basemodels import db
 db.init_app(app)
celery.conf.update(app.config)
#https://github.com/Leo-G/Flask-Celery-Linux/blob/master/run.py
from app import create_app
app = create_app('config')
if __name__ == '__main__':
 app.run(host=app.config['HOST'],
 port=app.config['PORT'],
 debug=app.config['DEBUG'])
To run celery in the background you can use supervisord.

That's it for now, if you have any suggestions add them in the comments below

Ref:

http://docs.celeryproject.org/en/latest/userguide/calling.html
http://blog.miguelgrinberg.com/post/using-celery-with-flask

Images are not mine and are found on the internet
