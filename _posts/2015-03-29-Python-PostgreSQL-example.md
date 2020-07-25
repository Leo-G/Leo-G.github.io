In this post I will describe how you can connect to a  PostgreSQL database  with Psycop2 , SQLAlchemy and Flask.

Update: I have written a  python script which lets you create a CRUD app along with validations in Python and PostgreSQL by simply specifying the  database fields, You can download it from https://github.com/Leo-G/Flask-Scaffold.

Software versions
Python 3.4
PostgreSQL 9.3
Flask 0.10.1
Flask-SQLAlchemy 2.0
psycopg2 2.6
SQLAlchemy 0.9.9
Ubuntu Linux 14.4

Install PostgreSQL 9 on Ubuntu Linux

leo@python-postgresql $ sudo apt-get install postgresql-9.3postgresql-client-9.3postgeresql-contrib-9.3
leo@python-postgresql $ sudo su - postgres
postgres@python-postgresql $ psql
postgres =&gt; create database yourdbname;
postgres =&gt; create user yourusername;
postgres =&gt;\password yourusername
postgres =&gt;GRANT ALL PRIVILEGES ON DATABASE yourdbname TO yourusername;
postgres =&gt;\q
postgres@python-postgresql $ exit
leo@python-postgresql $ sudo vim /etc/postgresql/9.3/main/pg_hba.conf
#Modify peer to md5
local md5
local md5
# IPv4 local connections:
host all all
127.0.0.1/32 md5
# IPv6 local connections:
host all all
::1/128 md5
leo@python-postgresql $ sudo service postgresql restart
Psycop2

Psycop2 is  a PostgreSQL adapter for python. You don’t need Flask or SQLAlchemy to connect to PostgreSQL you can connect straight away with Psycop2.

Install Psycopg2

leo@python-postgresql $  virtualenv -p /usr/bin/python3.4 venv-3.4
leo@python-postgresql $ source venv-3.4/bin/activate
(venv-3.4)leo@python-postgresql $ pip install psycopg2

Psycop2 sample code

(venv-3.4)leo@python-postgresql $ gedit psycop2_example.py
import psycopg2
import sys
db_con = None
try:
#Create a database session
db_con = psycopg2.connect(database='yourdbname', user='yourusername', password='yourpassword')
#Create a client cursor to execute commands
cursor = db_con.cursor()
cursor.execute("CREATE TABLE customers (id SERIAL PRIMARY KEY, name VARCHAR age INTEGER);")
#The variables placeholder must always be a %s, psycop2 will automatically convert the values to SQL literal
cursor.execute("INSERT INTO customers ( name, AGE) VALUES (%s, %s)",("leo", 26))
db_con.commit()
cursor.execute("SELECT * FROM customers)
print(cursor.fetchone())
except psycopg2.DatabaseError as e:
print ('Error %s' % e &nbsp;&nbsp;&nbsp;)
sys.exit(1)
finally:
if db_con:
db_con.close()
Running the script will give you

(venv-3.4)leo@python-postgresql $ python psycop2_example.py
(1,  'leo', 26)

However, if you use only Psycop2 you will need to write a lot of SQL statements,  Scaling your application will be a nightmare with all that SQL and python code.

Introducing SQLAlchemy

SQLAlchemy  is a SQL toolkit which gives you access to ORM's (Object Relational Mappers). Instead of writing lengthy code with SQL queries you can map your classes to database tables and operate on data with class methods like add, query etc.

Install SQLAlchemy on Ubuntu

(venv-3.4)leo@python-postgresql $ pip install SQLAlchemy
SQLAlchemy PostgreSQL sample code
(venv-3.4)leo@python-postgresql $ gedit sqlalchemy_example.py
from sqlalchemy import create_engine, Column, Integer, String
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemey.orm import sessionmaker
#Create a DBAPI connection
engine = create_engine('postgresql://leo:qwedsa@localhost:5432/yourdbname')
#Declare an instance of the Base class for mapping tables
Base = declarative_base()
#Map a table to a class by inheriting base class
class Customer(Base):
__tablename__ = 'customer'
id = Column(Integer, primary_key=True)
 name = Column(String(1000), nullable=False)
 email = Column(String, unique=True)
def __init__(self, name, email):
 self.name = name
 self.email = email
#Create the table using the metadata attribute of the base class
Base.metadata.create_all(engine)
‘’’SQLAlchemy SESSIONS
Sessions give you access to Transactions, whereby on success you can commit the transaction or rollback one incase you encounter an error’’’
Session = sessionmaker(bind=engine)
session = Session()
#Insert multiple data in this session, similarly you can delete
customer = Customer(name='Linux', email='linux@example.com')
customer2=Customer(name='Python', email='python@example.com')
session.add(customer)
session.add(customer2)
try:
 session.commit()
#You can catch exceptions with &nbsp;SQLAlchemyError base class
except SQLAlchemyError as e:
 session.rollback()
 print (str(e))
#Get data
for customer in session.query(Customer).all():
 &nbsp;print ("name of the customer is" ,customer.name)
 &nbsp;print ("email id of the customer is" ,customer.email)
#Close the connection
engine.dispose()
Running this Script will give you

(venv-3.4)leo@python-postgresql $ python sqlalchemy_example.py
name of the customer is Linux
email id of the customer is linux@example.com
name of the customer is Python
email id of the customer is python@example.com

These examples and methods are all good if you are writing command line applications but you will need to use a web framework like  Flask and segregate your code into MVC (Models, Views and Controllers) for better scaling.

If you have never used Flask, I recommend you read this tutorial before moving on.

Install Flask, Flask-SQLAlchemy,Flask-Migrate,Flask-Script

pip install Flask-SQLAlchemyFlask-Migrate Flask-Script

Model

Models contain all the database code.

(venv-3.4)leo@python-postgresql $ gedit model.py

from flask import Flask
from flask.ext.sqlalchemy import SQLAlchemy
from sqlalchemy.sql.expression import text
from sqlalchemy.exc import SQLAlchemyError
from flask.ext.script import Manager
from flask.ext.migrate import Migrate, MigrateCommand
app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI']= 'postgresql://yourusername:yourpassword@localhost:5432/yourdbname'
app.secret_key = 'some_secret'
db = SQLAlchemy(app)
#Create Database migrations
migrate = Migrate(app, db)
manager = Manager(app)
manager.add_command('db', MigrateCommand)
#Create the Post Class
class Post(db.Model):
id = db.Column(db.Integer, primary_key=True)
 author = db.Column(db.String(255))
 title = db.Column(db.String(255),nullable=False)
 created_on=db.Column(db.TIMESTAMP,server_default=db.func.current_timestamp())
 content = db.Column(db.Text)
 published = db.Column(db.Boolean, server_default='True', nullable=False)
def __init__(self, author,title, content,published):
 self.author = author
 self.title = title
 self.content=content
 self.published=published
def add(self,post):
 db.session.add(post)
 return session_commit()
def update(self):
 return session_commit()
def delete(self,post):
 db.session.delete(post)
 return session_commit()
def &nbsp;session_commit():
 try:
 db.session.commit()
 except SQLAlchemyError as e:
 reason=str(e)
if __name__ == '__main__':
manager.run()
Controller
Controller will contain all the python code.

(venv-3.4)leo@python-postgresql $ gedit controller.py

from flask import render_template, request,flash, redirect, url_for
from model import Post, app
#READ
@app.route('/' )
def post_index():
 post = Post.query.all()
 return render_template('index.html', post=post)
#CREATE
@app.route('/add' , methods=['POST', 'GET'])
def post_add():
 if request.method == 'POST':
 post=Post(request.form['author'],request.form['title'],request.form['content'], request.form['published'])
 post_add=post.add(post)
 if not post_add:
 flash("Add was successful")
 return redirect(url_for('post_index'))
 else:
 error=post_add
 flash(error)
 return render_template('add.html')
#UPDATE
@app.route('/update/&lt;id&gt;' , methods=['POST', 'GET'])
def post_update (id):
 #Check if the post exists:
 post = Post.query.get(id)
 if post == None:
 flash("This entry does not exist in the database")
 return redirect(url_for('post_index'))
 if request.method == "POST":
 post.author=request.form['author']
 post.title = request.form['title']
 post.content = &nbsp;request.form['content']
 post.published=request.form['published']
 post_update=post.update()
 #If post.update does not return an error
 if not post_update:
 flash("Update was successful")
 return redirect(url_for('post_index'))
 else:
 error=post_update
 flash(error)
 return render_template('update.html', post=post)
#DELETE
@app.route('/delete/&lt;id&gt;' , methods=['POST', 'GET'])
def post_delete (id):
 post = Post.query.get(id)
 #Check if the post exists:
 if post == None:
 flash("This entry does not exist in the database")
 return redirect(url_for(post_index))
 post_delete=post.delete(post)
 if not post_delete:
 flash("Post was deleted successfully")
 else:
 error=post_delete
 flash(error)
 return redirect(url_for('post_index'))
if __name__ == '__main__':
app.debug=True
app.run()


Views
All your HTML/CSS and jinja code goes here.

(venv-3.4)leo@python-postgresql $ gedit templates/index.html

&lt;--! Start flash message --&gt;
{% with messages = get_flashed_messages() %}
 {% if messages %}
 &lt;ul class=flashes&gt;
 {% for message in messages %}
 &lt;li&gt;{{ message }}&lt;/li&gt;
 {% endfor %}
 &lt;/ul&gt;
 {% endif %}
{% endwith %}
&lt;--! End flash message --&gt;
&lt;div class="center row"&gt;
 &lt;div class="small-8 columns"&gt;&lt;p&gt;&lt;h4 style="display:inline"&gt;Posts &nbsp;&lt;/h4&gt;&lt;a href="{{ url_for('post_add')}}"&gt;Add new&lt;/a&gt;&lt;/p&gt;&lt;/div&gt;
&lt;/div&gt;
&lt;hr /&gt;
&lt;div class="center row"&gt;
 &lt;div class="small-12 columns"&gt;
 &lt;table&gt;
 &lt;thead&gt;&lt;tr&gt;&lt;th&gt;Title&lt;/th&gt;&lt;th&gt;Author&lt;/th&gt;&lt;th&gt;Content&lt;/th&gt;&lt;th&gt;Published&lt;/th&gt;&lt;th&gt;Actions&lt;/th&gt;&lt;/tr&gt;&lt;/thead&gt;
 &lt;tbody&gt;
 &lt;tr&gt;
 {% for post in post %}
 &lt;td&gt;{{ post.title }}&lt;/td&gt;
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&lt;td&gt;{{ post.author }}&lt;/td&gt;
 &lt;td&gt;{{ post.content|safe }}&lt;/td&gt;
 &lt;td&gt;{{ post.published }}&lt;/td&gt;
 &lt;td&gt;&lt;a href="{{ url_for('post_update', id=post.id) }}"&gt;Edit&lt;/a&gt;&lt;/td&gt;
 &lt;td&gt;&lt;a href="{{ url_for('post_delete', id=post.id) }}"&gt;Delete&lt;/a&gt;&lt;/td&gt;
 &lt;/tr&gt;
 {% endfor %}
 &lt;/tbody&gt;
 &lt;/table&gt;
 &nbsp;&nbsp;&lt;/div&gt;
&lt;/div&gt;
(venv-3.4)leo@python-postgresql $ gedit templates/add.html

&lt;--! Start flash message --&gt;
{% with messages = get_flashed_messages() %}
 {% if messages %}
 &lt;ul class=flashes&gt;
 {% for message in messages %}
 &lt;li&gt;{{ message }}&lt;/li&gt;
 {% endfor %}
 &lt;/ul&gt;
 {% endif %}
{% endwith %}
&lt;--! End flash message --&gt;
&lt;!--Tiny MCE --&gt;
&lt;script src="//tinymce.cachefly.net/4.1/tinymce.min.js"&gt;&lt;/script&gt;
&lt;script&gt;tinymce.init({paste_data_images: true, paste_as_text: true,selector: '#content', plugins: ["image","code"] });&lt;/script&gt;
&lt;!--Tiny MCE end --&gt;
&lt;div class="center row"&gt;
 &lt;div class="large-12 columns"&gt;
&lt;fieldset&gt;
&lt;form action="" method="POST"&gt;
&lt;label&gt;Title &nbsp;&lt;input type="text" name="title" required/&gt;&lt;/label&gt;&lt;br&gt;
 &lt;label&gt;Author &nbsp;&lt;input type="text" name="author" &nbsp;required/&gt;&lt;/label&gt;&lt;br&gt;
 &lt;label&gt;Body&lt;textarea type="text" id="content" name="content" rows="20" &gt;&lt;/textarea&gt;&lt;/label&gt;
 &lt;label&gt;Published
 &lt;select name="published"&gt;
 &lt;option value="True"selected&gt;Yes&lt;/option&gt;
 &lt;option value="False"&gt;No&lt;/option&gt;
 &lt;/select&gt;&lt;/label&gt;
 &lt;button class="button " &nbsp;type="submit"&gt;Add&lt;/button&gt;
 &lt;/form&gt;
 &lt;/fieldset&gt;
 &lt;/div&gt;
 &lt;/div&gt;


(venv-3.4)leo@python-postgresql $ gedit templates/update.html


&lt;--! Start flash message --&gt;
{% with messages = get_flashed_messages() %}
 {% if messages %}
 &lt;ul class=flashes&gt;
 {% for message in messages %}
 &lt;li&gt;{{ message }}&lt;/li&gt;
 {% endfor %}
 &lt;/ul&gt;
 {% endif %}
{% endwith %}
&lt;!--Tiny MCE --&gt;
&lt;script src="//tinymce.cachefly.net/4.1/tinymce.min.js"&gt;&lt;/script&gt;
&lt;script&gt;tinymce.init({paste_data_images: true, paste_as_text: true,selector: '#content', plugins: ["image","code"] });&lt;/script&gt;
&lt;!--Tiny MCE end --&gt;
&lt;div class="center row"&gt;
 &lt;div class="large-12 columns"&gt;
 &lt;fieldset&gt;
&lt;form action="" method="POST"&gt;
 &lt;input type="text" name="author" value="{{post.author}}" required/&gt;
 &lt;label&gt;Title&lt;input type="text" name="title" value="{{post.title}}" required/&gt;&lt;/label&gt;
 &lt;label&gt;Body&lt;textarea type="text" id="content" name="content" rows="20"&gt;{{post.content}}&lt;/textarea&gt;&lt;/label&gt;
 &lt;label&gt;Published
 &lt;select name="published"&gt;
 {% if post.published == "True" %}
 &lt;option value="True"selected&gt;Yes&lt;/option&gt;
 &lt;option value="False"&gt;No&lt;/option&gt;
 {% else %}
 &lt;option value="True"&gt;Yes&lt;/option&gt;
 &lt;option value="False"selected&gt;No&lt;/option&gt;
 {% endif %}
 &lt;/select&gt;
 &lt;/label&gt;
 &lt;button class="button " &nbsp;type="submit"&gt;Update&lt;/button&gt;
 &lt;/form&gt;
 &lt;/fieldset&gt;
 &lt;/div&gt;
&lt;/div&gt;
Run the application

python controller.py
* Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)

* Restarting with stat

When you resolve http://127.0.0.1:5000/, you should be able to create, read, update and delete posts as shown in the video below.



You should also read my other articles about Database Relationships

One to Many Relationships with SQLAlchemy

Many to Many Relationships with SQLAlchemy

And in case you need to build a Database Driven API with Authentication then you should read:

Building Database Driven API's in Python with Flask-Restful and SQLAlchemy

Token Based Authentication in Python with Flask

 

 
