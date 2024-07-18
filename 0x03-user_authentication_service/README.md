Here's the improved README file in code format with emojis:

```markdown
# 📋 User Authentication Service

This project contains tasks for learning to create a user authentication service using Flask, SQLAlchemy, and bcrypt. The service includes user registration, login, session management, and password reset functionalities.

## 📚 Table of Contents

- [Requirements](#requirements)
- [Installation](#installation)
- [Usage](#usage)
- [API Endpoints](#api-endpoints)
- [Tasks To Complete](#tasks-to-complete)
- [Contributing](#contributing)
- [License](#license)

## ⚙️ Requirements

- SQLAlchemy 1.3.x
- pycodestyle 2.5
- bcrypt
- python3 3.7

## 🛠 Installation

1. Clone the repository:
    ```sh
    git clone https://github.com/yourusername/auth-service.git
    cd auth-service
    ```

2. Create a virtual environment and activate it:
    ```sh
    python3 -m venv venv
    source venv/bin/activate
    ```

3. Install the required packages:
    ```sh
    pip install -r requirements.txt
    ```

## 🚀 Usage

1. Start the Flask app:
    ```sh
    python app.py
    ```

2. Use an API client like Postman or cURL to interact with the endpoints.

## 🔗 API Endpoints

- `GET /`: Welcome message.
- `POST /users`: Register a new user.
- `POST /sessions`: Log in and create a session.
- `DELETE /sessions`: Log out and destroy the session.
- `GET /profile`: Get the user profile.
- `POST /reset_password`: Generate a password reset token.
- `PUT /reset_password`: Update the password using the reset token.

## 📋 Tasks To Complete

### 0. **User model**

[user.py](user.py) contains a SQLAlchemy model named `User` for a database table named `users` and meets the following requirements:
- The model will have the following attributes:
  - `id`, the integer primary key.
  - `email`, a non-nullable string.
  - `hashed_password`, a non-nullable string.
  - `session_id`, a nullable string.
  - `reset_token`, a nullable string.

### 1. **Create user**

[db.py](db.py) contains a completion of the `DB` class to implement the `add_user` method according to the given requirements:
    ```python
    #!/usr/bin/env python3
    """DB module.
    """
    from sqlalchemy import create_engine
    from sqlalchemy.orm import sessionmaker
    from sqlalchemy.orm.session import Session
    from sqlalchemy.ext.declarative import declarative_base

    from user import Base

    class DB:
        """DB class.
        """

        def __init__(self) -> None:
            """Initialize a new DB instance.
            """
            self._engine = create_engine("sqlite:///a.db", echo=False)
            Base.metadata.drop_all(self._engine)
            Base.metadata.create_all(self._engine)
            self.__session = None

        @property
        def _session(self) -> Session:
            """Memoized session object.
            """
            if self.__session is None:
                DBSession = sessionmaker(bind=self._engine)
                self.__session = DBSession()
            return self.__session
    ```
- Note that `DB._session` is a private property and hence should NEVER be used from outside the DB class.
- Implement the `add_user` method, which has two required string arguments: `email` and `hashed_password`, and returns a `User` object. The method should save the user to the database. No validations are required at this stage.

### 2. **Find user**

[db.py](db.py) contains the following updates:
- Implement the `DB.find_user_by` method. This method takes in arbitrary keyword arguments and returns the first row found in the `users` table as filtered by the method’s input arguments. No validation of input arguments required at this point.
- Make sure that SQLAlchemy’s `NoResultFound` and `InvalidRequestError` are raised when no results are found, or when wrong query arguments are passed, respectively.
- **Warning:**
  - `NoResultFound` has been moved from `sqlalchemy.orm.exc` to `sqlalchemy.exc` between the version 1.3.x and 1.4.x of SQLAchemy - please make sure you are importing it from `sqlalchemy.orm.exc`.

### 3. **Update user**

[db.py](db.py) contains the following updates:
- Implement the `DB.update_user` method that takes as arguments a required `user_id` integer, an arbitrary keyword arguments, and returns `None`.
- The method will use `find_user_by` to locate the user to update, then will update the user’s attributes as passed in the method’s arguments then commit changes to the database.
- If an argument that does not correspond to a user attribute is passed, raise a `ValueError`.

### 4. **Hash password**

[auth.py](auth.py) contains a `_hash_password` method that takes in a password string arguments and returns bytes, which is a salted hash of the input password, hashed with `bcrypt.hashpw`.

### 5. **Register user**

[auth.py](auth.py) contains the following updates:
- Implement the `Auth.register_user` in the `Auth` class provided below:
    ```python
    from db import DB

    class Auth:
        """Auth class to interact with the authentication database.
        """

        def __init__(self):
            self._db = DB()
    ```
- Note that `Auth._db` is a private property and should NEVER be used from outside the class.
- `Auth.register_user` should take mandatory `email` and `password` string arguments and return a `User` object.
- If a user already exists with the passed `email`, raise a `ValueError` with the message `User <user's email> already exists`.
- If not, hash the password with `_hash_password`, save the user to the database using `self._db` and return the `User` object.

### 6. **Basic Flask app**

[app.py](app.py) contains a basic Flask app with the following requirements:
- Create a Flask app that has a single `GET` route (`"/"`) and use `flask.jsonify` to return a JSON payload of the form:
    ```json
    {"message": "Bienvenue"}
    ```
- Add the following code at the end of the module:
    ```python
    if __name__ == "__main__":
        app.run(host="0.0.0.0", port="5000")
    ```

### 7. **Register user**

[app.py](app.py) contains the following updates:
- Implement the end-point to register a user. Define a `users` function that implements the `POST /users` route.
- Import the `Auth` object and instantiate it at the root of the module as such:
    ```python
    from auth import Auth

    AUTH = Auth()
    ```
- The end-point should expect two form data fields: `"email"` and `"password"`. If the user does not exist, the end-point should register it and respond with the following JSON payload:
    ```json
    {"email": "<registered email>", "message": "user created"}
    ```
- If the user is already registered, catch the exception and return a JSON payload of the form:
    ```json
    {"message": "email already registered"}
    ```
    and return a 400 status code.
- Remember that you should only use `AUTH` in this app. `DB` is a lower abstraction that is proxied by `Auth`.

### 8. **Credentials validation**

[auth.py](auth.py) contains the following updates:
- Implement the `Auth.valid_login` method. It should expect `email` and `password` required arguments and return a boolean.
- Try locating the user by email. If it exists, check the password with `bcrypt.checkpw`. If it matches return `True`. In any other case, return `False`.

### 9. **Generate UUIDs**

[auth.py](auth.py) contains the following updates:
- Implement a `_generate_uuid` function in the `auth` module. The function should return a string representation of a new UUID. Use the `uuid` module.
- Note that the method is private to the `auth` module and should **NOT** be used outside of it.

### 10. **Get session ID**

[auth.py](auth.py) contains the following updates:
- Implement the `Auth.create_session` method. It takes an `email` string argument and returns the session ID as a string.
- The method should find the user corresponding to the email, generate a new UUID and store it in the database as the user’s `session_id`, then return the session ID.

### 11. **Log in**

[app.py](app.py) contains the following updates:
- Implement a `login` function to respond to the `POST /sessions` route.
- The request is expected to contain form data with `"email"` and a `"password"` fields.
- If the login information is incorrect, use `flask.abort` to respond with a 401 HTTP status.
- Otherwise, create a new session for the user, store it the session ID as a cookie with key `"session_id"` on the response and return a JSON payload of the form:
    ```json
    {"email": "<user email>", "message": "logged in"}
    ```

### 12. **Find user by session ID**

[

auth.py](auth.py) contains the following updates:
- Implement the `Auth.get_user_from_session_id` method. It takes a single `session_id` string argument and returns a corresponding `User` or `None`.

### 13. **Log out**

[app.py](app.py) contains the following updates:
- Implement a `logout` function to respond to the `DELETE /sessions` route.
- The request is expected to contain the session ID as a cookie with key `"session_id"`.
- Find the user with the requested session ID. If the user exists destroy the session and remove the cookie, and return a JSON payload of the form:
    ```json
    {"message": "logged out"}
    ```
- If the session ID is invalid, respond with a 403 HTTP status code.

### 14. **User profile**

[app.py](app.py) contains the following updates:
- Implement a `profile` function to respond to the `GET /profile` route.
- The request is expected to contain the session ID as a cookie with key `"session_id"`.
- Use it to find the corresponding user. If the user is found, respond with a JSON payload of the form:
    ```json
    {"email": "<user email>"}
    ```
- If the session ID is invalid, respond with a 403 HTTP status code.

### 15. **Generate reset password token**

[auth.py](auth.py) contains the following updates:
- Implement the `Auth.get_reset_password_token` method. It takes an `email` string argument and returns a string.
- Find the corresponding user. If the user does not exist, raise a `ValueError`.
- Otherwise, generate a UUID and update the user’s `reset_token` database field. Return the token.

### 16. **Get reset password token**

[app.py](app.py) contains the following updates:
- Implement a `get_reset_password_token` function to respond to the `POST /reset_password` route.
- The request is expected to contain form data with the `"email"` field.
- If the email is not registered, respond with a 403 status code.
- Otherwise, generate a token and respond with a JSON payload of the form:
    ```json
    {"email": "<user email>", "reset_token": "<reset token>"}
    ```

### 17. **Update password**

[auth.py](auth.py) contains the following updates:
- Implement the `Auth.update_password` method. It takes `reset_token` string argument and a `password` string argument and returns `None`.
- Use the `reset_token` to find the corresponding user. If it does not exist, raise a `ValueError`.
- Otherwise, hash the password and update the user’s `hashed_password` field in the database, as well as reset the `reset_token` field.

### 18. **Update password end-point**

[app.py](app.py) contains the following updates:
- Implement the `update_password` function to respond to the `PUT /reset_password` route.
- The request is expected to contain form data with fields: `"email"`, `"reset_token"`, and `"new_password"`.
- Update the password. If the token is invalid, respond with a 403 HTTP status code. Otherwise, respond with a JSON payload of the form:
    ```json
    {"email": "<user email>", "message": "Password updated"}
    ```

```