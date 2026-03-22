# FTEC 6V97 – Financial Applications of Web Development

## Sample Exam — Study Reference

> **This sample exam is provided so that you can familiarize yourself with the format, style, and depth of questions you will encounter on the midterm. It is shorter than the actual exam and covers a representative selection of topics. Answers are included at the end of each section.**

---

## Part I — Multiple Choice (5 questions)

Select the best answer for each question.

1. In Flask, which method is used to safely read a JSON request body, returning `None` instead of raising an exception when the body is absent or malformed?  
   a) `request.data`  
   b) `request.args.get('json')`  
   c) `request.get_json(silent=True)`  
   d) `request.form.get('json')`  

2. Which of the following best describes a SQLAlchemy **scoped session** in a Flask application?  
   a) A single global session shared by all requests and threads  
   b) A thread-safe, context-aware session automatically created per request and cleaned up when the response is returned
   c) A session that is created once at application startup and never closed  
   d) A read-only connection to the database used exclusively for SELECT queries  

3. When Pydantic validates a field declared as `quantity: int = Field(..., gt=0)` and receives the value `0`, it:  
   a) Accepts the value because `0` is a valid integer  
   b) Coerces the value to `1` automatically  
   c) Raises a `ValidationError` because `gt=0` means the value must be strictly greater than zero  
   d) Stores `None` as a fallback value  

4. In the OIDC authentication flow, the **Refresh Token** is used to:  
   a) Obtain new Access Tokens after the existing one expires, without requiring the user to log in again  
   b) Verify the identity of a user when they first log in  
   c) Sign outgoing API requests on behalf of the user  
   d) Encrypt the user's password before it is sent to the Identity Provider  

5. `db.session.add(portfolio)` in a Flask-SQLAlchemy route:  
   a) Immediately writes the `portfolio` record to the database  
   b) Stages the `portfolio` object for insertion into the database, which is finalized only when `db.session.commit()` is called  
   c) Updates an existing record with the same primary key  
   d) Opens a new database connection for the current request  

---

### Part I — Answers

| # | Answer | Explanation |
| --- | -------- | ------------- |
| 1 | **c** | `request.get_json(silent=True)` returns `None` if the body is missing, empty, or not valid JSON, rather than raising a `BadRequest` exception. This allows the route to handle the missing body gracefully. |
| 2 | **b** | A scoped session in SQLAlchemy is tied to the current request context. Flask-SQLAlchemy creates it automatically the first time the database is accessed in a request and removes it when the application context tears down after the response is sent. |
| 3 | **c** | `gt=0` enforces strictly greater than zero. The value `0` fails this constraint, and Pydantic raises a `ValidationError` containing a description of the failing field and rule. |
| 4 | **a** | The Refresh Token is a long-lived token whose sole purpose is to retrieve a new Access Token after the short-lived one expires. It avoids forcing the user to re-enter their credentials on every session. |
| 5 | **b** | `db.session.add()` stages (queues) the object for insertion. No SQL is sent to the database until `db.session.commit()` is called. This is the foundation of the Unit of Work pattern. |

---

## Part II — Code Evaluation (2 questions)

### Question 1

A developer defines the following Pydantic schema for creating a new investment position, but it contains an error:

```python
from pydantic import BaseModel, Field
from typing import Optional

class CreatePositionSchema(BaseModel):
    ticker: str = Field(..., min_length=1, max_length=5)
    quantity: int = Field(..., gt=0)
    note: str = Field(...)
```

A client sends a request body with `ticker` and `quantity` provided but omits `note`. The developer expected this to work because notes are optional in the product requirements.

a) Explain why the schema as written does not match the product requirement that `note` is optional.  
b) Rewrite the `note` field declaration so that it is optional and defaults to `None` when not provided by the client.

---

### Question 2

Consider the following Flask route:

```python
@app.route('/api/portfolios', methods=['POST'])
def create_portfolio():
    data = request.get_json()
    portfolio = Portfolio(name=data['name'], owner=data['owner'])
    db.session.add(portfolio)
    db.session.commit()
    return jsonify({'id': portfolio.id, 'name': portfolio.name}), 201
```

a) Describe two separate scenarios in which this route will raise an unhandled exception.  
b) For one of those scenarios, show how a Flask global error handler could be added to return a clean JSON error response rather than letting the exception propagate.

---

### Part II — Answers

#### Question 1 - Code Evaluation

**a)** The field `note: str = Field(...)` uses the ellipsis `...`, which marks the field as **required**. Pydantic will raise a `ValidationError` if `note` is absent from the request body, regardless of the product intent. There is no default value and no `Optional` type hint, so omitting `note` always fails validation.

**b)** Declare `note` using `Optional[str]` and provide `None` as the default:

```python
from typing import Optional

class CreatePositionSchema(BaseModel):
    ticker: str = Field(..., min_length=1, max_length=5)
    quantity: int = Field(..., gt=0)
    note: Optional[str] = None
```

Now `note` is not required. If the client omits it, the field is set to `None` automatically.

---

#### Question 2 - Code Evaluation

a)

- **Scenario 1 — Missing or malformed JSON body:** If the client sends a request without a JSON body (or with an invalid `Content-Type`), `request.get_json()` returns `None`. Accessing `data['name']` on `None` raises a `TypeError`.
- **Scenario 2 — Missing key in the JSON body:** If the client sends a valid JSON body but omits `name` or `owner`, accessing `data['name']` raises a `KeyError`.

b) A global error handler for `Exception` (or more specifically `KeyError` / `TypeError`) ensures a clean JSON response:

```python
@app.errorhandler(Exception)
def handle_generic_error(error):
    db.session.rollback()
    return jsonify({'error': 'An internal error occurred', 'code': 500}), 500
```

For the missing-key case specifically, a better practice is to validate incoming data with Pydantic and handle `ValidationError` with a `400` response, rather than catching generic exceptions. The handler above covers unexpected failures and ensures that any uncommitted database changes are rolled back before the response is sent.

---

## Part III — Short Essays (2 questions)

Answer each question in 4–8 sentences.

### Question 1 - Short Essay

Explain the role of Flask **Blueprints** in organizing a web service. Describe what problem they solve as a project grows, how they are defined and registered, and how they interact with the application factory pattern.

### Question 2 - Short Essay

Explain what the **`@requires_auth` decorator** does, step by step, when a request arrives at a protected Flask route. In your answer, describe what it checks, what it stores, and what it returns in both the success and failure cases.

---

### Part III — Answers

#### Question 1 — Model Answer

As a Flask application grows, defining all routes directly on a single `app` object in one file becomes difficult to navigate and maintain. Blueprints solve this by allowing routes to be grouped into modular, domain-specific collections — for example, one blueprint for portfolio routes and another for investment routes. A Blueprint is created with `Blueprint("name", __name__, url_prefix="/api/...")` and routes are defined on it exactly as they would be on the app object. The Blueprint is then registered with the Flask app inside the application factory using `app.register_blueprint(blueprint_name)`. This registration step is what connects the Blueprint's routes to the running application. Because Blueprints are registered inside the factory rather than at import time, each environment (development, test, production) gets its own clean app instance with the same modular route structure — a key advantage of combining Blueprints with the factory pattern.

---

#### Question 2 — Model Answer

The `@requires_auth` decorator wraps a route handler function so that authentication is enforced before the handler's code ever runs. When a request arrives, the decorator's inner `wrapper` function executes first. It reads the `Authorization` header from the request and confirms it follows the `Bearer <token>` format. If the header is missing or malformed, the wrapper immediately returns a `401 Unauthorized` JSON response and the original handler is never called. If the header is valid, the token string is extracted and passed to `validate_token()`, which verifies the JWT signature, checks the expiry and audience claims, and confirms the `token_use` is `access`. If `validate_token()` raises a `ValueError` for any reason, the wrapper again returns a `401` response. If validation succeeds, the decoded claims dictionary is stored in `g.current_user` — Flask's per-request context object — so the route handler can access the authenticated user's identity without receiving it as a parameter. Finally, the wrapper calls the original route handler and returns its response to the client.
