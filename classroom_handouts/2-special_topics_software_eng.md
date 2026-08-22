# Special Topics in Software Engineering

## Understanding Modularity in Python

**Modularity** is the practice of breaking a large program into smaller, self-contained pieces
that can be developed, tested, and reused independently. Each piece handles a specific
part of the program’s functionality.

- **Analogy:** Think of building with LEGO blocks. Instead of one giant block, you have
    many smaller blocks that you can arrange in different ways to create new things.
- In programming, these smaller blocks are **modules** and **functions**.

## Why Modularity Matters: Redundancy Without It

Without modularity, code quickly becomes **repetitive and hard to manage**. Imagine
writing a program that calculates the area of rectangles in different parts of your code:

```python
# Without modularity
length1 = 10
width1 = 5
area1 = length1 * width1
print("Area 1:", area1)

length2 = 20
width2 = 15
area2 = length2 * width2
print("Area 2:", area2)
```

Notice how the formula length * width is repeated. If the formula changes, you would need
to update it in every place it appears.

Now with modularity:

```python
def rectangle_area(length, width):
    return length * width

print("Area 1:", rectangle_area(10, 5))
print("Area 2:", rectangle_area(20, 15))
```

The code is shorter, clearer, and easier to maintain.

## Benefits of Modularity

When we say that modularity is “good practice,” what do we really mean? Let’s break it
down.

1. **Reusability**
    - Once you write a function or a module, you can use it in multiple programs
       without rewriting the code.
    - Example: A function that calculates compound interest could be reused in
       your **portfolio app** , a **loan calculator** , or a **simulation program**.
    - This saves time and reduces errors because you are relying on code that is
       already tested.
2. **Maintainability**
    - If you find a bug, you only need to fix it in one place.
    - Example: If your compound interest formula was wrong, without modularity
       you might have to fix it in **five files**. With modularity, you fix it **once in the**
       **module** , and every program that imports it automatically gets the correction.
3. **Clarity and Readability**
    - Breaking a program into modules makes the logic easier to follow. Each
       module handles a single responsibility.
    - Example: In a game, you might have one module for **graphics** , another for
       **sound** , and another for **rules**. Each module is small enough that you can
       understand it without being overwhelmed.
4. **Collaboration**
    - Modularity allows teams to work on different parts of the project
       simultaneously without stepping on each other’s toes.
    - Example: One student can work on the **database module** , another on the **UI**
       **module** , and another on the **math utilities**. As long as the interfaces
       between modules are clear, the team can work in parallel.
5. **Testing and Debugging**
    - Smaller, self-contained modules are easier to test individually.
    - Example: You can test your math_utils.py module independently before plugging it into the larger application.

**Key Idea:** Modularity keeps your code organized like a well-structured library — instead of one giant book with everything jumbled, you have neat shelves with clearly labeled sections.

## Modules and Packages

In Python, modularity is achieved using **modules** and **packages**.

- **Module:** A single .py file containing functions, classes, or variables.
- **Script:** A Python file meant to be executed directly (e.g., python myscript.py).
  - The difference is in intent:
    - A **script** is written to _do something_.
    - A **module** is written to _be reused by other code_.
- **Package:** A folder that contains multiple modules, plus a special file `__init__.py` that tells Python this is a package.

## Running Python as a Script vs as a Module

When you run a Python file, there are two main ways to do it:

### 1. As a script

`> python my_program.py`

1. Python executes the file line by line.
2. The file’s directory is added to sys.path, but its parent package is not considered.
3. This works fine for small standalone scripts.

### 2. As a module (with -m)

`> python -m my_package.my_module`

1. Python uses the import system to locate and execute the module.
2. It ensures that the parent package (my_package) is recognized.
3. The module is executed within a package-aware context.

### What does package-aware context mean?

When Python runs a file as a script, it treats that file as if it lives “alone.” Relative imports
like this:

```python
from. import helpers
```

will fail, because Python doesn’t know about the file’s parent package.

When you use -m, Python treats the file as part of its package structure. It knows that
my_module belongs to my_package, so relative imports work.

**Without -m:**

`> python my_package/my_module.py` → Might throw ImportError: attempted relative import with no known parent package.

**With -m:**

`> python -m my_package.my_module` → Works correctly, because Python sets up the import system with the package hierarchy
in mind.

### How does -m avoid import errors?

By using the import machinery:

- Python first looks in sys.path for the package.
- It loads the module with the correct package name.
- Relative imports are resolved correctly because Python knows where the module
    belongs in the project tree.

**Key Idea:** Running with -m is like telling Python: _“Don’t just run this file, run it as part of its
family of modules.”_ This avoids common headaches when your project grows beyond a
few scripts and becomes a package with many interconnected modules.

## Code Reuse with Imports

Once you write a module, you can reuse it in other programs with the **import statement**.

### Import Styles

#### 1. Basic import**

```python
import math_utils
math_utils.rectangle_area(10, 5)
```

#### 2. Import with alias**

```python
import math_utils as mu
mu.rectangle_area(10, 5)
```

#### 3. Import specific functions**

```python
from math_utils import rectangle_area
rectangle_area(10, 5)
```

#### 4. Import everything (not recommended)**

```python
from math_utils import *
```

This pollutes the namespace and can cause name conflicts.

## Where Python Looks for Modules

When you import something, Python has to find the module. It searches in this order:

1. **The current directory** – Python looks for a file with the same name.
2. **The standard library** – Built-in modules like math, os, sys.
3. **Third-party packages** – Installed via pip, stored in site-packages.
4. **Directories in sys.path** – A list of paths that Python checks.

## From Local Modularity to Community Packages: PyPI, pip, and Virtual

## Environments

## Extending Modularity to the Community

So far, we have looked at modularity from the perspective of a single developer: breaking
code into modules and packages to **reuse** within the same project. But modularity extends
beyond just one person — it scales to the entire programming community.

Python developers around the world share their modules as **packages** that can be
installed and reused in any project. This ecosystem saves time, reduces duplication, and
allows developers to build on top of each other’s work. Instead of reinventing the wheel,
you can install a package that someone else has already written and tested.

## PyPI and pip

### What is PyPI?

The **Python Package Index (PyPI)** is the official repository where developers publish
Python packages. Think of it as a giant “app store” for Python libraries.

- Packages like **NumPy** (for mathematics), **Flask** (for web development), and **Pandas**
    (for data analysis) all come from PyPI.
- Anyone can publish their own package to PyPI so others can use it.

### What is pip?

**pip** is the tool that lets you install and manage packages from PyPI.

- To install a package: `> pip install requests` → Downloads the requests package from PyPI and makes it available in your project.

- To uninstall a package: `> pip uninstall requests`

- To list installed packages: `> pip list`

- To search PyPI: `> pip search flask`

**Key Idea:** PyPI is the **library of packages** , and pip is the **tool to access and install them**.

## Virtual Environments

One challenge with using packages is **dependency management**. Different projects may
require different versions of the same package. For example:

- Project A requires Django 3.2.
- Project B requires Django 4.0.

If you install everything system-wide, these requirements will conflict.

### Solution: Virtual Environments

A **virtual environment** is an isolated workspace where each project keeps its own
dependencies separate from the system.

- You create a virtual environment: `> python -m venv venv`

- Activate it:
  - On macOS/Linux: source venv/bin/activate
  - On Windows: venv\Scripts\activate
- Once active, pip install only affects that environment, not the rest of your system.
- To deactivate: `> deactivate`

**Analogy:** Think of virtual environments as **separate kitchens**. Each kitchen has its own
fridge and ingredients. Two kitchens can cook the same dish with different recipes without
interfering with each other.

## The requirements.txt File

When working in a team, everyone needs the same versions of packages for the project to
work consistently. This is where the **requirements.txt file** comes in.

- After installing your packages, you can save them to a file: `pip freeze > requirements.txt`

This file lists all installed packages and their versions, for example:

```txt
Flask==2.2.2
requests==2.28.1
numpy==1.23.4
```

Another developer can then recreate your environment exactly by running: `pip install -r requirements.txt`

### Why is this important?

Without requirements.txt, one developer might be using Flask 2.2 while another uses Flask
3.0, leading to errors like _“function not found”_. The file guarantees everyone is working with
the same versions.

## Exception Handling in Python

### Why do we need exception handling?

Programs interact with the messy real world: files go missing, networks time out, users
enter invalid data, APIs throttle, divisions hit zero, JSON is malformed, DB connections
drop.
Without exception handling, your program **crashes** the first time something unexpected
happens. With exception handling, you can:

- **Fail gracefully** (show a helpful message instead of a stack trace).
- **Recover or retry** when possible.
- **Guarantee cleanup** (close files, release DB connections).
- **Isolate bugs** so one bad operation doesn’t take down the whole app.

### The try/except pattern

In Python it’s called **try/except** (other languages call it try/catch). Core shapes:

```python
try:
    risky_thing()
except SomeError as e:
    handle(e)
```

You’ll often also see:

```python
try:
    # risky operations
except SpecificError as e:
    # targeted handling
except (ErrorA, ErrorB) as e:
    # handle a group
except Exception as e:
    # last-resort safety net
else:
    # runs only if the try block had NO exception
finally:
    # runs ALWAYS (success or failure) - great for cleanup
```

**When to use else:** put the “happy path” that depends on successful try work in else.
**When to use finally:** closing files, releasing locks, finishing spans/metrics—things that
must happen regardless.

### Creating and using custom exceptions

Custom exceptions let you express **domain-specific problems** cleanly. They compose
well across layers (e.g., I/O layer → service layer → API layer).

```python
class PortfolioError(Exception):
    """Base class for portfolio-related errors."""

class HoldingsNotFound(PortfolioError):
    """Raised when holdings are missing for a requested account."""

class PricingServiceUnavailable(PortfolioError):
    """Raised when the pricing API cannot be reached."""
```

You can now **raise** and **catch** them in the right place:

```python
def fetch_holdings(account_id: str) -> list:
    holdings = query_db_for_holdings(account_id)
    if not holdings:
        raise HoldingsNotFound(f"No holdings for account {account_id}")
    return holdings


def price_portfolio(account_id: str) -> float:
    try:
        holdings = fetch_holdings(account_id)
        return call_pricing_api(holdings)
    except HoldingsNotFound as e:
        # user-level issue: show a friendly message
        print(e)
        return 0.0
    except TimeoutError as e:
        # infra issue: convert to domain exception and re-raise
        raise PricingServiceUnavailable("Pricing API timed out") from e
```

### Why a base exception?

By having PortfolioError as a base, higher layers can **handle all portfolio errors at once**
while still allowing fine-grained handling below:

```python
try:
    value = price_portfolio("ACC-123")
except PortfolioError as e:
    # single recovery path for any portfolio-related problem
    log_warning(e)
    value = 0.0
```

## Best practice (read these twice!)

1. **EAFP over LBYL** (Pythonic): “It’s Easier to Ask Forgiveness than Permission.”
    Don’t pre-check everything; just try it and handle the exception.
2. **Catch the most specific exception you can handle**. Avoid broad except
    Exception: unless it’s a top-level “safety net” with logging/metrics.
3. **Never use bare except:** (it also catches KeyboardInterrupt, SystemExit). If you need
    a catch-all, use except Exception as e:.
4. **Don’t swallow exceptions silently.** Log or surface enough detail for debugging.
    Consider logging.exception("msg") inside an except block to include a traceback.

5. **Use else for the success path** and **finally for cleanup** (closing files, releasing DB
    sessions).
6. **Prefer custom exceptions** for domain errors; keep a small, meaningful hierarchy.
7. **Don’t use exceptions for normal control flow** (e.g., to exit loops) unless it’s a truly
    exceptional path; it hurts readability and performance.
8. **Validate at the edges** (API inputs, CLI args) and raise clear, user-facing exceptions
    early.
9. **Wrap and re-raise with context** using raise NewError(...) from e so you keep the
    original cause.
10. **Unit-test failure paths** : assert that your code raises the right exception with
    pytest.raises(...).

## Understanding Typing in Python: From Weak Typing to Type Hints

### Introduction: Weak vs. Strong Typing

In programming, every piece of data has a **type**. Numbers are integers or floats, text is a
string, and lists are collections. The way a language enforces and checks these types is
called its **typing system**.

- **Strongly typed languages** (like Java or C#) enforce type rules strictly. If you declare
    a variable as an integer, you cannot later use it as a string without explicit
    conversion.
       o **Analogy:** Think of strong typing like boarding a plane with a ticket. If your
          ticket says “Gate 3,” you cannot use it at Gate 4.
- **Weakly typed languages** (like JavaScript, PHP, and Python) are more flexible. A
    variable’s type can change at runtime, and the language often does automatic
    conversions for you.
       o **Analogy:** Weak typing is like entering a festival where one wristband gives
          you access to multiple zones. The system doesn’t strictly enforce where you
          belong.

**Example:**

```python
x = 5       # x is an integer
x = 'five'  # now x is a string, Python allows this
```

In Java, this would be illegal unless you explicitly converted the type.

### Pros and Cons of Weak Typing in Python

#### Pros

1. **Flexibility:** Developers can write code quickly without worrying about declaring
    types in advance.
2. **Ease of use:** Ideal for beginners, since you don’t need to know formal type systems
    to get started.
3. **Rapid prototyping:** Great for testing ideas and building small projects quickly.

#### Cons

1. **Hidden bugs:** Type errors may only appear when the program runs, sometimes in
    production.
       a. Example: A function expecting a number might receive a string and crash
          unexpectedly.
2. **Lack of clarity:** Without explicit types, it can be harder for others (or your future
    self) to understand what kind of data is expected.
3. **Reduced tool support:** Without type information, editors and IDEs have less ability
    to suggest completions or catch mistakes early.

### Type Hinting: Adding Safety to Python

To balance flexibility with safety, Python introduced **type hints** (PEP 484). Type hints allow
developers to specify the expected types of variables, function arguments, and return
values — without changing how Python executes code.

- **Important:** Type hints are optional. Python will not enforce them at runtime.
    Instead, they act as **guidelines** for developers and external tools.
- **Analogy:** Adding type hints is like labeling boxes when moving. The contents don’t
    change, but the labels make it easier for you and your helpers to know what’s
    inside.

### Examples with the Typing Library

The typing module provides a standard way to write type hints in Python.

#### Primary Data Types

```python
def greet(name: str) -> str:
    return "Hello, " + name

def add(a: int, b: int) -> int:
    return a + b
```

Here, name must be a string, a and b must be integers, and the functions return a string
and integer respectively.

#### Collections

```python
from typing import List, Dict, Tuple

def average(numbers: List[int]) -> float:
    return sum(numbers) / len(numbers)

def phone_book() -> Dict[str, str]:
    return {"Alice": "555-1234", "Bob": "555-5678"}

def coordinates() -> Tuple[float, float]:
    return (10.5, 20.3)
```

These examples show how to annotate lists, dictionaries, and tuples.

#### Custom Data Types

```python
from typing import Optional

class Car:
    def __init__(self, brand: str, year: int):
        self.brand = brand
        self.year = year

def find_car(brand: str) -> Optional[Car]:
    if brand == "Toyota":
        return Car("Toyota", 2020)
    return None
```

Here, the function find_car returns either a Car object or None, so we use Optional[Car].

## Pylance: Catching Type Issues Early

While type hints improve code readability, they don’t prevent runtime errors by
themselves. That’s where **tools** like **Pylance** come in.

- **What is Pylance?**
    Pylance is a language server for Python used in Visual Studio Code. It analyzes your
    code and uses type hints to detect errors _before you run the program_.
- **What does it do?**
  - Flags mismatched types.
  - Provides intelligent auto-completion.
  - Warns when you call a function incorrectly (wrong number or type of arguments).
  - Improves navigation by showing type information on hover.

- **Example:**

```python
def add(a: int, b: int) -> int:
    return a + b

result = add("five", 10)
```

- Python will only complain when you try to run this code.
- With Pylance, you’ll see an error immediately in your editor: “Argument of type ‘str’ is not assignable to parameter of type ‘int’.”

This early detection prevents bugs from slipping into production and makes collaboration
easier by keeping expectations clear.
