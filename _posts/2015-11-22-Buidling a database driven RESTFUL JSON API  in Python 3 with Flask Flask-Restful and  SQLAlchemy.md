In this tutorial I will be describing how you can quickly build a simple database driven Create,  Read,  Update and Delete JSON API in Python 3 with  Flask and it’s extensions namely Flask-Restful and Flask-SQLAlchemy.

Before we start, here's what you should know about REST.

What is REST?

REST is  a programming style which describes how data should be transferred between two systems on the Internet.  The key principles of REST are as follows:

Client–server : There must be a clear separation between client and server such that clients are not concerned with data storage and servers are not concerned with the User Interface.

Stateless: State information is not stored on the server.  A client  request must contain all information like session to service the request.

Cacheable : The server must indicate if  request data is cacheable.

Layered system: To improve performance instead of an API server intermediaries like load balancers must be able to serve requests.

Uniform interface : The communication method between the client and server must be uniform.

Code on demand (optional) : Servers can provide executable code for the client to download and execute.

It is also important to note that REST is not a standard but encourages the use of standards such as the JSON API Specification at jsonapi.org which we will be using to format JSON data in this tutorial.

Installation

The first step is to install the required dependencies. The following are what you need to install:

Flask-Restful: will be used to define our API endpoints and bind them to Python Classes.

Flask-SQLAlchemy: will be used to define our database models using the underlying SQLAlchmey ORM framework.

Marshmallow: is used to Serailize/Deserialize JSON data to python objects and vice versa. We will also use it for validation.
Marshmallow-jsonapi:   Is a modified version of Marshmallow which will produce JSON API-compliant data.
Psycopg2 : Python database driver for PostgresSQL, if you are using MySQL then you can install PyMySQL
Flask-Migrate and Flask-Script: will be used for database migrations.

 pip install Flask Flask-Restful Flask-SQLAlchemy marshmallow marshmallow-jsonapi psycopg2 Flask-Migrate Flask-Script 
Defining Database Models  and Validation Schema with Flask-SQLAlchemy and Marshmallow_jsonapi

Flask-SQLAlchemy's provides access to the SQLAlchemy Object Relation Mapper (ORM) . The ORM object makes it easy to bind tables to Classes and columns to Class objects as you should see in the following code. This enables you to interact with the Database using Python Class Objects instead of long SQL queries.

To validate and format JSON as per the JSON API specification at jsonapi.org, I have used Marshmallow and Marshmallow-jsonapi respectively. In the view code you will see that Marshmallow makes it easy to serialize python objects to JSON formatted data. Marshmallow also allows you to validate data with it's schema class as follows:


#models.py
from marshmallow_jsonapi import Schema, fields
from marshmallow import validate
from flask.ext.sqlalchemy import SQLAlchemy
from sqlalchemy.exc import SQLAlchemyError
#Create an instance of SQLAlchemy and Flask
app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = "postgresql://{DB_USER}:{DB_PASS}@{DB_ADDR}/{DB_NAME}".format(DB_USER="pg_db_username", DB_PASS="pg_db_password", DB_ADDR="pg_db_hostname", DB_NAME="pg_db_name")
db = SQLAlchemy()
#Class to add, update and delete data via SQLALchemy sessions
class CRUD():
def add(self, resource):
 db.session.add(resource)
 return db.session.commit()
def update(self):
 return db.session.commit()
def delete(self, resource):
 db.session.delete(resource)
 return db.session.commit()
#Our Users Models, which will inherit Flask-SQLAlchemy Model class and the CRUD class defined above
class Users(db.Model, CRUD):
 id = db.Column(db.Integer, primary_key=True)
 email = db.Column(db.String(250), unique=True, nullable=False)
 name = db.Column(db.String(250), nullable=False)
 creation_time = db.Column(db.TIMESTAMP, server_default=db.func.current_timestamp(), nullable=False)
 is_active = db.Column(db.Boolean, server_default="false", nullable=False)
def __init__(self, email, name, is_active):
 self.email = email
 self.name = name
 self.is_active = is_active
class UsersSchema(Schema):
 not_blank = validate.Length(min=1, error='Field cannot be blank')
 id = fields.Integer(dump_only=True)
 email = fields.Email(validate=not_blank)
 name = fields.String(validate=not_blank)
 is_active = fields.Boolean()
#self links
 def get_top_level_links(self, data, many):
  if many:
    self_link = "/users/"
  else:
    self_link = "/users/{}".format(data['id'])
  return {'self': self_link}
class Meta:
 type_ = 'users'
Database Migrations

You can use  Flask-Migrate and Flask-Script to create your tables with the following script. This will import the SQLAlchemy object we created in our models code which contains the Database string. This is sample code hence I have added the Database configuration in the models code, but you should generally store all configuration including database in a separate config file.


#migrate.py
from flask.ext.script import Manager
from flask.ext.migrate import Migrate, MigrateCommand
from models import db

migrate = Migrate(app, db)
manager = Manager(app)
manager.add_command('db', MigrateCommand)
if __name__ == '__main__':
 manager.run()


Flask-Migrate and Script provide access to command line arguments through which you can run your migrations and create your tables.

python migrate.py init
python migrate.py db migrate
python migrate.py db upgrade


Defining your API endpoints with Flask-Restful

With Flask-Restful's Resource Class you should be able to create API endpoints as follows:

#The following class will be used for GET and POST requests to /users
#Complete code at https://github.com/Leo-G/Flask-SQLALchemy-RESTFUL-API/blob/master/app/users/views.py
from models import Users, UsersSchema, db
from flask_restful import Api, Resource
#Declare an instance of the UsersSchema and Flask-Restful Api class
schema = UsersSchema()
api = Api()
class UsersList(Resource):
#Function for a GET request
 def get(self):
 #Query the database and return all users
 users_query = Users.query.all()
 #Serialize the query results in the JSON API format
 results = schema.dump(users_query, many=True).data
 return results
 
 #Function for a POST request
 def post(self):
 raw_dict = request.get_json(force=True)
 user_dict = raw_dict['data']['attributes']
 
 try:
  #Validate the data or raise a Validation error if incorrect
  schema.validate(user_dict)
  #Create a User object with the API data recieved
  user = Users(user_dict['email'], user_dict['name'],user_dict['is_active'], user_dict['role'])
  #Commit data
  user.add(user)
  query = Users.query.get(user.id)
  results = schema.dump(query).data
  return results, 201
 
 except ValidationError as err:
  resp = jsonify({"error": err.messages})
  resp.status_code = 403
  return resp
 
 except SQLAlchemyError as e:
  db.session.rollback()
  resp = jsonify({"error": str(e)})
  resp.status_code = 403
  return resp
 
#The following class is used when a GET,PATCH,DELETE HTTP request is sent to /user/{user_id}
class UsersUpdate(Resource):
#http://jsonapi.org/format/#fetching
 def get(self, id):
  user_query = Users.query.get_or_404(id)
  result = schema.dump(user_query).data
  return result
 
#http://jsonapi.org/format/#crud-updating
 def patch(self, id):
  user = Users.query.get_or_404(id)
  raw_dict = request.get_json(force=True)
  user_dict = raw_dict['data']['attributes']
 try:
  for key, value in user_dict.items():
    schema.validate({key:value})
    setattr(user, key, value)
    user.update()
  return self.get(id)
 except ValidationError as err:
  resp = jsonify({"error": err.messages})
  resp.status_code = 401
  return resp
 except SQLAlchemyError as e:
  db.session.rollback()
  resp = jsonify({"error": str(e)})
  resp.status_code = 401
  return resp
 
#http://jsonapi.org/format/#crud-deleting
 def delete(self, id):
  user = Users.query.get_or_404(id)
  try:
   delete = user.delete(user)
   response = make_response()
   response.status_code = 204
   return response
 except SQLAlchemyError as e:
  db.session.rollback()
  resp = jsonify({"error": str(e)})
  resp.status_code = 401
  return resp
#Define API endpoints
#Map classes to API endpoints
api.add_resource(UsersList, 'user.json')
api.add_resource(UsersUpdate, 'user/<span style="color: #183691;"><int:id>.json</span>')


In the above code with Flask-Restful I have create two classes. One (UsersList) to serve GET and POST requests for all users. The other (UsersUpdate) to serve GET,PATCH and DELETE requests for a single user.

When you run the above code you should be able to send HTTP API requests to the following Resource URL's



HTTP Method	URL	Results
GET	http://localhost:5000/api/v1/users.json	Returns a list of all users
POST	http://localhost:5000/api/v1/users.json	Creates a user and returns with user id
GET	http://localhost:5000/api/v1/users/<user_id>.json	Returns user details for the given user id if the it exists
PATCH	http://localhost:5000/api/v1/users/<user_id>.json	Updates the user details if the user exists and returns with the updated results
DELETE	http://localhost:5000/api/v1/users/<user_id>.json	Deletes the user and returns a 204 on success as per the JSON API spec

The complete code is available at https://github.com/Leo-G/Flask-SQLALchemy-RESTFUL-API, If you need Authentication then I recommend you use the Secure Token based Authentication where Auth tokens are stored on the client side,   read my turorial for more details at JSON web token authentication with Flask. You can also use a REST client like Angularjs's $resource service to make API calls.

Update : I wrote a python script which will build the API for you along with the views and models. You can download it from https://github.com/Leo-G/Flask-Scaffold

Here’s a video of  how you can Download, Install and Run the code.



 

Reference

https://github.com/WhiteHouse/api-standards

https://www.nsa.gov/ia/_files/support/guidelines_implementation_rest.pdf

http://flask-restful.readthedocs.org/

https://pythonhosted.org/Flask-SQLAlchemy/quickstart.html#quickstart

https://github.com/marshmallow-code/marshmallow-jsonapi

https://marshmallow.readthedocs.org

http://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm
