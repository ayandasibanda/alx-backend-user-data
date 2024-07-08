# 🔒 Basic Authentication

This project contains tasks for learning to authenticate a user using the Basic authentication scheme.

## 📝 Tasks To Complete

### ✅ 0. Simple-basic-API
- **Setup and start the server**:
    ```sh
    bob@dylan:~$ pip3 install -r requirements.txt
    ...
    bob@dylan:~$ API_HOST=0.0.0.0 API_PORT=5000 python3 -m api.v1.app
     * Serving Flask app "app" (lazy loading)
    ...
    bob@dylan:~$
    ```
- **Use the API** (in another tab or in your browser):
    ```sh
    bob@dylan:~$ curl "http://0.0.0.0:5000/api/v1/status" -vvv
    *   Trying 0.0.0.0...
    * TCP_NODELAY set
    * Connected to 0.0.0.0 (127.0.0.1) port 5000 (#0)
    > GET /api/v1/status HTTP/1.1
    > Host: 0.0.0.0:5000
    > User-Agent: curl/7.54.0
    > Accept: */*
    >
    * HTTP 1.0, assume close after body
    < HTTP/1.0 200 OK
    < Content-Type: application/json
    < Content-Length: 16
    < Access-Control-Allow-Origin: *
    < Server: Werkzeug/1.0.1 Python/3.7.5
    < Date: Mon, 18 May 2020 20:29:21 GMT
    <
    {"status":"OK"}
    * Closing connection 0
    bob@dylan:~$
    ```

### ✅ 1. Error handler: Unauthorized
- **Edit `api/v1/app.py`**:
  - Add a new error handler for status code `401` with a JSON response: `{"error": "Unauthorized"}`.
  - Use `jsonify` from Flask.

- **Testing**:
  - Add a new endpoint in `api/v1/views/index.py`:
    - Route: `GET /api/v1/unauthorized`.
    - This endpoint raises a 401 error using `abort`.

### ✅ 2. Error handler: Forbidden
- **Edit `api/v1/app.py`**:
  - Add a new error handler for status code `403` with a JSON response: `{"error": "Forbidden"}`.
  - Use `jsonify` from Flask.

- **Testing**:
  - Add a new endpoint in `api/v1/views/index.py`:
    - Route: `GET /api/v1/forbidden`.
    - This endpoint raises a 403 error using `abort`.

### ✅ 3. Auth class
- **Create a class to manage API authentication**:
  - Create a folder `api/v1/auth`.
  - Create an empty file `api/v1/auth/__init__.py`.
  - Create the `Auth` class in `api/v1/auth/auth.py`:
    - Methods:
      - `require_auth(self, path: str, excluded_paths: List[str]) -> bool` returns `False`.
      - `authorization_header(self, request=None) -> str` returns `None`.
      - `current_user(self, request=None) -> TypeVar('User')` returns `None`.

### ✅ 4. Define which routes don't need authentication
- **Update `require_auth` method in `Auth` class**:
  - Returns `True` if path is not in `excluded_paths`.
  - Must handle `None` and empty values gracefully.
  - Make method slash tolerant.

### ✅ 5. Request validation!
- **Update `authorization_header` method in `Auth` class**:
  - Returns the `Authorization` header value if present, otherwise `None`.

- **Update `api/v1/app.py`**:
  - Create a variable `auth` and initialize it based on `AUTH_TYPE`.
  - Add a method to handle `before_request` to validate requests:
    - Use `require_auth`, `authorization_header`, and `current_user` methods.
    - Raise `401` or `403` errors as needed.

### ✅ 6. Basic auth
- **Create `BasicAuth` class**:
  - Inherit from `Auth` in `api/v1/auth/basic_auth.py`.
  - Update `api/v1/app.py` to use `BasicAuth` based on `AUTH_TYPE`.

### ✅ 7. Basic - Base64 part
- **Add `extract_base64_authorization_header` method in `BasicAuth`**:
  - Extracts and returns the Base64 part of the `Authorization` header.

### ✅ 8. Basic - Base64 decode
- **Add `decode_base64_authorization_header` method in `BasicAuth`**:
  - Decodes the Base64 string and returns it as a UTF-8 string.

### ✅ 9. Basic - User credentials
- **Add `extract_user_credentials` method in `BasicAuth`**:
  - Extracts and returns the user email and password from the Base64 decoded value.

### ✅ 10. Basic - User object
- **Add `user_object_from_credentials` method in `BasicAuth`**:
  - Returns the `User` instance based on email and password credentials.

### ✅ 11. Basic - Overload current_user - and BOOM!
- **Add `current_user` method in `BasicAuth`**:
  - Retrieves the `User` instance for a request using the Basic Authentication flow.

### ✅ 12. Basic - Allow password with ":"
- **Improve `extract_user_credentials`**:
  - Handle passwords containing `:` correctly.

### ✅ 13. Require auth with stars
- **Improve `require_auth`**:
  - Allow `*` at the end of excluded paths for flexible matching.
