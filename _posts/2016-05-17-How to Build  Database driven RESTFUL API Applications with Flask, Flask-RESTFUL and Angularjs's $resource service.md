In this tutorial I will describe how you can Build a RESTFUL API CRUD(Create, Read, Update and Delete) application with the Flask Python Micro-framework, and the Angularjs $resource Service. I be will using the Flask-RESTFUL API extension to create an API backend and then use Angularjs's $resource service to communicate with it.

Prerequisites
This Tutorial is for users who are familiar with the Flask Python  and Angularjs Javascript Web frameworks.

Minimum System/Software Requirements that you need to have are:

Ubuntu Linux 14.04
Python 3.4
Flask 0.10
Angularjs 1.4
PostgreSQL 9+ or MySQL/Mariadb 5+

Part 1: Installing and Configuring a Flask RESTFUL API Server.
The first step is to download Flask, Flask-RESTFUL and Flask-SQLAlchemy along with their dependencies into a virtual environment and configure them to connect to your database.

#Create a Flask project directory
mkdir -p myproject/app
cd myproject
#create and activate the virtual environment
virtualenv -p /usr/bin/python3.4 venv-3.4
source venv-3.4/bin/activate
#Install Flask, and FLask-SQLAlchemy along with PostgreSQL and MySQL database drivers
pip install flask  flask-sqlalchemy marshmallow-jsonapi marshmallow psycopg2  pymysql flask-restful
#You can also clone the requirements file from #https://github.com/Leo-G/Flask-Scaffold/blob/master/requirements.txt
#Create a config file and add your database string
vim config.py

# DATABASE SETTINGS
pg_db_username = 'postgres'
pg_db_password = ''
pg_db_name = 'flask-angularjs-api’'
pg_db_hostname = 'localhost'
# MYSQL
mysql_db_username = 'root'
mysql_db_password = ''
mysql_db_name = 'flask-angularjs-api’'
mysql_db_hostname = 'localhost'
SQLALCHEMY_TRACK_MODIFICATIONS = True
# PostgreSQL
# Comment the below string if you are using MySQL or MariaDB
SQLALCHEMY_DATABASE_URI = "postgresql://{DB_USER}:{DB_PASS}@{DB_ADDR}/{DB_NAME}".format(DB_USER=pg_db_username,
 DB_PASS=pg_db_password,
 DB_ADDR=pg_db_hostname,
 DB_NAME=pg_db_name)
# MySQL
# Uncomment the below string if you are using MySQL or MariaDB
"""SQLALCHEMY_DATABASE_URI = "mysql+pymysql://{DB_USER}:{DB_PASS}@{DB_ADDR}/{DB_NAME}".format(DB_USER=mysql_db_username,
 DB_PASS=mysql_db_password,
 DB_ADDR=mysql_db_hostname,
 DB_NAME=mysql_db_name)"""
Next using the Flask-SQLAlchemy ‘Model Class ‘ create Table and Column Definitions as follows:

vim app/models.py

from marshmallow_jsonapi import Schema, fields
from marshmallow import validate
from flask.ext.sqlalchemy import SQLAlchemy
db = SQLAlchemy()
class Users(db.Model):
 id = db.Column(db.Integer, primary_key=True)
 name = db.Column(db.String(250), unique=True, nullable=False)
 address = db.Column(db.Text, nullable=False)
 age = db.Column(db.Integer, nullable=False)
 profession = db.Column(db.String(250), nullable=False)
 website = db.Column(db.String(250), nullable=False)
def __init__(self, name, address, age, profession, website):
 self.name = name
 self.address = address
 self.age = age
 self.profession = profession
 self.website = website
 
def add(self, resource):
 db.session.add(resource)
 return db.session.commit()
def update(self):
 return db.session.commit()
def delete(self, resource):
 db.session.delete(resource)
 return db.session.commit()
class UsersSchema(Schema):
 not_blank = validate.Length(min=1, error='Field cannot be blank')
 # add validate=not_blank in required fields
 id = fields.Integer(dump_only=True) 
 name = fields.String(validate=not_blank)
 address = fields.String(validate=not_blank)
 age = fields.Integer(required=True)
 profession = fields.String(validate=not_blank)
 website = fields.String(validate=not_blank)
# self links
def get_top_level_links(self, data, many):
 if many:
 self_link = "/users/"
 else:
 self_link = "/users/{}".format(data['id'])
 return {'self': self_link}
 #The below type object is a resource identifier object as per http://jsonapi.org/format/#document-resource-identifier-objects
 class Meta:
 type_ = 'users'
In the above code example I have also included the Schema class from the ‘marshmallow_jsonapi’ python module to send JSON formatted data as per the JSON specification at jsonapi.org and the validation class from the ‘marshmallow’ python module to validate Data. Refer to the links at the bottom of this article to learn more about these modules.

Next create a 'controller.py' file and define the CRUD Classes and API endpoints using the Flask-RESTful flask extension as follows:

vim app/controller.py

from flask import Blueprint, request, jsonify, make_response
from flask_restful import Api, Resource
from app.models import db,Users, UsersSchema
from sqlalchemy.exc import SQLAlchemyError
from marshmallow import ValidationError
#Initialize a Flask Blueprint,
users = Blueprint('users', __name__)
#Initialize the UserSchema we defined in models.py
schema = UsersSchema(strict=True)
#Initialize the API object using the Flask-RESTful API class
api = Api(users)
# Create CRUD classes using the Flask-RESTful Resource class
class CreateListUsers(Resource):
 
 def get(self):
 users_query = Users.query.all()
 results = schema.dump(users_query, many=True).data
 return results
def post(self):
 raw_dict = request.get_json(force=True)
 try:
 schema.validate(raw_dict)
 request_dict = raw_dict['data']['attributes']
 print(raw_dict)
 user = Users(request_dict['name'], request_dict['address'], request_dict['age'], request_dict[
 'profession'], request_dict['website'])
 user.add(user)
 # Should not return password hash
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
class GetUpdateDeleteUser(Resource):
 
 def get(self, id):
 user_query = Users.query.get_or_404(id)
 result = schema.dump(user_query).data
 return result
def patch(self, id):
 user = Users.query.get_or_404(id)
 raw_dict = request.get_json(force=True)
 try:
 schema.validate(raw_dict)
 request_dict = raw_dict['data']['attributes']
 for key, value in request_dict.items():
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
#Map classes to API enpoints
api.add_resource(CreateListUsers, '.json')
api.add_resource(GetUpdateDeleteUser, '/<int:id>.json')
The above code will first validate incoming data using the 'Marshmallow Schema Class' that was defined in our model and only then read or update data in the database. Next initialize the Flask Application with the SQLAlchemy database object.

vim app/__init__.py

from flask import Flask
# http://flask.pocoo.org/docs/0.10/patterns/appfactories/
def create_app(config_filename):
    app = Flask(__name__, static_folder='templates/static')
    app.config.from_object(config_filename)
            
    #Init Flask-SQLAlchemy
    from app.models import db
    db.init_app(app)
from app.controller import users
 app.register_blueprint(users, url_prefix='/api/v1/users')
 
 from flask import render_template, send_from_directory 
 
 @app.route('/&lt;path:filename&gt;')
 def file(filename):
 from os import path
 return send_from_directory(path.join(app.root_path, 'templates'), filename)
 
 @app.route('/')
 def index():
 return render_template('index.html')
 
 return app
In the above file I have also defined the API URL ‘/api/v1/users’. This URL is as per the WhiteHouse RESTFUL API guidelines on Github where ‘v1’ is the version number and 'users' is the resource. The CRUD API calls need to be to made to /api/v1/users.json and /api/v1/users/{id}.json for all and single user respectively.

Finally create the database tables and columns with the Flask-Migrations extension as follows:

vim migrate.py

from flask.ext.migrate import Migrate, MigrateCommand
from config import SQLALCHEMY_DATABASE_URI
from app.models import db
from flask import Flask
from flask.ext.script import Manager
from app import create_app
app = create_app('config')
migrate = Migrate(app, db)
manager = Manager(app)
manager.add_command('db', MigrateCommand)
 
if __name__ == '__main__':
 manager.run()
#run the following commands to perform the actual database migrations
Python migrate.py db init
Python migrate.py db migrate
Python migrate.py db upgrade

Run the application using Flask’s inbuilt web server as follows:

vim run.py

#!/usr/bin/env python
from app import create_app
app = create_app('config')
if __name__ == '__main__':
 app.run(host='0.0.0.0',
 port=5000,
 debug='true')
python run.py

Below is a small demo  video using the PostMan API client of the REST API calls



Part 2 : Making CRUD RESTFUL API calls with the Angularjs $Resource Service
The Angularjs $Resource Service provides an easy way to make RESTFUL API calls without having to define each method. It has the HTTP methods defined and available by default, you can also define your own method if required.

mkdir -p  app/templates/static/js/
vim app/templates/static/js/app.js

//Initialize an Angularjs Application
var app =angular.module('myApp', ['ui.router','ngResource', 'myApp.controllers', 'myApp.services', 'toaster']);
// Create a User Resource Object using the resource service
angular.module('myApp.services', []).factory('User', function($resource) {
 return $resource('api/v1/users/:id.json', { id:'@users.id' }, {
 update: {
 method: 'PATCH',
 }
 }, {
 stripTrailingSlashes: false
 });
});
// Create routes with UI-Router and display the appropriate HTML file for listing, adding or updating users
angular.module('myApp').config(function($stateProvider, $urlRouterProvider) {
 //
 // For any unmatched url, redirect to /state1
 $urlRouterProvider.otherwise("/");
 
 $stateProvider
 .state('users', { 
 // https://github.com/angular-ui/ui-router/wiki/Nested-States-and-Nested-Views
 abstract: true,
 url: '/',
 title: 'Users',
 template: '&lt;ui-view/&gt;'
 })
 .state('users.list', {
 url: 'list',
 templateUrl: 'list.html',
 controller: 'UserListController',
}).state('users.add', {
 url: 'add',
 templateUrl: 'add.html',
 controller: 'UserCreateController',
}).state('users.edit', {
 url: ':id/edit',
 templateUrl: 'update.html',
 controller: 'UserEditController',
 
 })
});
// Define CRUD controllers to make the add, update and delete calls using the User resource we defined earlier
angular.module('myApp.controllers', []).controller('UserListController', function($scope, User, $state, toaster) {
 User.get(function(data) {// Get all the users. Issues a GET to /api/v1/users.json
 
 $scope.users = [];
 angular.forEach(data.data, function(value, key)
 {
 this.user = value.attributes;
 this.user['id'] = value.id;
 this.push(this.user);
 }, $scope.users); 
 
 },
 function(error){
toaster.pop({
 type: 'error',
 title: 'Error',
 body: error,
 showCloseButton: true,
 timeout: 0
 });
 });
 $scope.deleteUser = function(selected_id) { // Delete a User. Issues a DELETE to /api/v1/users/{user-id}.json
 user = User.get({ id: selected_id});
 user.$delete({ id: selected_id},function() {
 toaster.pop({
 type: 'success',
 title: 'Success',
 body: "User deleted successfully",
 showCloseButton: true,
 timeout: 0
 });
 $state.go('users.list'); //redirect to home on a successful deletion
 $state.reload();
 
 }, function(error) {
 toaster.pop({
 type: 'error',
 title: 'Error',
 body: error,
 showCloseButton: true,
 timeout: 0
 });
 });
 };
}).controller('UserCreateController', function($scope, User, $state, toaster) {
$scope.user = new User(); //create new user. 
 
 $scope.addUser = function() {
 
 $scope.user.data.type = "users";
 $scope.user.$save(function() { // Add a User. Issues a POST to /api/v1/users.json
 toaster.pop({
 type: 'success',
 title: 'Success',
 body: "User saved successfully",
 showCloseButton: true,
 timeout: 0
 });
 
 $state.go('users.list'); // on success go back home
 }, function(error) { toaster.pop({
 type: 'error',
 title: 'Error',
 body: error,
 showCloseButton: true,
 timeout: 0
 }); 
 
 });
}
}).controller('UserEditController', function($scope, $state, $stateParams, toaster, $window, User) {
$scope.updateUser = function() { //Update the user. Issues a PATCH to /api/v1/users/{user-id}.json
$scope.user.$update({ id: $stateParams.id },function() {
 toaster.pop({
 type: 'success',
 title: 'Success',
 body: "Update was a success",
 showCloseButton: true,
 timeout: 0
 });
 
 $state.go('users.list'); // on success go back to home
 
 }, function(error) {
 toaster.pop({
 type: 'error',
 title: 'Error',
 body: error,
 showCloseButton: true,
 timeout: 0
 });
 });
 };
$scope.loadUser = function() { //Issues a GET request to /api/v1/users/{user-id}.json. Fetches a single user
 $scope.user = User.get({ id: $stateParams.id },
 function() {}, function(error) {
 toaster.pop({
 type: 'error',
 title: 'Error',
 body: error,
 showCloseButton: true,
 timeout: 0
 });
 });
 };
$scope.loadUser();
 });
//Tutorial at Techarena51.com

Along with $resource service I have also used the ‘ui-router’ module for routing.
With 'ui-router' I can define states for each of the CRUD routes except for DELETE and then use them to transition between states. You can check the demo on my Application for a live example of how they work. You can also refer to the link at the end of this article for more information

Lastly create the relevant HTML pages to Add, View and Update your Application.

vim app/templates/index.html
<!DOCTYPE html>
 <html >
 <head lang="en">
 <meta charset="utf-8">
<!-- CSS-->
 <link href="https://cdnjs.cloudflare.com/ajax/libs/angularjs-toaster/1.1.0/toaster.min.css" rel="stylesheet" />
<link href=" https://cdnjs.cloudflare.com/ajax/libs/skeleton/2.0.4/skeleton.css" rel="stylesheet" />
 <link href="static/css/my.css" rel="stylesheet" />
<!-- JS -->
 <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.4.8/angular.min.js"></script>
 <script src=" https://cdnjs.cloudflare.com/ajax/libs/angular.js/1.5.5/angular-resource.js"> </script>
 <script src="https://cdnjs.cloudflare.com/ajax/libs/angularjs-toaster/1.1.0/toaster.min.js"></script>
 <script src="https://code.angularjs.org/1.2.0/angular-animate.min.js" ></script>
 <script src="static/js/app.js"></script>
 <script src="https://cdnjs.cloudflare.com/ajax/libs/angular.js/1.5.5/angular-route.min.js"></script>
 <script src="https://cdnjs.cloudflare.com/ajax/libs/angular-ui-router/0.2.18/angular-ui-router.min.js"></script>
</head>
<body ng-app="myApp" >
<div id="main">
<div class="nav">
<ul >
<li><a ui-sref="users.list" >List</a></li>
<li><a ui-sref="users.add" >New</a></li>

 </ul>
</div>
<div ui-view> </div>

 </div>

 <toaster-container></toaster-container>
 </body>
</html>
vim app/templates/list.html
<table>
<tr>
<th>Name</th>
<th>Address</th>
<th>Age</th>
<th>Profession</th>
<th>Website</th>
</tr>
<tr ng-repeat="user in users">
<td>{{ user.name }}</td>
<td>{{ user.address }}</td>
<td>{{ user.age}}</td>
<td>{{ user.profession }}</td>
<td>{{ user.website}}</td>
<td><a ui-sref="users.edit({id:user.id})" >Edit</a> </td>
<td><a href ng-click="deleteUser(user.id)">Delete</a> </td>

 </tr>
</table>
vim app/templates/form.html

<input placeholder="Name*" type="text" id="name" name="name" ng-model="user.data.attributes.name" required>
    <input placeholder="Address*" type="text" id="address" name="address" ng-model="user.data.attributes.address" required>
    <input placeholder="Age*" type="number" id="age" name="age" ng-model="user.data.attributes.age" required>
    <input placeholder="Profession*" type="text" id="profession" name="profession" ng-model="user.data.attributes.profession" required>
    <input placeholder="Website*" type="url" id="website" name="website" ng-model="user.data.attributes.website" required>   
 <button class="button-primary" type="submit" > Save </button>
vim app/templates/add.html
<form role="form" ng-submit="addUser()" >
<div class="form" ng-include="'form.html'">
 </div>
</form>
vim app/templates/update.html
<form role="form" ng-submit="updateUser()">
<div class="form" ng-include="'form.html'"> 
 </div>

 </form>


Restart the Flask server and you should be able to view your CRUD application at http://localhost:5000

python run.py

Below is a video of a working Application



The complete code is available at https://github.com/Leo-G/Flask-Scaffold/. An easier way to build CRUD Applications with Flask and Angularjs is with Flask-Scaffold. Just provide your Database definitions in a YAML file and it will generate all the code including the valiadations for you. The instructions are at https://github.com/Leo-G/Flask-Scaffold/blob/master/README.md

You can also add Authentication to your API Application with JSON Token Based Authentication.

Reference:
https://docs.angularjs.org/api/ngResource/service/$resource
https://github.com/angular-ui/ui-router
https://github.com/WhiteHouse/api-standards
http://flask-restful-cn.readthedocs.io/en/0.3.4/
http://marshmallow.readthedocs.io/en/latest/
