# 🛡️ Session Authentication

This project contains tasks for learning to authenticate a user through session authentication.

## 📝 Tasks To Complete

### ✅ 0. Et moi et moi et moi!
- **Description**: Copy all your work of the [0x06. Basic authentication](../0x01-Basic_authentication/) project in this new folder.
- **Requirements**:
  - Implement **Basic authentication** for accessing all User endpoints:
    - `GET /api/v1/users`
    - `POST /api/v1/users`
    - `GET /api/v1/users/<user_id>`
    - `PUT /api/v1/users/<user_id>`
    - `DELETE /api/v1/users/<user_id>`
  - Add a new endpoint: `GET /users/me` to retrieve the authenticated `User` object.
  - Copy folders `models` and `api` from the previous project [0x06. Basic authentication](../0x01-Basic_authentication/).
  - Update `@app.before_request` in [api/v1/app.py](api/v1/app.py):
    - Assign the result of `auth.current_user(request)` to `request.current_user`.
  - Update method for the route `GET /api/v1/users/<user_id>` in [api/v1/views/users.py](api/v1/views/users.py):
    - If `<user_id>` is equal to `me` and `request.current_user` is `None`: `abort(404)`.
    - If `<user_id>` is equal to `me` and `request.current_user` is not `None`: return the authenticated `User` in a JSON response.

### ✅ 1. Empty session
- **File**: [api/v1/auth/session_auth.py](api/v1/auth/session_auth.py)
- **Description**: Create a class `SessionAuth` that inherits from `Auth`.
- **Requirements**:
  - Validate if everything inherits correctly without any overloading.
  - Validate the "switch" by using environment variables.
  - Update [api/v1/app.py](api/v1/app.py) to use `SessionAuth` based on the environment variable `AUTH_TYPE`.

### ✅ 2. Create a session
- **File**: [api/v1/auth/session_auth.py](api/v1/auth/session_auth.py)
- **Description**: Update `SessionAuth` class to create a session.
- **Requirements**:
  - Create a class attribute `user_id_by_session_id` initialized by an empty dictionary.
  - Create an instance method `create_session(self, user_id: str = None) -> str:` that creates a Session ID for a `user_id`.

### ✅ 3. User ID for Session ID
- **File**: [api/v1/auth/session_auth.py](api/v1/auth/session_auth.py)
- **Description**: Update `SessionAuth` class to return a `User` ID based on a Session ID.
- **Requirements**:
  - Create an instance method `user_id_for_session_id(self, session_id: str = None) -> str:` that returns a `User` ID based on a Session ID.

### ✅ 4. Session cookie
- **File**: [api/v1/auth/auth.py](api/v1/auth/auth.py)
- **Description**: Update `auth.py` by adding a method `session_cookie(self, request=None)` that returns a cookie value from a request.

### ✅ 5. Before request
- **File**: [api/v1/app.py](api/v1/app.py)
- **Description**: Update the `@app.before_request` method.
- **Requirements**:
  - Add the URL path `/api/v1/auth_session/login/` to the list of excluded paths of the method `require_auth`.
  - If `auth.authorization_header(request)` and `auth.session_cookie(request)` return `None`, `abort(401)`.

### ✅ 6. Use Session ID for identifying a User
- **File**: [api/v1/auth/session_auth.py](api/v1/auth/session_auth.py)
- **Description**: Update `SessionAuth` class to return a `User` instance based on a cookie value.
- **Requirements**:
  - Create an instance method `current_user(self, request=None)` that returns a `User` instance based on a cookie value.

### ✅ 7. New view for Session Authentication
- **File**: [api/v1/views/session_auth.py](api/v1/views/session_auth.py)
- **Description**: Create a new Flask view that handles all routes for the Session authentication.
- **Requirements**:
  - Create a route `POST /auth_session/login`:
    - Use `request.form.get()` to retrieve `email` and `password` parameters.
    - Create a Session ID for the `User` ID and set the cookie to the response.
  - Update [api/v1/views/__init__.py](api/v1/views/__init__.py) to add this new view.

### ✅ 8. Logout
- **File**: [api/v1/auth/session_auth.py](api/v1/auth/session_auth.py)
- **Description**: Update the class `SessionAuth` by adding a new method `destroy_session(self, request=None)` that deletes the user session / logout.
- **Requirements**:
  - Create a new route `DELETE /api/v1/auth_session/logout` in [api/v1/views/session_auth.py](api/v1/views/session_auth.py):
    - Use `auth.destroy_session(request)` to delete the Session ID contents in the request as cookie.

### ✅ 9. Expiration?
- **File**: [api/v1/auth/session_exp_auth.py](api/v1/auth/session_exp_auth.py)
- **Description**: Create a class `SessionExpAuth` that inherits from `SessionAuth`.
- **Requirements**:
  - Overload `__init__` method to assign an instance attribute `session_duration`.
  - Overload `create_session(self, user_id=None)` to create a session with an expiration date.
  - Overload `user_id_for_session_id(self, session_id=None)` to return `User` ID based on session expiration.

### ✅ 10. Sessions in database
- **Description**: Store Session IDs in the database.
- **Requirements**:
  - Create a model `UserSession` in [models/user_session.py](models/user_session.py) that inherits from `Base`.
  - Create a new authentication class `SessionDBAuth` in [api/v1/auth/session_db_auth.py](api/v1/auth/session_db_auth.py) that inherits from `SessionExpAuth`:
    - Overload `create_session(self, user_id=None)` to create and store a new instance of `UserSession`.
    - Overload `user_id_for_session_id(self, session_id=None)` to return the User ID from the database.
    - Overload `destroy_session(self, request=None)` to destroy the `UserSession` based on the Session ID.
  - Update [api/v1/app.py](api/v1/app.py) to instantiate `auth` with `SessionDBAuth` if the environment variable `AUTH_TYPE` is equal to `session_db_auth`.
