While building database applications with Flask-SQLAlchemy, you will come across a situation where you need to define a relationship between two database tables. While creating blogging application I came across such a situation where I had to define a One to Many Relationship between a blog post (Post Table) and post comments(Comments Table).

In order to define a One to Many Relationships with Flask-SQLAlchemy you need to first add a Foreign key on the Comments or Child table referencing the Post or Parent table. You can then define the relationship, by adding  the relationship() function on the Parent or Post table, as referencing a collection of comments represented by the Comments table.

Example Code


#models.py
#Complete code at https://github.com/Leo-g/flask-blog/blob/master/app/models.py
from flask import Flask
from flask.ext.sqlalchemy import SQLAlchemy
app = Flask(__name__)
db = SQLAlchemy(app)
class Post(db.Model):
 id = db.Column(db.Integer, primary_key=True)
 author = db.Column(db.String(255))
 title = db.Column(db.String(255),nullable=False)
 content = db.Column(db.Text)
#Defining One to Many relationships with the relationship function on the Parent Table
 comments = db.relationship('Comments', backref="post", cascade="all, delete-orphan" , lazy='dynamic')
class Comments(db.Model):
id = db.Column(db.Integer, primary_key=True)
 author = db.Column(db.String(128),nullable=False)
 website = db.Column(db.String(255))
 content = db.Column(db.Text, nullable=False)
#Defining the Foreign Key on the Child Table
 post_id = db.Column(db.Integer, db.ForeignKey('post.id'))
I have given the relationship() function the following arguments:


'Comments' : The referencing Child table
backref="post" : This argument adds a post attribute on the Comments table, so you can access a Post via the Comments Class as Comments.post.
cascade="all, delete-orphan": This will delete all comments when a referenced post is deleted.
“lazy=’dynamic’”: This will return a query object which you can refine further like if you want to add a limit etc.

In order to access the comments for a particular post you need to add the below code to your controller and template respectively.


#controller.py
#Complete code at https://github.com/Leo-g/flask-blog/blob/master/app/controller.py
from flask import render_template
from models import Post
@app.route('/post/<id>' )
def single_post(id):
 post = Post.query.get(id)
 if post == None:
 return abort(404)
 return render_template('single_post.html', post=post)
Template


#single_post.html
<!--POST START -->
{{ post.title }}
{{ post.content|safe }}
<!--POST END -->
<!--COMMENTS START -->
{%for comment in post.comments.all()%}
 {{comment.author}} </br>
 {{comment.content}}
{%endfor%}
<!--COMMENTS END→
The object returned by post.comments.all() is a python list. You can add comments to this list as follows:


>>>post=Post.query.get(post_id)
comment=Comments(author='Leo',website='flask.com',content='flask-sqlalchemy example',post_id=post_id)
>>> post.comments.append(comment)
>>> db.session.add(post)
>>> db.session.commit()


This is just a sample of the code for better understanding the complete code is available at the https://github.com/Leo-g/flask-blog/.

If you need create a Many to Many Relationship then read Many to Many relationships with Flask-SQLAlchemy. You can also create a database driven CRUD app  complete with validations and tests  with Flask-Scaffold
Working demo of the code


Feel free to fork or let me know about your suggestions or you.

 

For database driven API's or JSON toke based authentication I recommend you read my latest tutorials at

Building  a Database Driven API with Flask-RestFUl and SQLAlchemy

JSON web Token Authentication with Flask and Angularjs

Ref
http://docs.sqlalchemy.org/en/latest/orm/backref.html
http://docs.sqlalchemy.org/en/latest/orm/cascades.html#save-update
http://docs.sqlalchemy.org/en/latest/orm/relationships.html
http://docs.sqlalchemy.org/en/latest/orm/basic_relationships.html
