# Object–Relational Mapping (ORM) in Python with SQLAlchemy

## Relational databases

Relational databases organize information into tables made of rows and columns, and they connect those tables through _relationships_. The relational model is decades-tested, powers everything from banking to streaming platforms, and excels at data integrity, concurrency, and querying. If an app tracks portfolios, investments, users, and prices, a relational database naturally represents each as a table, with keys that tie them together.

> A useful analogy: imagine a set of spreadsheets where each sheet is a table and some cells contain “references” to rows on other sheets. A relational database is that idea—formalized, efficient, and consistent—plus a powerful language (SQL) to ask questions of the data.

## What an ORM is—and why it helps

An Object–Relational Mapper maps database tables to Python classes and rows to Python objects. Instead of hand-crafting SQL for every operation, you work with objects: create, query, update, and delete them. The ORM:

- Tracks objects and changes (unit of work) and persists them efficiently.
- Handles joins and relationships as attribute access.
- Centralizes schema knowledge in model definitions instead of scattering SQL strings across the codebase.
- Improves testability by letting you swap backends or use transactions/fixtures per test.

Think of the ORM as an interpreter between two languages: Python’s object world and SQL’s relational world.

### Why not just write raw SQL in the code?

It’s tempting to paste SQL strings into Python and call it a day. The trouble is that embedding SQL everywhere couples application logic to the database layer. Over time, it becomes difficult to refactor schemas, change vendors, or reuse logic. Testing also gets harder when logic and storage are intertwined.

> A gentle rule of thumb: if your business logic knows too much about table/column names, migrations become costly. ORMs help separate concerns: Python code expresses “what” the app needs; the ORM translates it into “how” the database runs it.

## SQL Alchemy

SQLAlchemy is a powerful and widely used Python library that provides tools for working with relational databases in an object-oriented way. It serves as both a database toolkit and an Object–Relational Mapper (ORM), allowing developers to interact with databases using Python classes and objects instead of writing raw SQL queries. SQLAlchemy handles the translation between Python code and SQL commands automatically, managing tasks such as table creation, relationships, transactions, and query execution. This abstraction layer makes applications easier to maintain, test, and extend, while still giving developers full control to drop down to raw SQL when needed.

SQLAlchemy is the most widely used ORM in Python. It has two layers:

1. Core: a powerful SQL expression toolkit.
2. ORM: built atop Core; provides identity maps, sessions, relationships, and model mapping.

You can drop to Core when you need precise SQL and stay in ORM for day-to-day work—best of both worlds.

### Connecting to a database with SQL Alchemy

A clean application isolates database setup in a single module. Other modules never create engines or sessions directly; they import helpers from db. Below is a small but production-friendly for ```db.py```.

```python
# db.py
from sqlalchemy import create_engine
from sqlalchemy.orm import DeclarativeBase, sessionmaker, Session

# 1) Declarative Base all models will extend
class Base(DeclarativeBase):
    pass

# 2) Build the engine once
DATABASE_URL = "replace with database connection string"
engine = create_engine(
    DATABASE_URL,
    echo=False, # set True while debugging SQL
)

# 3) Session factory
SessionLocal = sessionmaker(
    bind=engine,
    autoflush=False,
    autocommit=False,
    expire_on_commit=False # keeps objects usable after commit
)

# 4) A dedicated function to get a session
def get_session() -> Session:
    return SessionLocal()

```

The ```sessionmaker()``` function in SQLAlchemy is used to create a factory for sessions—objects that manage the connection between your Python code and the database. You can think of it like a “session template” that defines how all sessions in your app should behave. Its most common parameters fine-tune how sessions interact with the database.

- The ```bind``` parameter attaches the session factory to an engine, which acts like the physical “doorway” to your database; every session you create will walk through that same door.
- The ```autoflush``` parameter, when set to True, automatically pushes changes (like new or updated objects) to the database before running queries—like a notepad that syncs automatically before you check what’s written.
- ```autocommit```, which is almost always left as False, determines whether transactions are committed automatically after each operation or manually via ```session.commit()```; in practice, you want control, so manual commits are safer.
- Finally, ```expire_on_commit``` controls whether objects “forget” their data after committing. When True, SQLAlchemy will refresh them from the database next time they’re accessed—like clearing your browser cache so you always see the latest data. When False, the session keeps their state in memory, which is often convenient for simple applications.

Best practices embedded here:

- One engine per process.
- Centralized session factory.
- expire_on_commit=False so objects don’t “forget” their values right after a commit (useful for simple apps; you can revisit later).
- A single, obvious way to get a session: get_session().

### Connection Strings

A connection string in SQLAlchemy defines how your Python application connects to the database — it’s essentially the database’s address plus credentials. For a MySQL database, the connection string follows this general format:

```python
mysql+pymysql://username:password@host:port/database_name
```

Here, ```mysql+pymysql``` specifies the database dialect (mysql) and the Python driver (pymysql) used to communicate with it. The ```username``` and ```password``` are the database credentials, ```host``` is the server address (often localhost for local development), ```port``` is typically 3306 for MySQL, and ```database_name``` is the specific schema to connect to.

For example, a valid connection string might look like this:

```python
DATABASE_URL = "mysql+pymysql://root:mysecret@localhost:3306/investments_db"
```

This string can then be passed to SQLAlchemy’s ```create_engine()``` function to establish a connection.

>In production environments, it’s best practice to store the connection string in an environment variable rather than hardcoding it in your source code.

### Using session with SQL Alchemy

A ```Session``` is a _conversational_ workspace with the database. It:

- Tracks objects you load or create.
- Stages changes and flushes them to SQL when needed.
- Coordinates transactions (commit/rollback).

> Crucial idea: the session is the “unit of work.” Do a bit of work, then commit or rollback, then move on. Keep sessions short-lived and scoped to a request or a command in your CLI.

How to obtain one in this app: always call ```get_session()``` from ```db```.

```python
# anywhere in your code
from db import get_session

session = get_session()
    # do queries, adds, deletes
    ...
```

#### Common Session Operations

##### Insert a new row

```python
from db import get_session
from models import Portfolio
try:
    session = get_session()
    p = Portfolio(name="Retirement")
    session.add(p)
    session.commit()
except Exception as e:
    session.rollback() if session else None
    raise e
finally:
    session.close() if session else None
```

##### Query data

```python
from sqlalchemy import select
from db import get_session
from models import Portfolio

try:
    session = get_session()
    # query all rows
    portfolios = session.query(Portfolio).all()
    # query by id
    portfolio = session.query(Portfolio).filter_by(id=1).one_or_none()
    # query with a filter
    special_portfolio = session.execute(select(Portfolio).where(Portfolio.name.ilike("%retire%"))).scalars().all()
except Exception as e:
    session.rollack() if session else None
    raise e
finally:
    session.close()
```

##### Delete an existing row

```python
from db import get_session
from models import Portfolio

try:
    session = get_session()
    portfolio_to_delete = session.query(Portfolio).filter_by(id=1).one_or_none()
    session.delete(portfolio_to_delete)
    session.commit()
except Exception as e:
    session.rollback() if session else None 
    raise e
finally:
    session.close() if session else None
```

## Data modeling with SQL Alchemy

Models extend ```Base``` from the ```db``` module and declare columns/relationships. Keep models in their own package (for example, ```models/```), and ensure an __init__.py imports them so mappers register once.

> Imports with SQL alchemy can become tricky and cause errors that are hard to troubleshoot. In order to carefully understand how python import system interacts with alchemy's see notes here. It is always recommended to follow best practice in order to avoid tricky errors.

File structure:

```python
app/
  db.py
  models/
    __init__.py
    portfolio.py
    investment.py
    security.py
```

models/__init__.py (central importer so everything registers):

```python
from .portfolio import Portfolio
from .investment import Investment
from .security import Security

__all__ = ["Portfolio", "Investment", "Security"]
```

In SQLAlchemy, you’ll often see model classes defined using two key pieces of syntax:
```Mapped[...]``` and ```mapped_column(...)```. These bring stronger typing, better integration with IDEs and static analysis, and a more declarative, modern feel to your models. Alongside these, the ```relationship()``` function defines how model classes are linked together.

### ```Mapped``` and ```mapped_column```

When defining a SQLAlchemy model, every class attribute that maps to a database column must be declared using type annotations and mapping functions.

Example:

```python
from sqlalchemy.orm import Mapped, mapped_column
from sqlalchemy import String, Integer

class Portfolio(Base):
    __tablename__ = "portfolio"

    id: Mapped[int] = mapped_column(Integer, primary_key=True)
    name: Mapped[str] = mapped_column(String(100), nullable=False)
```

#### ```Mapped[...]```

- It’s a generic type that tells both SQLAlchemy and type checkers (like Pylance or mypy) that the attribute is mapped to a column in the database.
- The type inside the brackets (int, str, float, etc.) represents the Python type of the column when accessed through the ORM.
- Mapped helps IDEs provide autocompletion and type checking, improving readability and correctness.

#### ```mapped_column(...)```

- This function actually defines the database column.
- It replaces the older Column(...) used in SQLAlchemy 1.x.
- It specifies database-level details like data type, constraints, and behaviors.

For example:

```python
balance: Mapped[float] = mapped_column(Float, default=0.0, nullable=False)
```

Here:

- Float → the SQL data type.
- default=0.0 → sets a default value.
- nullable=False → marks the column as required.

You can also include foreign keys:

```python
portfolio_id: Mapped[int] = mapped_column(ForeignKey("portfolio.id"))
```

In short:

- Mapped → tells the ORM “this attribute is part of the mapping.”
- mapped_column → defines how that mapping appears in the database schema.

### The ```relationship()``` function

While mapped_column() maps columns (actual database fields), relationship() maps Python-level links between classes — for example, “a portfolio has many investments.”

A relationship connects model classes much like foreign keys connect tables. In SQLAlchemy:

- The foreign key lives on the column (mapped_column(ForeignKey("other_table.pk"))).
- The relationship lives on the class and refers to the other class, not the table name; it enables attribute-style navigation and loading strategies.

Two practical guidelines:

1. Always define the actual foreign key on the child column (e.g., Investment.portfolio_id → portfolio.id).
2. Mirror relationships with back_populates on both sides so navigation is symmetric and SQLAlchemy understands ownership.

When in doubt, start with one-to-many and use lazy="selectin" on the collection side to avoid N+1 queries when loading multiple parents.

Example:

```python
investments: Mapped[list["Investment"]] = relationship(
    "Investment",
    back_populates="portfolio",
    cascade="all, delete-orphan",
    lazy="selectin"
)
```

Let’s break down the key parameters.

#### ```relationship(target, ...)```

- target: The name of the related ORM class as a string, e.g., "Investment".
Using a string allows SQLAlchemy to resolve circular dependencies when both classes reference each other.

#### ```back_populates```

- Creates a two-way relationship.
- If Portfolio has investments = relationship("Investment", back_populates="portfolio"),
then Investment must have portfolio = relationship("Portfolio", back_populates="investments").
- This keeps both sides synchronized — assigning one updates the other automatically.

#### ```foreign_keys```

- Explicitly specifies which column(s) act as the foreign key(s) when multiple possible keys exist.
- Example:

```python
investment_owner: Mapped["User"] = relationship("User", foreign_keys=[user_id])
```

#### ```cascade```

- Defines what happens to related objects when a parent is deleted or updated.
- Common values:
  - "all, delete-orphan" → deleting the parent also deletes the children.
  - "save-update" → saves changes to related objects automatically.

### Putting it all together

models/portfolio.py

```python
from __future__ import annotations
from typing import TYPE_CHECKING

from sqlalchemy import String
from sqlalchemy.orm import Mapped, mapped_column, relationship

from db import Base

if TYPE_CHECKING:
    from .investment import Investment  # for type hints only

class Portfolio(Base):
    __tablename__ = "portfolio"

    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str] = mapped_column(String(100), unique=True, nullable=False)

    # one-to-many: a portfolio has many investments
    investments: Mapped[list["Investment"]] = relationship(
        back_populates="portfolio"
    )
```

## Conclusion

Relational databases provide durable, consistent storage; ORMs provide a clear vocabulary for working with that storage as Python objects. SQLAlchemy bridges these worlds with a robust, well-understood toolkit. Centralizing database concerns in a db module (engine, Base, and get_session()), keeping sessions short-lived and explicit, and modeling relationships with proper foreign keys and back_populates yields code that is easier to read, test, and evolve.

As the application grows, this foundation allows new features—additional models, more complex queries, migrations, and even background jobs—to slot in without rewriting how the app talks to the database.
