# An Introduction to Programming and Programming Paradigms

Programming is the process of giving computer instructions so that it can perform tasks
and solve problems. Unlike humans, computers cannot infer or guess what we mean; they
only do exactly what we tell them. That is why programming is all about breaking down a
problem into steps that a machine can follow.

But programmers don’t all think about problems in the same way. Over time, different
**programming paradigms** have emerged — each one representing a particular philosophy
or style of writing instructions. A paradigm is like a lens through which you approach
problem-solving with code.

## Imperative Programming: Step by Step

The **imperative paradigm** focuses on telling the computer _how_ to do something. You write
out each step in the process, and the program executes them in order. The program’s state
— the values stored in memory — changes as the instructions run.

- **Analogy:** Following a cooking recipe step by step.
- **Example:** To add numbers 1 through 5, you would tell the computer to create a
    total, loop through the numbers, and add each one.

Imperative programming is common in languages like Python, Java, and C, and it’s often
the first paradigm students encounter because it mirrors our own logical step-by-step
reasoning.

## Declarative Programming: Focus on the Goal

The **declarative paradigm** takes a different approach. Instead of spelling out each step,
you describe _what result_ you want, and the system figures out how to achieve it.

- **Analogy:** Ordering food at a restaurant. You simply say, “I’d like pasta,” and the
    chef handles the cooking details.
- **Example:** In a database query language like SQL, you can simply declare:

```SQL
SELECT SUM(value)
FROM numbers
WHERE value BETWEEN 1 AND 5;
```

Here, you didn’t explain loops or additions — you just declared the end goal.

Declarative programming is especially powerful when the “how” is complicated or
variable, but the “what” is clear.

## Object-Oriented Programming (OOP): Organizing Around Things

Another influential paradigm is **object-oriented programming (OOP)**. Instead of focusing
on steps or goals, OOP organizes programs around **objects**. An object bundles together
**data** (the information it holds) and **behavior** (what it can do).

This is useful because it mirrors the real world. A Car object, for example, might have data
like color or model, and methods like drive() or brake(). By combining data and actions,
OOP makes it easier to model complex systems in a structured, reusable way.

## Plato’s Theory of Forms and OOP

Philosophy may seem far removed from programming, but it offers a useful lens for
understanding Object-Oriented Programming (OOP). Plato’s **Theory of Forms** suggests
that everything in the physical world is an imperfect copy of an ideal “Form.” For instance,
we may encounter many different chairs — wooden chairs, plastic chairs, big chairs, small
chairs — but Plato would argue that they are all reflections of the perfect “Form of a chair.”

Each physical chair has **properties** (such as color, size, and material) and **behaviors** (such
as being able to be sat on or moved). The Form is the abstract idea of what a chair is, while
each physical chair is an **instance** of that idea.

In OOP, we adopt a similar way of thinking:

- A **class** is like Plato’s Form — an abstract definition that describes what an object
    should be.
- An **object** is like a particular chair — a concrete instance with its own specific
    properties and behaviors.

When we build software, we are creating a **model of the world**. Just as a board game
simplifies real-world activities (e.g., Monopoly models property trading, chess models

strategic battle), software uses classes and objects to represent real or abstract entities.
This allows us to write programs that feel intuitive because they mirror the way we already
think about the world.

## Classes and Objects

In programming, a **class** is a blueprint. It describes what data (called _attributes_ ) and what
actions (called _methods_ ) an object of that class will have. For example, a class might say
that every Car has a color and a model, and it can drive() or stop().

An **object** is an actual instance created from that blueprint. If the class is the recipe, then
the object is the cake you bake by following it. Each object is independent: you might have
a red Toyota and a blue Honda, both built from the same Car class.

The process of creating an object from a class is called **instantiation**. To return to our
earlier analogy of board games: a game designer might sketch the rules and layout of a
game. That design is like a **class**. When someone cuts out the pieces, prints the board, and
plays, they’ve created **objects** that follow the design.

## A Demo: Building a Car Class

Let’s make this concrete with an example in Python. Suppose we want to represent cars in
our program.

### Step 1: Define a Class

```python
class Car:
    def __init__(self, color, model):
        self.color = color
        self.model = model

    def drive(self):
        print(f"The {self.color} {self.model} is driving.")

    def stop(self):
        print(f"The {self.color} {self.model} has stopped")
```

Here we’ve created a Car class. Inside it, we have:

- A **constructor** method called **init**. This special method runs whenever we
    create a new object, and it sets up the object’s initial attributes.

- The keyword **self** , which represents the individual object being created. This allows
    each object to keep track of its own attributes.

### Step 2: Instantiate Objects

```python
car1 = Car("red", "Toyota")
car2 = Car("blue", "Honda")

car1.drive() # the red Toyota is driving
car2.stop() # the blue Honda has stopped.
```

Here, car1 and car2 are **objects**. Both come from the same class, but each has its own
unique values. This shows how one blueprint can produce many different instances.

## The Constructor

A **constructor** is a special method in a class that runs automatically whenever a new
object is created. Its main job is to set up the initial state of the object by assigning values
to its attributes. In Python, the constructor is always named **init**, and it uses the
keyword self to refer to the object being built. For example, if you create a Car class with a
constructor that takes color and model as inputs, then every time you make a new Car
object, the constructor ensures those values are stored inside the object. Without a
constructor, every object would start “empty” and would need to be configured manually
after creation. By defining a constructor, we make sure every object is born ready with the
information it needs.

## Self

In Python, the keyword **self** is used inside a class to refer to the specific object that is being
created or acted upon. When you call a method on an object, Python automatically passes
that object itself as the first argument to the method, and by convention we call that
parameter self. This allows each object to keep track of its own data. For example, if you
have two Car objects — one red and one blue — the self.color attribute makes sure that
when the red car calls drive(), it refers to its own color, not the blue car’s. Without self,
Python would have no way of knowing which object’s data we are trying to access or
modify.

## Extending the Concept: Beyond Physical Objects

OOP is not limited to modeling physical things like cars, houses, or chairs. We can also
model **ideas and processes**.

For example, imagine building a **Trip Planner** app. You might create a Trip class:

- Its **attributes** could be destination, start_date, end_date, and budget.
- Its **methods** might include add_activity(), calculate_total_cost(), and
    show_itinerary().

Even though a “trip” is not a physical object in the same sense as a car, it is still something
that can be represented as an object in code. This shows the flexibility of OOP: anything we
can describe with data and behaviors can be modeled.

## The Four Core Principles of Object-Oriented Programming

Object-Oriented Programming (OOP) is built on four fundamental principles:
**Encapsulation, Abstraction, Inheritance, and Polymorphism.** These principles provide
structure and flexibility, allowing programmers to build software that is modular, reusable,
and easy to maintain. Let’s explore each principle in turn.

## Encapsulation

At its heart, **encapsulation** means that everything about an object should be bundled
together and protected from the outside world. The data and the methods that operate on
that data live inside the object.

Encapsulation also means that **access to data should be controlled**. Not every part of the
program should have permission to directly change an object’s internal state. Instead,
access should be granted only through specific methods that are designed for that
purpose.

- **Analogy:** Think of a **car’s gas tank**. You cannot pour gas directly into the fuel
    injector or engine — you must use the gas cap and filler. The tank is encapsulated
    and insulated, and the car gives you a safe method for interacting with it.

In Python, encapsulation is typically implemented using naming conventions:

- Attributes or methods with a single underscore (e.g., _speed) are treated as
    **protected**.
- Attributes or methods with a double underscore (e.g., __engine_temp) are treated
    as **private**.

```python
class Car:
    def __init__(self):
        self.__fuel = 0 # private attribute

    def add_fuel(self, liters):
        self.__fuel += liters

    def drive(self):
        if self.__fuel > 0:
            print("The car is driving.")
            self.__fuel -= 1
        else:
            print("Out of fuel!")
```

Here, the __fuel attribute cannot be accessed directly from outside the class. The only way
to add or consume fuel is through the add_fuel() or drive() methods.

## Abstraction

**Abstraction** is about **hiding unnecessary details** and showing only the essential features.
You don’t need to know exactly how something works internally in order to use it.

- **Analogy:** When you drive a car, you don’t need to understand the internal
    mechanics of the engine. All you need are the controls: the steering wheel, pedals,
    and dashboard.

In programming, abstraction is achieved by exposing a **public interface** (methods
available to all users of the class) while keeping the implementation details hidden.

```python
class Engine:
    def start(self):
        self.__ignite()
        print("Engine started.")
    
    def __ignite(self):     # private helper metho
        print("Ignition sequence initiated")
```

Here, the user can call start() without worrying about how the ignition works. The __ignite()
method is hidden (private) and is only used internally. This way, the interface stays clean,
and the complexity is abstracted away.

## Inheritance

**Inheritance** allows you to create a **hierarchy of classes** , where a more specific class can
build upon a more general one.

- **Superclasses** (also called base classes) define the common structure and
    behavior.
- **Subclasses** inherit from the superclass and can add their own specific attributes
    and methods.
- **Analogy:** Think of **vehicles**. A Vehicle class might define general properties like
    speed and methods like drive(). A Car class can then inherit from Vehicle and add
    car-specific features like air conditioning or trunk capacity.

```python
class Vehicle:
    def __init__(self, brand):
        delf.brand = brand

    def drive(self):
        print("The {self.brand} is moving.")

class Car(Vehicle):     # Car inherits from Vehicle
    def __init__(self, brand, doors):
        super().__init__(brand)     # call vehicle constructor
        self.doors = doors

    def honk(self):
        print("Beep beep!")

my_car = Car("Toyota", 4)
my_car.drive()  # Inherited from Vehicle
my_car.honk()   # Defined in Car
```

Inheritance allows us to reuse code, reduce duplication, and create logical hierarchies.

## Polymorphism

The word **polymorphism** comes from Greek, meaning “many forms.” In OOP,
polymorphism means we can treat objects of different classes **in the same way** , as long as
they share a common interface.

- **Analogy:** Imagine a function that tells any **vehicle** to drive. It doesn’t matter
    whether the object is a Car, a Bike, or a Truck — as long as each has a drive()
    method, the function works.

```python
class Bike(Vehicle):
    def drive(self):
        print(f"The {self.brand} bike pedals forward.")

vehicles = [Car("Honda", 4), Bike("Trek")]

for v in vehicles:
    v.drive() # Each object responds in it's own way.
```

Here, both Car and Bike objects can be passed to the loop, and each responds with its own
version of drive(). This makes code more flexible and powerful, because we can reason
about objects abstractly rather than worrying about their specific types.
