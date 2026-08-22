# Managing Imports with SQL Alchemy

A conceptual guide to building ORM models without confusion or circular imports

## Introduction: Where the Confusion Begins

Working with SQLAlchemy often feels straightforward until models begin referencing each other.
At that point, obscure errors may appear:

- “ImportError: cannot import name…”  
- “MappedAnnotationError: expected a Python type but got a module…”

The source of these issues lies at the intersection of Python’s import system and SQLAlchemy’s ORM mapping mechanism.
The two operate on different assumptions about when and how names are resolved.
Understanding both systems together is the key to designing models that interact cleanly.

## How SQLAlchemy Sees the World

SQLAlchemy’s ORM allows Python classes to represent database tables.
Each model defines a table name, its columns, and relationships to other tables.
In small projects, all models might live in a single file.
As the application grows, they are typically divided into separate modules:
User.py, Portfolio.py, Investment.py, and so on.

The difficulty begins when those files import each other.
If Portfolio.py imports User.py, and User.py imports Portfolio.py, both modules try to load each other before finishing their own initialization.
Python stops and reports that a name cannot be imported because the module is only partially initialized.

This is known as a circular import.

## Why Circular Imports Are So Common in ORM Projects

Relational data models are naturally interdependent.
A User owns many Portfolios, and each Portfolio belongs to a User.
Both classes must refer to each other to describe that relationship.
SQLAlchemy is designed to handle such cycles conceptually, but Python’s import system is not.

The ORM is happy to refer to other models by string name and resolve those names later when all mappings are loaded.
Python’s import mechanism, by contrast, executes immediately and expects every reference to exist right now.
The tension between those two timelines is what leads to errors.

## How Python Imports Work: A Library Analogy

Python’s import mechanism behaves like a librarian retrieving books.

- Each .py file is a book.
- When one is imported, the librarian starts reading it line by line from top to bottom.
- If that book says “see also Book B,” the librarian begins reading Book B.
- But if Book B says “see also Book A,” the librarian discovers that Book A is still half-read and cannot yet be referenced.
- That interruption is a circular import.

The essential rule is that a module is executed once, from top to bottom, the first time it is imported.
While it is being executed, it is only partially built and unsafe to reference.

## What SQLAlchemy Does When Declaring a Relationship

Consider the following relationship declaration:

```python
user = relationship("User", back_populates="portfolios")
```

Here, SQLAlchemy stores the string "User" and postpones resolution until all models are imported and mapper configuration begins.
At that point it looks up the actual class object for "User".

However, type hints are evaluated differently.
The following line requires the actual User class to exist at import time:

```python
owner: Mapped[User] = mapped_column(ForeignKey("user.username"))
```

If User points to a module instead of a class, or if it is not yet defined, the ORM raises a MappedAnnotationError.
Understanding this distinction—relationships are resolved later, but type hints are evaluated immediately—is crucial.

## Breaking the Cycle: Deferring and Centralizing Imports

Circular dependencies are avoided by keeping each model file self-contained and letting the package initializer coordinate imports.
A common structure looks like this:

```python
domain/
    __init__.py
    Base.py
    User.py
    Portfolio.py
```

Each model file defines its own class and uses string names and type-checking imports to refer to other models.

## The Correct Pattern

```python
# User.py
from __future__ import annotations
from typing import TYPE_CHECKING
from sqlalchemy.orm import Mapped, mapped_column, relationship
from database import Base

if TYPE_CHECKING:
    from domain.Portfolio import Portfolio

class User(Base):
    __tablename__ = "user"

    username: Mapped[str] = mapped_column(primary_key=True)
    portfolios: Mapped[list["Portfolio"]] = relationship("Portfolio", back_populates="user")
    # notice that in the above type hint we surround Portfolio with quotes. That is so that Alchemy does not try to load the class before it is done importing.
```

```python
# Portfolio.py
from __future__ import annotations
from typing import TYPE_CHECKING
from sqlalchemy import ForeignKey
from sqlalchemy.orm import Mapped, mapped_column, relationship
from database import Base

if TYPE_CHECKING:
    from domain.User import User

class Portfolio(Base):
    __tablename__ = "portfolio"

    id: Mapped[int] = mapped_column(primary_key=True)
    owner_username: Mapped[str] = mapped_column(ForeignKey("user.username"))
    user: Mapped["User"] = relationship("User", back_populates="portfolios")
```

```python
# __init__.py
from .Base import Base
from .User import User
from .Portfolio import Portfolio
```

The __init__.py file acts as the central coordinator.
It imports all model modules once, ensuring SQLAlchemy registers every class and resolves relationship names correctly.

## Why TYPE_CHECKING and __future__ Are Essential

Two Python features make this pattern safe and expressive.

```python
from __future__ import annotations
```

This directive instructs Python to store type hints as unevaluated strings rather than live objects.
Because the names are not resolved immediately, annotations can safely reference classes that have not yet been defined.

```python
if TYPE_CHECKING:
```

This conditional ensures that imports inside the block are executed only by static analysis tools (such as type checkers or IDEs) and are skipped at runtime.
It provides full autocompletion and type validation without triggering import loops.

## A Useful Analogy: The Orchestra

Imagine the ORM package as an orchestra.
Each model file is a musician, and the package initializer (__init__.py) is the conductor.
The musicians know one another by name and understand when they are supposed to play, but they do not cue each other directly.
The conductor coordinates them in the correct sequence.
If the musicians tried to signal each other independently, the performance would break down.
Circular imports resemble that kind of chaos—communication without coordination.

## Summary of the Model Architecture

When building model packages:  

- Each model file defines its own class and avoids importing peer ORM classes at the top level.
- Relationships and foreign keys refer to other models by string names rather than direct references.
- The __future__ annotation import delays type evaluation until it is safe.
- TYPE_CHECKING imports provide static type information without affecting runtime behavior.
- The package’s __init__.py imports all model modules once to register them with SQLAlchemy in the proper order.

This structure allows the ORM to configure all relationships without encountering circular dependencies or partially initialized modules.

## Conclusion

Circular imports in SQLAlchemy are not random errors but natural consequences of how Python executes imports and how ORMs express relationships.
The solution lies in managing timing: letting Python load modules independently and allowing SQLAlchemy to resolve relationships later.
When imports are deferred, type hints are stored as strings, and models are orchestrated centrally, the ORM layer becomes stable, predictable, and easy to extend.

At that point, the model layer stops behaving like a fragile web of interdependent files and becomes a coherent, declarative representation of the database schema.
