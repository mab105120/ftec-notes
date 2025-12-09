# Final Exam - Sample Questions

## Part I — Sample Multiple Choice (5 Questions)

Select the best answer.

1. In a REST API, why is it recommended to return a consistent JSON envelope such as { "data": ..., "error": ... }?

    a) It improves CPU performance  
    b) It allows clients to parse results and errors using predictable structures  
    c) It makes all responses smaller  
    d) It eliminates the need for HTTP status codes  

2. In SQLAlchemy, when does a pending object (newly created with no ID) receive its primary key value?

    a) When the object is instantiated  
    b) When the object is added to the session  
    c) When the session flushes during commit  
    d) When the database connection opens  

3. What is the effect of defining a Pytest fixture with scope="module"?

    a) It is recreated before every test function  
    b) It runs once per test module and the same instance is shared across tests  
    c) It is executed only if a test explicitly calls it  
    d) It disables other fixtures with smaller scopes  

4. In Flask, which of the following is true about request.args?

    a) It contains JSON data sent in the body  
    b) It contains path parameters extracted from the URL pattern  
    c) It contains query parameters from the URL  
    d) It contains headers related to authentication  

## Part II — Sample Code Evaluation (2 Questions)

Explain behavior, identify issues, or provide corrected versions.

1. Handling Missing Fields in JSON Input

```python
@app.route("/add", methods=["POST"])
def add_item():
    data = request.get_json()
    name = data["name"]
    qty = int(data["qty"])
    return {"added": True}
```

a) Describe a realistic way this code might fail with real-world client requests.  
b) Suggest two improvements to make it more robust.

2. Misconfigured Session Usage

```python
def update_price(session, item_id, new_price):
    item = session.get(Item, item_id)
    item.price = new_price
    session.flush()
    session.close()
    return True
```

a) Identify the problem with the order in which operations occur.  
b) Rewrite the safe and correct sequence for updating a database record.

## Part III — Sample Short Essay Questions (2 Questions)

Write 3–5 sentence responses.

1. Explain how Pytest's fixture system contributes to modular, maintainable tests. In your answer, discuss how fixture scopes influence test behavior and why dependency injection matters.
2. Discuss why data validation is a critical part of web service design. Explain how Flask applications typically handle validation failures and why consistent error structures benefit client developers.

----

## Answer Key

### Part I — Multiple Choice Answers

Question|Correct Answer|Explanation|
|--|--|--|
1|b|Consistent JSON envelopes help clients reliably parse responses and errors.|
2|c|SQLAlchemy assigns primary keys during a flush, usually triggered by commit().|
3|b|A module-scoped fixture runs once per module and is shared across tests inside it.|
4|c|request.args stores query parameters from the URL.|
5|c|The Flask test client simulates requests without running a real HTTP server.|

### Part II — Code Evaluation Answers

1. Handling Missing Fields in JSON Input

a) How it might fail

Several realistic failure scenarios:

- lient sends empty body → data becomes None, causing data["name"] to raise TypeError.
- Client omits "qty" or "name" → KeyError.
- Client sends "qty" as a non-numeric string (e.g., "ten") → ValueError when casting to int.

b) Improvements

Any two of the following are acceptable:

- Use request.get_json(silent=True) or if not data: checks.
- Validate the presence of required fields:

```python
if "name" not in data or "qty" not in data:
    return {"error": "Missing fields"}, 400
```

- Wrap conversions in try/except.
- Return structured error responses with appropriate status codes.

2. Misconfigured Session Usage

a) Problem with operation order

- The code calls session.flush() before committing and then closes the session immediately, leaving the transaction incomplete.
- Closing the session before commit() means changes might not be persisted.
- Accessing ORM objects after closing the session may cause DetachedInstanceError.

b) Correct workflow

A valid corrected sequence:

```python
def update_price(session, item_id, new_price):
    try:
        item = session.get(Item, item_id)
        item.price = new_price
        session.commit()     # Ensures durability
        return True
    except Exception:
        session.rollback()
        return False
    finally:
        session.close()
```

Key ideas:

- Modify object before commit.
- Commit ensures persistence.
- Rollback guards against partial writes.
- Close session in `finally`.

### Part III — Short Essay Expected Points

Below are rubric-style expectations, not verbatim required text.

1. Pytest fixtures and maintainability

    A complete answer should mention:

    - Fixtures separate setup logic from test logic.
    - Dependency injection: Pytest detects fixture names and supplies them automatically.
    - Encourages modularity: multiple tests reuse the same setup.
    - Fixture scopes (function, module, etc.) influence performance and object sharing.
    - Improves readability and reduces duplication.

2. Importance of data validation in web services

    A complete answer should include:

    - Validation protects application logic from malformed inputs.
    - Ensures predictable server behavior and reduces risk of runtime exceptions.
    - Flask apps typically validate JSON, route parameters, or form data; failures return structured JSON errors with appropriate status codes.
    - Consistent error formats help frontend developers reliably handle error states.
    - Prevents security and correctness issues arising from untrusted inputs.
