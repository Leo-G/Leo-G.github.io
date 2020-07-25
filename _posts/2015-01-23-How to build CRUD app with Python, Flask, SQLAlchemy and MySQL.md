In this post I will briefly describe, how you can you build a database driven CRUD (Create, Read, Update, Delete) app on Linux with Python, Flask, SQLAlchemy and MySQL. I used this process to create a blog and hence the examples below will describe how to store and modify posts in a MySQL database.

Update: You can scaffold a database driven CRUD app with https://github.com/Leo-g/Flask-Scaffold

Software Versions
Python 2.7
Flask 0.11
Flask-SQLAlchemy 2.0
Flask-Migrate 1.3
MySQL-python 1.2
Foundation 5
Mariadb 10

Before you continue if you have not built an application on Linux with Flask or Python then I recommend you read Creating your first Linux App with Python and Flask.

Directory Creation

mkdir flask-blog
mkdir flask-blog/app
#HTML files will reside in the templates folder.
mkdir flask-blog/app/templates
#CSS/JS and other static files go in the static folder
mkdir flask-blog/app/static

Here is the directory structure and the files that they will contain

/flask-blog
    |-- run.py             
    |-- config.py   
    |__ /venv 
    |-- db.py           
    |__ /app            
             |-- __init__.py
             |-- views.py
             |-- models.py         #will contain our sql code           
         |__ /templates
             |-- index.html
         |__ /static
             |--foundation.css

Install and Setup your MySQL server.
I used Mariadb 10 which is a fork of MySQL, you can use MySQL 5.6 if you prefer and install the relevant development packages.

[leo@flask-blog]$ yum install MariaDB-server MariaDB-devel
#Start the server
[leo@flask-blog]$ sudo systemctl start mysql
#Set the root password
[leo@flask-blog]$  mysqladmin -u root -p  ‘’ password ‘newpassword’

Remember to install ‘ MariaDB-devel’, else you will get an error when you install MysSQL Python packages later.
Setup and activate the virtual environment

[leo@flask-blog]$ pip install virtualenv 
[leo@flask-blog]$ virtualenv flask-blog/venv
[leo@flask-blog]$ source flask-blog/venv/bin/activate

Flask installation

(venv)[leo@flask-blog]$ pip install https://github.com/mitsuhiko/flask/tarball/master
(venv)[leo@flask-blog]$ pip install  Flask-SQLAlchemy mysql-python Flask-Migrate 

I have installed the latest version of Flask via git, you can install it directly as well via ‘pip install Flask’. I also installed Flask-SQLAlchemy, This extension gives us all the benefits of SQLAlchemy which we need to use for database operations.

Optional Download and Install Foundation 5
This is an optional step, Foundation 5 takes care of the HTML/CSS framework and has ready templates which you can use to design your blog or application. You can use bootstrap if you like.

(venv)[leo@flask-blog]$ cd flask-blog/app/static
(venv)[leo@flask-blog]$ wget http://foundation.zurb.com/cdn/releases/foundation-5.5.0.zip
(venv)[leo@flask-blog]$unzip foundation-5.5.0.zip

Database Migrations
Migrations keep track of any changes made to the database, they are beneficial especially when you need to migrate your database. Flask-Migrate, which we installed earlier will help us take care of this.

#Create an initialization file for your App
(venv)[leo@flask-blog]$ vim app/__init.py__

from flask import Flask
from flask.ext.sqlalchemy import SQLAlchemy
#Create an Instance of Flask
app = Flask(__name__)
#Include config from config.py
app.config.from_object('config')
app.secret_key = 'some_secret'
#Create an instance of SQLAclhemy
db = SQLAlchemy(app)
from app import views, models
(venv)[leo@flask-blog]$ vim flask-blog/config.py
#Add your connection string
SQLALCHEMY_DATABASE_URI = 'mysql://root:password@localhost/blog'
The initialization file is where we need to declare our objects, for example the 'app' object will create an instance of Flask and the 'db' object will create an instance of SQLALchemy with the configuration of the app object as shown in the code above. I have stored the database connection string which contains the database login details in a global configurations file called config.py. This way if anyone wants to reuse my app they just need to make changes to'config.py'.

I have also imported two modules called views and models, views will contain our main code where as models will contain our database related code.

(venv)[leo@flask-blog]$ vim flask-blog/app/models.py

from app import db
class Post(db.Model):
  id = db.Column(db.Integer, primary_key=True)
  title = db.Column(db.String(128))
  body = db.Column(db.Text)
  def __init__(self, title, body):
   self.title = title
   self.body = body
The db SQLAlchemy object contains a ‘db.Model’ and a 'db.Column' method to map tables and columns to classes and objects respectively. The types of the column can be passed as an argument as shown above.

In order to create these tables and columns you need to use Flask-Migrate.

(venv)[leo@flask-blog]$ vim flask-blog/db.py

from flask.ext.script import Manager
from flask.ext.migrate import Migrate, MigrateCommand
from config import SQLALCHEMY_DATABASE_URI
from app import app, db
migrate = Migrate(app, db)
manager = Manager(app)
manager.add_command('db', MigrateCommand)
if __name__ == '__main__':
 manager.run()
Flask-Migrate has two methods 'Migrate' and 'MigrateCommand'. Migrate is used to initialize the extension, while Manager gives you access to command line options.
The application will now have a db command line option with several sub-commands.

(venv)[leo@flask-blog]$ chmod +x db.py
#Initialize migrations support
(venv)[leo@flask-blog]$ python db.py db init
#Generate a migration
(venv)[leo@flask-blog]$ python db.py db migrate
(venv)[leo@flask-blog]$ python db.py db upgrade

Once you have successfully connected and mapped your tables with SQLAlchemy you can begin to code the CRUD(Create,Read,Update, Delete) part of your app.
Create
To add posts to your blog, first create an add.html template.

(venv)[leo@flask-blog]$ vim flask-blog/app/templates/add.html

&lt;form action="" method="post"&gt;
 Title
 &lt;input type="text" name="title"/&gt; 
 Body
 &lt;textarea type="text" name="body"&gt;&lt;/textarea&gt;
 
 &lt;button class="button postfix" type="submit"&gt;Add&lt;/button&gt; 

Now define a function that will handle your 'add' requests.

 (venv)[leo@flask-blog]$ vim flask-blog/app/views.py

from flask import render_template, request,flash, redirect, url_for
from app import app, db
from app.models import Post
@app.route('/add' , methods=['POST', 'GET'])
def add():
 if request.method == 'POST':
 post=Post(request.form['title'], request.form['body'])
 db.session.add(post)
 db.session.commit()
 flash('New entry was successfully posted')
return render_template('add.html')
To add or delete entries we need to use 'session'. 'db.session.add(post)' will store data that needs to be entered but a change will not be made in the database until you call commit.

Running your app

(venv)[leo@flask-blog]$ vim flask-blog/run.py

from app import app
app.debug = True
app.run(host='1.2.3.4')
(venv)[leo@flask-blog]$ python flask-blog/run.py
* Running on http://1.2.3.4:5000/
 * Restarting with reloader


If everything works as planned then you should be able to add posts successfully on resolving “http://1.2.3.4:5000/add”
flask sqlachemy tutorial
Read
In order to display the blog post, create an index page with a corresponding function which will display all posts.

(venv)[leo@flask-blog]$ vim flask-blog/app/views.py

@app.route('/' )
def index():
  post = Post.query.all()
  return render_template('index.html', post=post)


SQLAlchemy provides a ‘query’ attribute to all instances of a class. You can use it to get data from the database, if you need to access data of the Post table then you can use 'Post.query.all' as shown above. I have used ‘all()’ to get all entries but you can use ‘filter_by(title=title).first’ to get selective data.

To display the data add the following code in your index.html file.

(venv)[leo@flask-blog]$ vim flask-blog/app/templates/index.html
&lt;!doctype html&gt;
&lt;html class="no-js" lang="en"&gt;
 &lt;head&gt;
 &lt;meta charset="utf-8" /&gt;
 &lt;meta name="viewport" content="width=device-width, initial-scale=1.0" /&gt;
 &lt;title&gt;Flask Blog&lt;/title&gt;
 &lt;link rel="stylesheet" href="static/css/foundation.css" /&gt;
 &lt;script src="static/js/vendor/modernizr.js"&gt;&lt;/script&gt;
 &lt;/head&gt;
 &lt;body&gt;
 
 &lt;!-- Main Page Content and Sidebar --&gt;
 {% for post in post %}
 &lt;div class="row"&gt; 
 &lt;!-- Main Blog Content --&gt;
 &lt;div class="large-9 columns" role="content"&gt; 
 &lt;article&gt; 
 &lt;h3&gt;{{ post.title }}&lt;/h3&gt; 
 
 &lt;div class="row"&gt;
 &lt;div class="large-6 columns"&gt;
 &lt;p&gt;{{ post.body }}&lt;/p&gt;
 &lt;p&gt;&lt;a href="{{ url_for('edit', id=post.id) }}"&gt;Edit&lt;/a&gt;&lt;/p&gt;
 
 &lt;/article&gt;
 
 &lt;hr /&gt;
 
 {% endfor %}
 
 &lt;!-- End Main Content --&gt; 
 
 &lt;script src="static/js/vendor/jquery.js"&gt;&lt;/script&gt;
 &lt;script src="static/js/foundation.min.js"&gt;&lt;/script&gt;
 &lt;script&gt;
 $(document).foundation();
 &lt;/script&gt;
 
 &lt;/body&gt;
 &lt;/html&gt; 

Update and Delete

For update and delete add the following code and the corresponding template for .

(venv)[leo@flask-blog]$ vim flask-blog/app/views.py

You can access this code in the Gist here, There was some problem displaying it.

Corresponding templates.

(venv)[leo@flask-blog]$ vim flask-blog/app/templates/edit.html 

&lt;!doctype html&gt;
&lt;html class="no-js" lang="en"&gt;
 &lt;head&gt;
 &lt;meta charset="utf-8" /&gt;
 &lt;meta name="viewport" content="width=device-width, initial-scale=1.0" /&gt;
 &lt;title&gt; Blog&lt;/title&gt;
 &lt;link rel="stylesheet" href="static/css/foundation.css" /&gt;
 &lt;script src="static/js/vendor/modernizr.js"&gt;&lt;/script&gt;
 &lt;/head&gt;
 &lt;body&gt;
&lt;form action="/edit" method="post"&gt;
 &lt;div class="row"&gt;
 &lt;div class="large-12 columns"&gt;
 &lt;label&gt;Title
 &lt;input type="text" name="title" value="{{post.title}}"/&gt;
 &lt;/label&gt;
 &lt;/div&gt;
 &lt;/div&gt;
&lt;div class="row"&gt;
 &lt;div class="large-12 columns"&gt;
 &lt;label&gt;Post
 &lt;textarea type="text" name="text"&gt;{{ post.body}}&lt;/textarea&gt;
 &lt;/label&gt;
 &lt;/div&gt;
 &lt;/div&gt;
 &lt;div class="small-2 columns"&gt;
 &lt;button class="button postfix" type="submit"&gt;Edit&lt;/button&gt;
 &lt;/div&gt;
 &lt;/div&gt;
 &lt;/div&gt;
&lt;/div&gt;
&lt;/form&gt;
&lt;script src="js/vendor/jquery.js"&gt;&lt;/script&gt;
 &lt;script src="js/foundation.min.js"&gt;&lt;/script&gt;
 &lt;script&gt;
 $(document).foundation();
 &lt;/script&gt;
 &lt;/body&gt;
&lt;/html&gt;
[/xml]
Add edit and delete links in your index.html
[xml]
&lt;!doctype html&gt;
&lt;html class="no-js" lang="en"&gt;
 &lt;head&gt;
 &lt;meta charset="utf-8" /&gt;
 &lt;meta name="viewport" content="width=device-width, initial-scale=1.0" /&gt;
 &lt;title&gt;Flask Blog&lt;/title&gt;
 &lt;link rel="stylesheet" href="static/css/foundation.css" /&gt;
 &lt;script src="static/js/vendor/modernizr.js"&gt;&lt;/script&gt;
 &lt;/head&gt;
 &lt;body&gt;
 

 &lt;!-- Main Page Content and Sidebar --&gt;
 {% for post in post %}
 &lt;div class="row"&gt;
 
 &lt;!-- Main Blog Content --&gt;
 &lt;div class="large-9 columns" role="content"&gt;
 
 &lt;article&gt;
 
 &lt;h3&gt;{{ post.title }}&lt;/h3&gt;
 
 
 &lt;div class="row"&gt;
 &lt;div class="large-6 columns"&gt;
 &lt;p&gt;{{ post.text }}&lt;/p&gt;
 &lt;p&gt;&lt;a href="{{ url_for('edit', id=post.id) }}"&gt;Edit&lt;/a&gt;&lt;/p&gt;
 &lt;p&gt;&lt;a href="{{ url_for('delete', id=post.id) }}"&gt;Delete&lt;/a&gt;&lt;/p&gt;

 &lt;/article&gt;
 
 &lt;hr /&gt;
 
 {% endfor %}
 
 &lt;!-- End Main Content --&gt; 
 
 &lt;script src="static/js/vendor/jquery.js"&gt;&lt;/script&gt;
 &lt;script src="static/js/foundation.min.js"&gt;&lt;/script&gt;
 &lt;script&gt;
 $(document).foundation();
 &lt;/script&gt; 
 &lt;/body&gt;
 &lt;/html&gt; 

Now run your app

(venv)[leo@flask-blog]$ python flask-blog/run.py
flask slqalchemy tutorial

Other Tutorials you must Read to build Database applications

One to Many Relationships with SQLAlchemy

Many to Many Relationships with SQLAlchemy

Building Database Driven REST API's with Flask

 You can also quickly Scaffold a CRUD app with  https://github.com/Leo-g/Flask-Scaffold. 

References:
https://pythonhosted.org/Flask-SQLAlchemy/
https://pythonhosted.org/Flask-SQLAlchemy/queries.html
http://docs.sqlalchemy.org/en/rel_0_9/core/engines.html
https://flask-migrate.readthedocs.org/en/latest/
https://github.com/miguelgrinberg/Flask-Migrate
