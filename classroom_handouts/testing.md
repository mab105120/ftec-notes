# Introduction to Software Testing with Pytest

Software testing is one of the most important skills for any aspiring software engineer. Whether you are building a small command-line tool or a production-grade web application, the quality of your software depends on your ability to detect defects early, prevent regressions, and ensure that every part of your code behaves as intended. This handout provides a guided introduction to the core ideas of software testing, demonstrates how testing is done in Python using the Pytest framework, and explains several essential testing concepts such as fixtures, mocking, database testing, and coverage analysis.

## Why Software Testing Matters

Software testing is the disciplined process of evaluating whether a program meets its requirements and behaves correctly under different conditions. Without testing, software development becomes guesswork. Developers might hope that the code works, but they have no systematic way to verify correctness, catch regressions, or prevent bugs from re-appearing.

Testing brings several benefits. First, it improves reliability by detecting defects early in the development cycle when they are easier to fix. Second, it improves maintainability by giving developers confidence to refactor or extend the code. Third, it documents behavior: a well-written test suite serves as a living specification of what the system is supposed to do.

A useful analogy is the construction of a bridge. Engineers do not simply hope that a bridge will withstand the load; they perform stress tests, simulations, and inspections. Testing in software fulfills the same role: it ensures that the structure we build behaves as expected under real-world conditions.

## Types of Testing and the Limitations of Manual Testing

Testing takes many forms, each serving a different purpose. Unit testing focuses on the smallest building blocks of a program: individual functions and methods. Integration testing checks how components work together. System testing evaluates the entire application end to end. Acceptance testing ensures that the system meets business and user requirements. Performance testing focuses on speed and scalability.

Manual testing, as the name suggests, relies on human testers interacting with the system by executing test scenarios themselves. While this approach can be effective for tiny applications, it does not scale. Humans are slow, inconsistent, and prone to error. Repeating the same test manually every time the code changes becomes tedious and unmanageable. Automated testing, by contrast, allows tests to be executed quickly, consistently, and automatically whenever the code is run or deployed.

A helpful way to understand the difference is to imagine proofreading a book. Manually reading the entire book every time you make a small change would be impractical. Automated spell checkers and grammar tools help you detect issues instantly. Automated tests play the same role for software.

## Introduction to Python Unit Testing with Pytest

Pytest is one of the most widely used testing frameworks in Python due to its simplicity, flexibility, and rich ecosystem. It allows developers to write tests using plain Python functions without boilerplate code. Pytest automatically discovers test files and test functions following a simple naming convention.

A recommended folder structure for Pytest looks like this:

```python
project/
    app/
        __init__.py
        calculator.py
    tests/
        __init__.py
        test_calculator.py
```

Here is a minimal example to illustrate the flow. Suppose we have a simple calculator module:

```python
# app/calculator.py
def add(x, y):
    return x + y
```

We can write a test for it in `tests/test_calculator.py`:

```python
# tests/test_calculator.py
from app.calculator import add

def test_add_numbers():
    result = add(2, 3)
    assert result == 5
```

Running pytest from the project root will automatically detect and execute this test. Pytest’s strength lies in its ability to remain simple for beginners while supporting advanced testing techniques for larger applications.

## Fixtures and Dependency Injection in Pytest

Fixtures are one of the defining features of Pytest. A fixture is a reusable piece of setup code that prepares the environment needed for a test. Instead of repeating setup logic across test functions, developers create fixtures that Pytest automatically injects into tests as function parameters.

To understand how this works, it helps to recall how modern factories operate. Instead of each worker gathering their own tools every time, the factory supplies the needed equipment at the workstation. Fixtures provide this “equipment” for tests.

Here is a simple fixture:

```python
import pytest

@pytest.fixture
def sample_data():
    return [1, 2, 3]
```

A test can access this fixture simply by naming it as a parameter:

```python
def test_list_length(sample_data):
    assert len(sample_data) == 3
```

Pytest uses what is known as dependency injection: instead of the test asking explicitly for the data by calling a helper function, Pytest inspects the test’s parameters and supplies the fixture automatically.

Fixtures can have different scopes:

- Function scope: executed once per test function.
- Class scope: executed once per test class.
- Module scope: executed once per module.
- Session scope: executed once per entire test run.

For example:

```python
@pytest.fixture(scope="module")
def db_connection():
    # Setup a connection
    conn = ...
    yield conn
    # Teardown logic
```

The choice of scope affects performance and correctness. For instance, expensive resources like database engines may benefit from a wider scope, while objects that must be fresh for each test should use function scope.

## Mocking and Monkeypatch

In real-world applications, many components depend on external systems: databases, APIs, email providers, or expensive computations. In tests, we often do not want to interact with the real systems. Instead, we replace those dependencies with controlled substitutes, a process known as mocking.

Mocking is essential because it isolates the unit under test. Testing a function that sends emails, for example, should not actually send emails every time the test suite runs. Instead, the test should replace the real email-sending function with a fake version that simulates the behavior.

Pytest provides a built-in tool called monkeypatch for lightweight mocking. Here is an example:

```python
def get_username():
    # Imagine this function fetches from the OS environment
    import os
    return os.getenv("USER")

def test_get_username(monkeypatch):
    monkeypatch.setenv("USER", "testuser")
    assert get_username() == "testuser"
```

Using monkeypatch, we modify the environment variable temporarily within the context of the test. After the test ends, Pytest restores the original environment, helping ensure isolation and repeatability.

## Case Study: Testing an Application That Connects to a MySQL Database

Many applications interact with databases, and testing such systems requires careful design. You do not want your tests modifying production or development databases. Instead, you should create a separate test database, or preferably use an in-memory database whenever possible.

While MySQL itself does not support an in-memory system the way SQLite does, SQLAlchemy allows you to swap the engine during tests. For unit testing logic that uses SQLAlchemy, SQLite’s in-memory database is sufficient and fast.

Below is an example that demonstrates how to configure fixtures to test database interactions:

`app/models.py`

```python
from sqlalchemy import Column, Integer, String
from sqlalchemy.orm import declarative_base

Base = declarative_base()

class User(Base):
    __tablename__ = "users"

    id = Column(Integer, primary_key=True)
    name = Column(String(50))
```

`tests/conftest.py`

```python
import pytest
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from app.models import Base

@pytest.fixture(scope="session")
def engine():
    engine = create_engine("sqlite:///:memory:", echo=False)
    Base.metadata.create_all(engine)
    return engine

@pytest.fixture(scope="function")
def db_session(engine):
    connection = engine.connect()
    transaction = connection.begin()

    Session = sessionmaker(bind=connection)
    session = Session()

    yield session

    session.close()
    transaction.rollback()
    connection.close()
```

`tests/test_user.py`

```python
from app.models import User

def test_create_user(db_session):
    user = User(name="Alice")
    db_session.add(user)
    db_session.commit()

    assert db_session.query(User).count() == 1
```

## Test Coverage with pytest-cov

Coverage analysis measures how much of your code is executed when tests run. High coverage can indicate a well-tested system, although it does not guarantee that tests are meaningful. For example, a test that only checks whether a function runs without errors might technically produce high coverage but still fail to catch logical errors.

Coverage is useful for identifying untested code paths, dead code, and missing edge-case tests.

Using pytest-cov, you can generate a coverage report with the following command:

`pytest --cov=app --cov-report=term-missing`

The term-missing option shows lines that were not executed during tests. You can also generate HTML reports:

`pytest --cov=app --cov-report=html`

This produces an htmlcov directory containing a clickable report that highlights covered and uncovered lines.

Coverage numbers should be used as guidance, not as the sole metric of test quality. A thoughtful test suite values correctness and clarity above raw percentages.
