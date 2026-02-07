# Transactional Boundaries in Flask Web Services

## Introduction

When building web services, ensuring data integrity is critical. This handout explores how to structure Flask applications to treat each HTTP request/response cycle as an atomic unit of work. By establishing proper transactional boundaries, we can guarantee that database operations either complete entirely or rollback completely—preventing partial updates that could corrupt application state.

## Flask Application Context

Flask creates an **application context** for every incoming request. This context provides a request-scoped environment where Flask stores request-specific data and manages resources. The context is created when a request arrives and is destroyed after the response is returned.

This context mechanism is crucial because it allows Flask extensions, including SQLAlchemy, to automatically manage resources that should exist only for the duration of a single request.

## Understanding SQLAlchemy Scoped Sessions

Before discussing transactional boundaries, we must understand how SQLAlchemy manages database sessions in Flask applications.

### What is a Scoped Session?

A **scoped session** is SQLAlchemy's mechanism for providing thread-safe, context-aware database sessions. Think of it as a smart container that ensures each request gets its own dedicated database session without manual session management.

**Analogy**: Imagine a library where each visitor receives a personal reading cart when they enter. The cart follows them throughout their visit, holding all the books they want to check out. When they leave, the cart is returned and cleared. The library doesn't give them someone else's cart, nor does their cart persist after they leave. This is how scoped sessions work—each request gets its own "cart" (session) that exists only for that request's lifetime.

### Session Lifecycle

In a Flask application configured with SQLAlchemy:

1. **Session Creation**: The first time a request accesses the database (e.g., executing a query), SQLAlchemy automatically creates a session scoped to that request's application context.
2. **Session Reuse**: All subsequent database operations within the same request reuse this session.
3. **Session Destruction**: When the response is returned and the application context is torn down, the session is automatically closed and removed.

This automatic lifecycle management is powerful, but it requires careful handling of transactions to maintain data integrity.

## The Unit of Work Pattern

The **Unit of Work pattern** treats the entire request/response cycle as a single transactional boundary. This means:

- A transaction begins when a request is received
- All database operations during the request are tracked by a single session
- The transaction commits only after all operations succeed
- Any failure triggers a complete rollback of all changes

### The Critical Rule: Service Functions Must Never Commit

When implementing the Unit of Work pattern, **service functions** (business logic functions called by route handlers) should **never commit changes** to the database. Here's why:

If a service function commits changes, SQLAlchemy finalizes the current transaction and starts a new session. Subsequent operations occur in this new session, making it impossible to rollback earlier changes if a later operation fails.

**Example Scenario**: Suppose a request calls two service functions—one to create a user account and another to create an associated profile. If the first function commits the user creation, and the profile creation fails, the rollback will only affect the profile. The user account remains in the database, resulting in an orphaned record and inconsistent state.

## Code Examples

### Anti-Pattern: Committing in Service Functions

```python
# services.py - INCORRECT APPROACH
def liquidate_investment(portfolio_id, ticker, quantity):
    portfolio = get_portolio_by_id(portfolio_id)
    investments = portfolio.investments
    for investment in investments:
        if investment.ticker == ticker:
            investment.quantity -= quantity
    db.session.commit()  # DON'T DO THIS

def update_balance(user_id, proceeds):
    user = get_user_by_id(user_id)
    user.balalnce += proceeds
    db.session.commit()  # DON'T DO THIS

# routes.py
@app.route('/liquidate-investment', methods=['POST'])
def register():
    data = request.json
    liquidate_investment(data['portfolio_id'], data['ticker'], data['quantity'])
    update_balance(data['user_id'], proceeds)  # If this fails, investment is already liquidated
    return jsonify({'message': 'investment liquidated'}), 201
```

**Problem**: If `create_profile()` raises an exception, the user has already been committed to the database, resulting in incomplete data.

### Correct Pattern: Commit Only at Route Level

```python
# services.py - CORRECT APPROACH
def liquidate_investment(portfolio_id, ticker, quantity):
    portfolio = get_portolio_by_id(portfolio_id)
    investments = portfolio.investments
    for investment in investments:
        if investment.ticker == ticker:
            investment.quantity -= quantity
    # NO COMMIT HERE

def update_balance(user_id, proceeds):
    user = get_user_by_id(user_id)
    user.balalnce += proceeds
    # NO COMMIT HERE

# routes.py
@app.route('/liquidate-investment', methods=['POST'])
def register():
    data = request.json
    liquidate_investment(data['portfolio_id'], data['ticker'], data['quantity'])
    update_balance(data['user_id'], proceeds)
    # commit here:
    db.session.commit()
    return jsonify({'message': 'investment liquidated'}), 201
```

**Benefit**: Both operations are part of the same transaction. If `create_profile()` fails, the rollback includes the user creation, maintaining atomicity.

## Error Handling and Rollbacks

Proper error handling ensures that exceptions trigger automatic rollbacks and return meaningful responses to clients.

### Exception Propagation

Service functions should **raise exceptions** rather than handling them. The route handler (or a global error handler) is responsible for catching exceptions and formatting error responses.

```python
# services.py
def create_user(username, email):
    if User.query.filter_by(email=email).first():
        raise ValueError(f"User with email {email} already exists")
    
    user = User(username=username, email=email)
    db.session.add(user)
    return user
```

### Flask Global Error Handlers

Flask provides a mechanism for centralized error handling using the `@app.errorhandler()` decorator. This allows consistent error responses across the application and automatic rollback on failures.

```python
# app.py or error_handlers.py
@app.errorhandler(ValueError)
def handle_value_error(error):
    db.session.rollback()  # Explicit rollback
    return jsonify({'error': str(error)}), 400

@app.errorhandler(Exception)
def handle_generic_error(error):
    db.session.rollback()  # Rollback on any unhandled exception
    return jsonify({'error': 'An internal error occurred'}), 500

# routes.py
@app.route('/register', methods=['POST'])
def register():
    data = request.json
    user = create_user(data['username'], data['email'])  # May raise ValueError
    profile = create_profile(user.id, data['bio'])
    
    db.session.commit()
    return jsonify({'id': user.id}), 201
```

**Note**: SQLAlchemy typically rolls back the session automatically when an exception occurs within a transaction. However, explicit rollbacks in error handlers ensure consistency, especially for custom error handling logic.

## Best Practices Summary

1. **Treat each request/response cycle as a transactional boundary**: Begin transactions implicitly when the request arrives; commit only after all operations succeed.

2. **Never commit in service functions**: Service functions should only prepare changes (add, update, delete). The route handler commits after all service calls complete.

3. **Raise exceptions in service functions**: Let errors propagate to route handlers or global error handlers rather than catching them in service functions.

4. **Use global error handlers**: Implement Flask's `@app.errorhandler()` to centralize error handling, ensure rollbacks, and provide consistent API responses.

5. **Single commit per request**: Execute `db.session.commit()` exactly once at the end of the route handler, after all business logic succeeds.

6. **Trust scoped sessions**: Let SQLAlchemy manage session lifecycle automatically—do not manually create or close sessions in most cases.

---

By following the Unit of Work pattern, your Flask applications maintain strong data integrity guarantees, ensuring that each request either completes fully or leaves no trace—preventing inconsistent database states and simplifying error recovery.
