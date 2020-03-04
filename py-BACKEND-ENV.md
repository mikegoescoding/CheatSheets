#SETUP VIRTUAL ENVIRONMENT:

## Install dependencies

```
pipenv install flask flask-sqlalchemy flask-cors flask-bcrypt flask-heroku gunicorn psycopg2-binary flask-marshmallow marshmallow-sqlalchemy
```

-> flask
-> flask-sqlalchemy
-> flask-cors
-> flask-bcrypt
-> flask-heroku
-> gunicorn
-> psycopg2-binary
-> flask-marshmallow
-> marshmallow-sqlalchemy

---

## Enter the virtual env shell

`pipenv shell`

---

## Import libraries in app.py

-> from flask import Flask, request, jsonify
-> from flask_sqlalchemy import SQLAlchemy
-> from flask_cors import CORS
-> from flask_heroku import Heroku
-> from flask_marshmallow import Marshmallow
-> from flask_bcrypt import Bcrypt
-> import os

---

## Initialize the app

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
