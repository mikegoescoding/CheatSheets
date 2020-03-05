# SETUP VIRTUAL ENVIRONMENT:

(examples for user creation / login / posting + receiving messages db)

## Install dependencies:

terminal cmd:

```python
pipenv install flask flask-sqlalchemy flask-cors flask-bcrypt flask-heroku gunicorn psycopg2-binary flask-marshmallow marshmallow-sqlalchemy
```

dependencies used:

> - flask
> - flask-sqlalchemy
> - flask-cors
> - flask-bcrypt
> - flask-heroku
> - gunicorn
> - psycopg2-binary
> - flask-marshmallow
> - marshmallow-sqlalchemy

---

## Enter the virtual env shell:

terminal cmd:
`pipenv shell`

---

## Import libraries in app.py:

```python
from flask import Flask, request, jsonify
from flask_sqlalchemy import SQLAlchemy
from flask_cors import CORS
from flask_heroku import Heroku
from flask_marshmallow import Marshmallow
from flask_bcrypt import Bcrypt
import os
```

---

## Initialize the app:

```python
app = Flask(__name__)

basedir = os.path.abspath(os.path.dirname(__file__))
app.config["SQLALCHEMY_DATABASE_URI"] = "sqlite:///" + os.path.join(basedir, "app.sqlite")

db = SQLAlchemy(app)
ma = Marshmallow(app)
bcrypt = Bcrypt(app)
heroku = Heroku(app)
CORS(app)


if __name__ == "__main__":
    app.debug = True
    app.run()
```

---

## Setup Class tables:

```python
# Example db table for User class, below (for user login info)
class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(30), nullable=False, unique=True)
    password = db.Column(db.String(), nullable=False, unique=False)
    email = db.Column(db.String()) #Since this is not a nullable field, the user does NOT have to provide an email (it is optional)

    def __init__(self, username, password, email):
        self.username = username
        self.password = password
        self.email = email

# Corresponding Marshmallow class for above User class, below
class UserSchema(ma.Schema):
    class Meta:
        fields = ("id", "username", "password", "email")

user_schema = UserSchema()
users_schema = UserSchema(many=True)



# Example db table for Post class, below (for message functionality)
class Post(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    content = db.Column(db.String(), nullable=False)
    color = db.Column(db.String(), nullable=False)
    userID = db.Column(db.Integer(), nullable=False)

    def __init__(self, content, color, userID):
        self.content = content
        self.color = color
        self.userID = userID

# Corresponding Marshmallow class for above Post class, below
class PostSchema(ma.Schema):
    class Meta:
        fields = ("id", "content", "color", "userID")

post_schema = PostSchema()
posts_schema = PostSchema(many=True)
```

---

## Create Sqlite DB:

**!must still be in shell!**

1. enter python repl:  
   terminal cmd: `python`

2. import db:  
   terminal cmd: `from app import db`

3. create db file:  
   terminal cmd: `db.create_all()`

---

## Build the End Points:

#### Current code:

```python
from flask import Flask, request, jsonify
from flask_sqlalchemy import SQLAlchemy
from flask_cors import CORS
from flask_heroku import Heroku
from flask_marshmallow import Marshmallow
from flask_bcrypt import Bcrypt
import os

app = Flask(__name__)

basedir = os.path.abspath(os.path.dirname(__file__))
app.config["SQLALCHEMY_DATABASE_URI"] = "sqlite:///" + os.path.join(basedir, "app.sqlite")

db = SQLAlchemy(app)
ma = Marshmallow(app)
bcrypt = Bcrypt(app)
heroku = Heroku(app)
CORS(app)


# DATABASE TABLES SETUP:
# Example db table for User class, below (for user login info)
class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(30), nullable=False, unique=True)
    password = db.Column(db.String(), nullable=False, unique=False)
    email = db.Column(db.String()) #Since this is not a nullable field, the user does NOT have to provide an email (it is optional)

    def __init__(self, username, password, email):
        self.username = username
        self.password = password
        self.email = email

# Corresponding Marshmallow class for above User class, below
class UserSchema(ma.Schema):
    class Meta:
        fields = ("id", "username", "password", "email")

user_schema = UserSchema()
users_schema = UserSchema(many=True)


# Example db table for Post class, below (for message functionality)
class Post(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    content = db.Column(db.String(), nullable=False)
    color = db.Column(db.String(), nullable=False)
    userID = db.Column(db.Integer(), nullable=False)

    def __init__(self, content, color, userID):
        self.content = content
        self.color = color
        self.userID = userID

# Corresponding Marshmallow class for above Post class, below
class PostSchema(ma.Schema):
    class Meta:
        fields = ("id", "content", "color", "userID")

post_schema = PostSchema()
posts_schema = PostSchema(many=True)



#BUILD END POINTS:

# create a new user
@app.route("/users/post", methods=["POST"])
def create_user():
    if request.content_type != "application/json":
        return jsonify("Error: Data must be sent as JSON.")
    post_data = request.get_json()
    username = post_data.get("username")
    password = post_data.get("password")
    email = post_data.get("email")

    encrypted_password = bcrypt.generate_password_hash(password).decode("utf8")

    record = User(username, encrypted_password, email)
    db.session.add(record)
    db.session.commit()

    return jsonify("User Created.")

# get all users
@app.route("/users/get", methods=["GET"])
def get_all_users():
    all_users = db.session.query(User).all()
    return jsonify(users_schema.dump(all_users))

# get user by ID
@app.route("/users/get/by_id/<id>", methods=["GET"])
def get_user_by_id(id):
    user = db.session.query(User).filter(User.id == id).first()
    return jsonify(user_schema.dump(user))

# get user by username
@app.route("/users/get/by_username/<username>", methods=["GET"])
def get_user_by_username(username):
    user = db.session.query(User).filter(User.username == username).first()
    return jsonify(user_schema.dump(user))

# user login credential verification
@app.route("/users/verification", methods=["POST"])
def verify_user():
    if request.content_type != "application/json":
        return jsonify("Error: Data must be sent as JSON.")

    post_data = request.get_json()
    username = post_data.get("username")
    password = post_data.get("password")

    encrypted_password = db.session.query(User.password).filter(User.username == username).first()

    if encrypted_password is None:
        return jsonify("User NOT verified: Username")

    password_check = bcrypt.check_password_hash(encrypted_password[0], password)

    if password_check == False:
        return jsonify("User NOT verified: Password")

    return jsonify("User Verified")


# get all posts by a specific user
@app.route("/users/posts/by_user_id/<user_id>", methods=["GET"])
def get_all_posts_by_user(user_id):
    all_posts = db.session.query(Post).filter(Post.userID == user_id).all()
    return jsonify(posts_schema.dump(all_posts))

#
@app.route("/users/posts/by_username/<username>", methods=["GET"])
def get_all_posts_by_user_with_username(username):
    user_id = db.session.query(User.id).filter(User.username == username).first()[0]

    all_posts = db.session.query(Post).filter(Post.userID == user_id).all()
    return jsonify(posts_schema.dump(all_posts))


# post a new message
@app.route("/posts/post_msg", methods=["POST"])
def create_post():
    if request.content_type != "application/json":
        return jsonify("Error: Data must be sent as JSON.")

    post_data = request.get_json()
    content = post_data.get("content")
    color = post_data.get("color")
    userID = post_data.get("userID")

    record = Post(content, color, userID)
    db.session.add(record)
    db.session.commit()

    return jsonify("Post Created")

# get all messages
@app.route("/posts/get_msg", methods=["GET"])
def get_all_posts():
    all_posts = db.session.query(Post).all()
    return jsonify(posts_schema.dump(all_posts))

# get single message by message id
@app.route("/posts/get_msg/<id>", methods=["GET"])
def get_post_by_id(id):
    post = db.session.query(Post).filter(Post.id == id).first()
    return jsonify(post_schema.dump(post))



if __name__ == "__main__":
    app.debug = True
    app.run()

```
