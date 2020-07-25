In my earlier post I described how to create  a One to Many relationship with Flask-SQLAlchemy . In this post I will describe how to create a Many to Many relationship with Flask-SQLAlcehmy .

Defining a Many to Many relationship.

A  Many to Many Relationship between two entities can be defined as one in which a parent entity can have many children and vice versa. Think of the parent entity as a Post and the child as Tags. If I were to define a Many to Many relationship between them then I can say that one Post can have many Tags and one Tag can have many Posts.

To create this relationship with Flask-SQLAlchemy between  two entities lets say a Posts table  and a Tags table you will need to add another table which will consists of  two Foreign keys, one Foreign key to the Posts  table and  one Foreign key to Tags table and then define the relationship on on either of two tables.

Example code to define a Many to Many relationship.


#https://github.com/Leo-g/flask-blog/blob/master/app/models.py
from flask import Flask
from flask.ext.sqlalchemy import SQLAlchemy
app = Flask(__name__)
SQLALCHEMY_DATABASE_URI='postgresql://username:password@localhost:5432/database'
db = SQLAlchemy(app)
relationship_table=db.Table('relationship_table', 
                             db.Column('post_id', db.Integer,db.ForeignKey('posts.id'), nullable=False),
                             db.Column('tags_id',db.Integer,db.ForeignKey('tags.id'),nullable=False),
                             db.PrimaryKeyConstraint('post_id', 'tags_id') )
class Posts(db.Model):
  id = db.Column(db.Integer, primary_key=True)
  title = db.Column(db.String(255),nullable=False)
  content = db.Column(db.Text)
  tags=db.relationship('Tags', secondary=relationship_table, backref='posts' )  
class Tags(db.Model):
     id=db.Column(db.Integer, primary_key=True)
     name=db.Column(db.String, unique=True, nullable=False)
     description=db.Column(db.Text)   
In the above  example code I have defined a Many to Many relationship on the Posts table with  “tags=db.relationship('Tags', secondary=relationship_table, backref='posts' )”.

Explanation of the arguments:

Tags: Is the child table which I want to create the relationship with.

secondary=relationship_table: You will need to use the secondary argument to specify the associative table(relationship_table).

backref='posts': The backref argument will create a posts attribute on the child table.

I have also defined a Composite Primary Key on the  associating table via “db.PrimaryKeyConstraint('post_id', 'tags_id')”, I have done this as I want a combination of the two foreign keys to be unique.

You may have also noticed that I have created the associating table (relationship_table) with the Table class instead of the Model class. This is how Flask-SQLAlchemy recommends you create an associating table. However if you need to map the table to a class then you can use the mapper function described here.

Example code for adding or removing a  Many to Many relationship.

I can get a list of tags via post.tags and then add or delete tags via post.tags.append(tag) or post.tags.remove(tags).


#https://github.com/Leo-g/flask-blog/blob/master/app/controller.py
tag_ids = [3,4,5]
post_ids = [1,2]
for id in post_ids:
   post=Posts.query.get_or_404(id)
   for id in tag_ids:
      tag=Tags.query.get(id)
 if tag is not None:
 #Add an association
       post.tags.append(tag)
 #OR Delete an association
 post.tags.delete(tag)
      
db.session.commit()
Similarly, you can also use tag.posts.append() and tag.posts.remove() to add or delete associations.

Working Demo of the code with a Posts table and a Terms table that consists of Tags.


The complete code is available on here, let me know if you have any questions or suggestions.
