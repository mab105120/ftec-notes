# Introduction to Web Services with Flask

## What Are Web Services—and Why Use Them?

A web service is an application whose functions are made available to other programs over a network. Instead of a human clicking through a command-line menu, another program sends an HTTP request to a URL and receives data in return. The core idea is remote access to functionality: an app in one place can trigger actions and receive results from another system, regardless of where either runs.

A simple analogy is a restaurant: you, the client, don’t enter the kitchen. You send a request (an order) to a well-defined endpoint (the waiter), who delivers your message to the server (the kitchen). The response—your meal—comes back in a predictable form. In computing, web services play the same role: they make business logic accessible remotely through standardized “menus” of URLs.

This is how modern systems communicate. A mobile app that checks your portfolio balance, a trading bot that fetches market data, or a dashboard that shows analytics—all call web services that perform the underlying computations.

## Flask: A Lightweight Framework for Building Web Applications

Flask is a micro-framework for Python designed for clarity and extensibility. It provides the essential building blocks of a web application—routing, request handling, templating, and extensibility—without forcing a rigid structure.

Its simplicity is what makes it ideal for learning: you start with a few lines of code and can gradually scale to large applications with databases, authentication, and APIs.

Example of the smallest possible web service:

```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello, world!"

if __name__ == "__main__":
    app.run(debug=True)
```

Run this file and visit <http://127.0.0.1:5000/>. You have just exposed a Python function to the world.

## Conceptual Structure of a Flask Web Service

As projects grow, structure matters. A well-organized Flask app separates concerns the same way a building separates rooms: each area serves a clear purpose. A conventional layout looks like this:

```python
app/
  __init__.py        # application factory
  routes/            # blueprints for endpoints
    portfolio.py
  models/            # SQLAlchemy models
    portfolio.py
  db.py              # database setup (engine, Base, session)
  config.py          # configuration classes
tests/
  test_routes.py
```

This modular structure mirrors the conceptual layers of a web service:

- Routes (Views) — map URLs to Python functions.
- Models — define data and database relationships.
- Extensions — such as SQLAlchemy or login managers that integrate extra functionality.
- Configuration — centralizes runtime settings (database URLs, debug flags, etc.).
- Factory — creates the Flask app, registers blueprints, and initializes extensions.

## Building the Application Using the Factory Pattern

Rather than creating a global app object, Flask encourages the application factory pattern—a function that builds and configures the app when called. This makes the application flexible, testable, and environment-aware.

```python
# app/__init__.py
from flask import Flask
from .routes.portfolio import portfolio_bp
from .db import db

def create_app(config_class):
    app = Flask(__name__)
    app.config.from_object(config_class)

    # Initialize extensions
    db.init_app(app)

    # Register blueprints
    app.register_blueprint(portfolio_bp)

    return app
```

Why is this pattern best practice?
Because it avoids tight coupling. Different environments (development, testing, production) can create the app with different configurations. Tests can spin up temporary apps in memory. This modularity keeps code cleaner and deployment easier.

## Application Configuration

Configurations control how the service behaves. Common configurable items include:

- Database connection strings
- Debug vs production flags
- Secret keys for sessions or JWTs
- API keys for external services

A configuration module keeps these concerns separate:

```python
# app/config.py
import os

class Config:
    SQLALCHEMY_DATABASE_URI = os.getenv("DATABASE_URL", "sqlite:///local.db")
    SQLALCHEMY_TRACK_MODIFICATIONS = False
    DEBUG = True
```

At creation time, the factory applies this configuration:

```python
app = create_app(Config)
```

## HTTP: The Language of Web Services

All communication between a client and a web service happens through the HTTP protocol. Each message has:

- A method (GET, POST, PUT, DELETE) specifying the action.
- A URL identifying the resource.
- Headers containing metadata.
- An optional body carrying data (for POST or PUT).

When your browser or frontend app calls a Flask route, it is effectively sending an HTTP request.

## Routes and Blueprints

### Understanding Routes

A route is the fundamental mechanism by which Flask maps an incoming HTTP request to a Python function. Each route declares a path—a fragment of the URL that identifies which piece of functionality should execute when that path is accessed—and associates it with a Python function that performs the desired action.

When a client (such as a browser or a frontend app) makes a request to a specific URL, Flask examines the path and checks whether any route definition matches it. If it finds a match, it calls the corresponding function. The function runs, possibly interacting with a database or other services, and returns a response.

At its simplest, a route is a mapping from a URL pattern to a Python function.

```python
@app.route("/hello")
def greet():
    return "Hello from Flask!"
```

In this example, Flask creates a mapping between the path /hello and the function greet(). Whenever a client sends a request to <http://localhost:5000/hello>, Flask calls greet() and returns its result as the HTTP response body.

The route decorator @app.route() tells Flask: “When you see this URL pattern, execute the function below it.”

### Introducing Blueprints

As applications grow, managing many routes directly on a single app object quickly becomes cumbersome. To organize routes into logical groups, Flask provides blueprints—modular collections of routes and related logic that can be registered with the main application.

You can think of blueprints as “mini-applications” inside your app. Each blueprint handles a subset of functionality (for example, user management, portfolios, or transactions). This modularity mirrors how large systems are designed: every component handles its own domain, but they all integrate under one central application.

A typical blueprint is defined like this:

```python
# app/routes/portfolio.py
from flask import Blueprint, jsonify

# Create a blueprint instance
portfolio_bp = Blueprint("portfolio", __name__, url_prefix="/api/portfolio")

@portfolio_bp.route("/", methods=["GET"])
def list_portfolios():
    return jsonify({"message": "Listing all portfolios"})

@portfolio_bp.route("/<int:portfolio_id>", methods=["GET"])
def get_portfolio(portfolio_id):
    return jsonify({"portfolio_id": portfolio_id})

```

Here, portfolio_bp is a blueprint object whose routes are prefixed with /api/portfolio. The first route responds to `/api/portfolio/` and the second to `/api/portfolio/<id>`.

Once defined, the blueprint must be registered in the application factory so Flask knows to include its routes:

```python
# app/__init__.py
from flask import Flask
from .routes.portfolio import portfolio_bp

def create_app(config_class):
    app = Flask(__name__)
    app.config.from_object(config_class)

    # Register blueprint
    app.register_blueprint(portfolio_bp)

    return app
```

When `create_app()` is called, the Flask app automatically becomes aware of all routes defined inside `portfolio_bp`.
Blueprint registration is therefore a key part of the application factory pattern: it connects modular route collections to the central app without global variables or circular imports.

### Sending Data through Routes

Routes often need to receive data from clients—for example, when fetching a specific portfolio by ID or filtering portfolios by owner. Flask supports two common ways to send data as part of a request:

1. Path parameters (sometimes called URL parameters)
2. Query parameters (also called argument parameters or URL arguments)

Both allow clients to pass small pieces of information directly through the URL, but they differ in placement, purpose, and typical use cases.

### Path Parameters

A path parameter is part of the URL itself. It acts as a placeholder in the route definition, which Flask fills with actual values from the incoming request.

In the route pattern, path parameters are declared inside angle brackets (< >). The name inside the brackets becomes a function argument, and you can optionally specify its type.

Example:

```python
@portfolio_bp.route("/<int:portfolio_id>", methods=["GET"])
def get_portfolio(portfolio_id):
    return jsonify({"message": f"Fetching portfolio {portfolio_id}"})
```

Here, `<int:portfolio_id>` declares that the path includes an integer segment. A request to:

```python
GET /api/portfolio/5
```

will make Flask call `get_portfolio(5)`—that is, it extracts the number 5 from the URL and passes it into the function.

You can also define multiple path parameters:

```python
@portfolio_bp.route("/<int:portfolio_id>/investments/<int:investment_id>")
def get_investment(portfolio_id, investment_id):
    return jsonify({"portfolio": portfolio_id, "investment": investment_id})
```

which responds to URLs such as:

```python
GET /api/portfolio/3/investments/12
```

Path parameters are ideal when the data being passed is hierarchical or identifies a resource directly, such as IDs, usernames, or slugs.

### Query Parameters

A query parameter appears after the `?` in a URL and provides optional, non-hierarchical information. Unlike path parameters, query parameters are not part of the URL’s structural pattern; instead, they are key-value pairs used to filter, sort, or customize results.

For example, the following route:

```python
@portfolio_bp.route("/", methods=["GET"])
def list_portfolios():
    owner = request.args.get("owner")
    limit = request.args.get("limit", type=int)
    return jsonify({
        "message": "Listing portfolios",
        "owner": owner,
        "limit": limit
    })
```

can be called with:

```python
GET /api/portfolio?owner=alice&limit=10
```

Flask makes all query parameters available through `request.args`, a dictionary-like object. Here, `owner` and `limit` are extracted from the URL and used to tailor the response.

Query parameters are most appropriate for filtering, pagination, or configuration options that do not define the identity of a resource but rather modify how data is retrieved.

### Comparing Path and Query Parameters

Aspect|Path Parameter|Query Parameter|
---|---|---|
Placement|Part of the route path itself (/api/portfolio/5)|After a question mark (/api/portfolio?owner=alice)|
Purpose|Identifies a specific resource or entity|Filters or customizes the request|
Visibility in Route Definition|Declared with `<variable>` in @route()|Accessed via request.args|
Typical Use Cases|/users/7, /portfolio/12/investments|/search?q=tech, /portfolio?owner=alice&limit=10|

Guideline:
Use path parameters when the data is essential to identifying the resource (like an ID).
Use query parameters when the data simply refines the request or controls optional behavior (like filters, limits, or sorts).

Note: Passing data through path or query parameters works well for small, textual pieces of information included in the URL itself. However, this is not the only way to send data between client and server.
In modern web APIs, larger or more complex data—such as new records or structured objects—is sent in the body of the request using the JSON format.
In a later section we will explore how to handle JSON in requests and responses, showing how Flask converts between Python objects and JSON seamlessly.

## The Request–Response Cycle in Flask

### Understanding the Request–Response Cycle

Every web service operates through a continuous dialogue between clients and servers called the **request–response cycle**. When a client—such as a browser, mobile app, or another backend service—calls one of your Flask routes, it sends a **request** message to the server. The server processes that request, executes the corresponding Python function, and sends back a **response** message.

This pattern repeats for every interaction: one request leads to one response. Understanding this cycle is fundamental for building predictable APIs and for debugging web applications.

Conceptually, the cycle looks like this:

```python
Client  →  Request  →  Flask (route handler)  →  Response  →  Client
```

- The **request** carries data from the client to the server.
- The **response** carries data from the server back to the client.

Becoming comfortable with this flow helps you reason about how input and output move through your application—from a user’s action to your function and back.

---

### The Request

A **request** represents the incoming message that the client sends to your web service. It contains several components:

- **URL** – the address being accessed, which may include path or query parameters
- **HTTP method** – the action type (GET, POST, PUT, DELETE, etc.)
- **Headers** – metadata such as authentication tokens or content type
- **Body** – optional data (for example, JSON payloads or form submissions)

Flask automatically constructs a `Request` object for every incoming call. You can access it using the global variable `flask.request`.

#### Example: Inspecting Request Data

```python
from flask import request, jsonify

@app.route("/inspect", methods=["GET", "POST"])
def inspect():
    info = {
        "method": request.method,
        "url": request.url,
        "path": request.path,
        "headers": dict(request.headers),
        "args": request.args,            # query parameters
        "form": request.form,            # form data from POST
        "json": request.get_json(silent=True)  # body data if JSON
    }
    return jsonify(info)
```

A sample call such as:

```python
GET /inspect?owner=alice
```

might produce this output:

```json
{
  "method": "GET",
  "url": "http://127.0.0.1:5000/inspect?owner=alice",
  "path": "/inspect",
  "headers": {
    "Host": "127.0.0.1:5000",
    "User-Agent": "curl/8.0.1",
    "Accept": "*/*"
  },
  "args": {
    "owner": "alice"
  },
  "form": {},
  "json": null
}
```

This example shows how Flask gives you full access to all incoming data.
Depending on your route’s purpose, you can extract what you need:

- `request.args` → query parameters
- `request.view_args` → path parameters
- `request.get_json()` → JSON data sent in the request body

---

### The Response

A **response** is the server’s reply to a request. It contains the results of the operation, together with information describing whether the request was successful or not.

Each response includes:

- **Body** – the data being returned (often in JSON)
- **Headers** – metadata describing the response (such as content type)
- **Status code** – a numeric code indicating the result (e.g., 200, 201, 404)

Flask automatically converts return values from your route functions into response objects. You can also construct them manually when you need more control.

#### Example: Creating a Response

```python
from flask import jsonify, make_response

@app.route("/status", methods=["GET"])
def status():
    data = {"message": "Service is running normally"}
    headers = {"X-Service-Version": "1.0"}
    return make_response(jsonify(data), 200, headers)
```

This route sends a response containing:

- a JSON body (`{"message": "Service is running normally"}`)
- a status code (`200 OK`)
- a custom header (`X-Service-Version: 1.0`)

The client receives:

```python
HTTP/1.1 200 OK
Content-Type: application/json
X-Service-Version: 1.0
```

and the body:

```json
{
  "message": "Service is running normally"
}
```

#### Common Status Codes

Code                          | Meaning      | Typical Use                      |
----------------------------- | ------------ | -------------------------------- |
**200 OK**                    | Success      | Resource fetched successfully    |
**201 Created**               | Success      | New resource created             |
**400 Bad Request**           | Client error | Invalid input or missing data    |
**404 Not Found**             | Client error | Requested resource doesn’t exist |
**500 Internal Server Error** | Server error | Unexpected exception on server   |

Using the correct status codes helps clients quickly interpret results and handle errors consistently.

---

## Handling Errors and Failed Requests

Not all requests succeed. Clients might send invalid data, request missing resources, or trigger unexpected failures. Flask lets you define **custom error handlers** to respond gracefully and consistently when something goes wrong.

By default, Flask returns HTML error pages. For APIs, however, returning structured JSON errors is better practice.

### Example: Custom Error Handlers

```python
from flask import jsonify

@app.errorhandler(404)
def not_found_error(error):
    response = jsonify({
        "error": "Not Found",
        "message": "The requested resource could not be located."
    })
    return response, 404

@app.errorhandler(400)
def bad_request_error(error):
    response = jsonify({
        "error": "Bad Request",
        "message": "The request parameters were invalid or incomplete."
    })
    return response, 400

@app.errorhandler(500)
def internal_error(error):
    response = jsonify({
        "error": "Internal Server Error",
        "message": "An unexpected error occurred on the server."
    })
    return response, 500
```

Now, when a request fails—for example:

```python
GET /api/portfolio/999
```

the response might look like this:

```json
{
  "error": "Not Found",
  "message": "The requested resource could not be located."
}
```

with an HTTP status of `404 Not Found`.
This structured format ensures that client applications can handle errors programmatically.

---

## Summary and Transition

The request–response cycle is the backbone of all web communication:

- **Requests** carry data into the application (URLs, parameters, headers, bodies).
- **Responses** carry results and status information back to the client.
- **Error handling** ensures that even failures return clear, structured messages.

So far, the examples have shown how to exchange simple text and URL parameters.
However, modern web applications often send more complex data—such as full records or objects—in the **body** of the request.
This is commonly done using the **JSON format**, which Flask supports natively.
Later sections will introduce how to send and receive structured JSON data using `request.get_json()` and `jsonify()`.

## JSON as a Data Format for Web Services

### Why Data Needs to Be Exchanged Between Client and Server

In any interactive application, **data exchange** lies at the heart of the request–response cycle.
When a client sends a request to the server, it is typically asking for some data (for example, the list of portfolios a user owns) or sending new data to be stored (for example, creating a new investment). The server then processes that request—querying databases, performing calculations, or applying business logic—and returns the results as a **response**.

Without a structured way to represent and transmit this information, the client and server would have no common language.
Different systems—mobile apps, web browsers, and backend APIs—must all agree on how data is formatted, regardless of what programming languages they use internally.

This is why the internet relies on **standardized data formats**, and the most widely adopted of these today is **JSON**.

---

### JSON: The Universal Language of Data Exchange

**JSON (JavaScript Object Notation)** is the de facto standard for data exchange on the internet. It is lightweight, human-readable, and language-agnostic—meaning it can be created and parsed easily by almost every programming language, including Python, JavaScript, and Java.

At its core, JSON represents data using a small set of **notation rules** derived from JavaScript, but its simplicity and universality have made it the standard for web communication.

### Basic JSON Notations

Python Concept | JSON Equivalent | Example                        |
-------------- | --------------- | ------------------------------ |
Dictionary     | Object          | `{"name": "Alice", "age": 30}` |
List           | Array           | `["AAPL", "MSFT", "GOOGL"]`    |
String         | String          | `"portfolio"`                  |
Number         | Number          | `123.45`                       |
Boolean        | Boolean         | `true`, `false`                |
`None`         | `null`          | `null`                         |

These simple structures can be nested to represent complex, hierarchical data.
For example:

```json
{
  "user": {
    "id": 1,
    "name": "Alice"
  },
  "portfolios": [
    {"id": 10, "name": "Retirement"},
    {"id": 11, "name": "Education"}
  ]
}
```

This flexibility makes JSON ideal for modeling objects, collections, and relationships across the web.

---

### Sending JSON Data in the Request Body

When a client wants to send complex data—such as creating or updating a resource—it places that data inside the **body** of the HTTP request, formatted as JSON.
This approach is most common with `POST` and `PUT` methods, which create or modify data on the server.

#### Example: Sending JSON Data to the Server

A client might send this request:

```python
POST /api/portfolio
Content-Type: application/json

{
  "name": "Retirement",
  "owner": "Alice"
}
```

On the Flask side, the route can read and parse the JSON payload using the `request.get_json()` method:

```python
from flask import request, jsonify

@app.route("/api/portfolio", methods=["POST"])
def create_portfolio():
    data = request.get_json()
    name = data.get("name")
    owner = data.get("owner")

    # Simulate saving to the database
    return jsonify({"message": f"Portfolio '{name}' created for {owner}"}), 201
```

When Flask receives the request, it automatically converts the raw JSON text from the HTTP body into a Python dictionary so that your code can access the values easily.

In this case, `data` becomes:

```python
{"name": "Retirement", "owner": "Alice"}
```

This automatic parsing makes working with JSON intuitive: developers can use native Python types while Flask handles the translation.

---

### Sending JSON Data in the Response Body

When the server responds to a client, it often returns structured data—such as the details of a portfolio, an investment record, or a success message.
Flask provides the `jsonify()` helper to create JSON responses easily and safely.

#### Example: Returning JSON from Flask

```python
from flask import jsonify

@app.route("/api/portfolio/<int:portfolio_id>", methods=["GET"])
def get_portfolio(portfolio_id):
    portfolio = {
        "id": portfolio_id,
        "name": "Retirement",
        "owner": "Alice",
        "balance": 125000.50
    }
    return jsonify(portfolio), 200
```

Here, the dictionary `portfolio` is converted into a properly formatted JSON object, and Flask automatically sets the correct HTTP header:

```http
Content-Type: application/json
```

The client receives the response body:

```json
{
  "id": 1,
  "name": "Retirement",
  "owner": "Alice",
  "balance": 125000.5
}
```

Using `jsonify()` ensures that:

1. The response body follows valid JSON syntax.
2. Special characters are properly encoded.
3. The correct content type is added to the response headers.

---

### How Data Is Converted Between Python and JSON

While Flask makes working with JSON appear seamless, it is useful to understand what happens behind the scenes.
When data travels between the client and server, it must often change **form** to suit different contexts:

- Inside Python, data exists as **objects** (e.g., dictionaries, lists, classes).
- Over the network, data must be **serialized** into a plain-text representation that any system can read—JSON provides this textual encoding.

#### Conversion from Python to JSON (Serialization)

When you call `jsonify()` or use `json.dumps()`, Flask converts Python objects into their JSON equivalents:

```python
import json

data = {"name": "Alice", "balance": 1000.0, "active": True}
json_string = json.dumps(data)
print(json_string)
```

Output:

```json
{"name": "Alice", "balance": 1000.0, "active": true}
```

The conversion process maps Python types to JSON notations:

Python Type      | Converted JSON   |
---------------- | ---------------- |
`dict`           | JSON object      |
`list`           | JSON array       |
`str`            | string           |
`int` / `float`  | number           |
`True` / `False` | `true` / `false` |
`None`           | `null`           |

#### Conversion from JSON to Python (Deserialization)

When a Flask route receives a request with JSON data, calling `request.get_json()` converts that JSON string back into a Python data structure:

```python
from flask import request

data = request.get_json()
```

If the client sends:

```json
{"name": "Alice", "balance": 1000.0}
```

then `data` becomes a Python dictionary:

```python
{"name": "Alice", "balance": 1000.0}
```

This back-and-forth conversion is what allows two programs written in entirely different languages to communicate seamlessly. The client and server do not need to share the same internal representations—they only need to agree on the **JSON schema** for their communication.

---

### The Bigger Picture

Understanding JSON and how Flask handles it is essential for modern web development.
Data must move between layers—browser to API, API to database, and back again—and JSON provides the flexible bridge that connects them all.

At this stage, you have seen how:

- JSON structures information using simple key–value and list-based notation.
- Clients send JSON data in **request bodies** to transmit complex information.
- Servers return JSON data in **response bodies** to convey results.
- Flask automatically converts between Python’s internal structures and JSON text representation.

In the next section, we will explore how Flask integrates with extensions like **SQLAlchemy** to manage databases, showing how the data exchanged as JSON eventually becomes persistent records stored in a relational database.

## Extensions and Database Integration

### Extensibility in Flask Applications

One of Flask’s most powerful design features is its **extensibility**.
Flask itself provides only the minimal tools needed to build a web application—routing, request handling, and responses—but almost everything else (such as database access, authentication, caching, or form handling) can be added as **extensions**.

An extension is a **plugin** that integrates seamlessly with the Flask application object. Each extension encapsulates specialized functionality and can be registered at the time the app is created—typically inside the **application factory**.

This design allows you to:

- Keep your application modular and maintainable.
- Add or remove capabilities without modifying core code.
- Configure and initialize multiple extensions in one place.

For example, an application can register Flask-SQLAlchemy for database access, Flask-Migrate for migrations, and Flask-Login for authentication—all initialized at creation time.

### Example: Registering Extensions in the Factory Pattern

```python
# app/__init__.py
from flask import Flask
from .db import db
from .routes.portfolio import portfolio_bp

def create_app(config_class):
    app = Flask(__name__)
    app.config.from_object(config_class)

    # Register database extension
    db.init_app(app)

    # Register blueprints
    app.register_blueprint(portfolio_bp)

    return app
```

Each extension provides an `init_app(app)` method.
By calling this inside the factory, you “attach” the extension to the Flask app instance. This approach decouples the extension’s setup from its declaration—allowing the same extension object to be reused in testing or different configurations.

---

### Introducing SQLAlchemy for Flask

In earlier lessons, SQLAlchemy was introduced as a powerful Python ORM that maps database tables to Python classes. Flask can integrate with SQLAlchemy through the **Flask-SQLAlchemy** extension, which simplifies connection management, configuration, and ORM session usage.

#### 1. Extension Registration

To start using SQLAlchemy with Flask, first create a `db` object from the extension:

```python
# app/db.py
from flask_sqlalchemy import SQLAlchemy

db = SQLAlchemy()
```

This `db` object is an instance of the `SQLAlchemy` class—it acts as a bridge between Flask and the underlying SQLAlchemy engine.
It is **not yet bound** to any application. To bind it, you call `db.init_app(app)` in your factory, as shown earlier. This is the step that registers the extension with your Flask app.

By using this pattern, you can create multiple independent Flask applications (for example, for testing or production) that each use the same `db` object with different configurations.

---

#### 2. Database Configuration

Flask-SQLAlchemy reads its configuration from the app’s settings. These can be defined in your configuration class, typically in `config.py`:

```python
# app/config.py
import os

class Config:
    SQLALCHEMY_DATABASE_URI = os.getenv("DATABASE_URL", "sqlite:///local.db")
    SQLALCHEMY_TRACK_MODIFICATIONS = False
    DEBUG = True
```

The key configuration options include:

- `SQLALCHEMY_DATABASE_URI` – defines the connection string for your database.

  - Example:
    `mysql+pymysql://user:password@localhost:3306/investments_db`
- `SQLALCHEMY_TRACK_MODIFICATIONS` – disables unnecessary overhead related to object tracking (should always be set to `False`).
- Other optional parameters, such as `SQLALCHEMY_ECHO=True`, can be used for debugging SQL queries.

When the app factory calls `app.config.from_object(Config)`, these values are automatically loaded into the Flask app and picked up by the SQLAlchemy extension during initialization.

---

#### 3. Defining Models

Once `db` has been registered, you can define your ORM models by extending `db.Model`. This allows you to map Python classes to database tables.

```python
# app/models/portfolio.py
from app.db import db

class Portfolio(db.Model):
    __tablename__ = "portfolio"

    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(100), nullable=False)
    owner = db.Column(db.String(100), nullable=False)
    balance = db.Column(db.Float, default=0.0)
```

Each attribute represents a database column, and each instance of the class represents a row in the table.
SQLAlchemy automatically manages the table schema, relationships, and type conversions behind the scenes.

---

#### 4. Using Database Sessions in Routes

A **session** represents an active connection between the Flask application and the database. It handles the communication layer for reading and writing data. Flask-SQLAlchemy provides a session object accessible as `db.session`.

Within a route, you can use this session to query, add, or delete records.

#### Example: Adding a New Portfolio

```python
from flask import request, jsonify
from app.db import db
from app.models.portfolio import Portfolio

@app.route("/api/portfolio", methods=["POST"])
def add_portfolio():
    data = request.get_json()
    name = data.get("name")
    owner = data.get("owner")

    portfolio = Portfolio(name=name, owner=owner)
    db.session.add(portfolio)
    db.session.commit()

    return jsonify({"message": f"Portfolio '{name}' created successfully."}), 201
```

Here’s what happens step by step:

1. The request arrives with JSON data in the body.
2. Flask converts the JSON into a Python dictionary using `request.get_json()`.
3. A `Portfolio` object is created and staged using `db.session.add()`.
4. `db.session.commit()` writes the changes permanently to the database.

If something goes wrong, you can roll back safely:

```python
try:
    db.session.add(portfolio)
    db.session.commit()
except Exception as e:
    db.session.rollback()
    return jsonify({"error": str(e)}), 500
```

This pattern ensures database consistency—any failed transaction is reversed.

---

#### 5. Querying and Returning Data

Retrieving data follows the same pattern. You query through the session and return results as JSON responses.

```python
@app.route("/api/portfolio/<int:portfolio_id>", methods=["GET"])
def get_portfolio(portfolio_id):
    portfolio = Portfolio.query.get(portfolio_id)
    if not portfolio:
        return jsonify({"error": "Portfolio not found"}), 404

    result = {
        "id": portfolio.id,
        "name": portfolio.name,
        "owner": portfolio.owner,
        "balance": portfolio.balance
    }
    return jsonify(result), 200
```

`Portfolio.query.get()` fetches the record with the given primary key. Flask then returns it as a JSON response.
This simple flow—query, map, return—is the foundation for building RESTful API endpoints.

---

#### 6. Why the Extension Pattern Matters

By isolating database logic inside the extension and initializing it through `init_app()`, you achieve several architectural advantages:

- **Modularity** – The database setup lives in one place (`db.py`), not scattered across your code.
- **Reusability** – The same `db` instance can serve multiple environments (development, testing, production).
- **Testability** – You can create test apps with in-memory databases by passing different configurations to `create_app()`.
- **Scalability** – Other extensions (e.g., Flask-Migrate, Flask-Login) can follow the same registration pattern without conflict.

---

### Summary

Flask’s extension mechanism allows applications to grow from simple prototypes into robust, production-ready systems.
Through the **factory pattern**, extensions such as **Flask-SQLAlchemy** can be registered cleanly and configured dynamically.

Key takeaways:

- Flask apps remain **lightweight and modular** because each extension manages its own lifecycle.
- SQLAlchemy integrates seamlessly as a Flask extension via the `db` object.
- Configuration values (like database URIs) are defined centrally and loaded automatically.
- Routes interact with the database through **sessions**, enabling querying, adding, and committing changes efficiently.

With this setup, the application gains a persistent data layer that bridges the web service routes to relational databases—forming the foundation for the next stage of building a full-featured, database-backed web application.
