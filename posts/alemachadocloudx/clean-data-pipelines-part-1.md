---
title: Introduction to Clean Code in Python
published: false
description: Clean Code practices applied to data engineering in Python
tags: 'python, softwareengineering, datangineering, cleancode'
cover_image: ./assets/cover1.png
series: Clean Data Pipelines in Python
id: 1782367
---

## Introduction

During my recent assignment, I extensively worked with Python and Airflow, in pretty complex solution for dynamically creating data pipelines based on configurations defined in a web application, also developed using Python and Flask. Sooner than later, we encountered challenges and began falling behind in delivery times and quality, so, in order to improve this, I investigated a lot about Clean Code techniques.
Recognizing that many data engineering teams consist of individuals from diverse backgrounds, not necessarily from software engineering, I had the idea to share the insights I got the hard way in this article series.

### Technical debt, the silent evil

Our primary challenge revolved around the accumulation of technical debt. But what is technical debt? It refers to problems arising as a result of compromises or poor decisions made during coding. In essence, *technical debt occurs when a decision is made during coding that makes the code harder to change in the future than it is at present.*

### Why Clean Code?

Our initial motivation for embracing Clean Code was to mitigate technical debt, but it offers additional benefits. In a large and intricate software environment, **code serves not only as instructions for a machine but also as a means for developers to understand and communicate the system's logic.** Since you depend on other developers, including your future self, to read and maintain the code, its quality becomes vital. *Quality code enhances estimation accuracy, facilitating a steady and predictable delivery of features.*

Consider this analogy: driving towards a destination. If the car works well and the road conditions are favorable, estimating your arrival time should be reasonably accurate. On the contrary, if the road is in poor condition, filled with obstacles and your car is struggling, estimating the arrival time becomes challenging. In this analogy, the road represents the code. To consistently contribute new features at a steady and predictable pace, the code must be readable, maintainable, and easy to change. If it isn't, every attempt to add a new feature would necessitate stopping to refactor and address accumulated technical debt.

### When Clean Code Might Not Be Appropriate

It's essential to recognize that Clean Code may not be the best choice for every project, particularly **when the code will not be revisited after its initial production.** Consider the following scenarios:

- Single-use scripts.
- Proof of concepts.
- Prototypes that are certain to be discarded.
- Legacy code slated for deprecation.

## Code Formatting and Tools

Even though, code formatting and layout are not the only concern of Clean Code, it is really important to have these addressed to reach a quality codebase.
For this purpose, we will introduce the use of coding standard [PEP-8](https://www.python.org/dev/peps/pep-0008/). It is not the only standard, but it is the most well-known and accepted one, so you should try to stick to it.
Adhering to a formatting standard will improve the readibility of the codebase by giving it consistency, which will give developers the ability to identify quickly any error or bug. It also improves the searchability of specific parts, for example, PEP-8 differentiates how to use `=` operator, when used for assignments, it should have spaces around it, but when used for keyword argument, it must not. This way you could find easier a variable assignment and a keyword argument, even if the name is the same:

```python
# Variable assignment
name = "Peter"
```

```python
# Keyword argument
def foo(name=None)
    pass
```

In short, implementing a coding style guide using a standard will improve the overall quality of the code, making contributions (adding new features or bug fixing) easier for developers.

## Documentation Best Practices

A good quality code is a well documented code, where it is clear stated what the code is doing, and in some cases why it is doing it in a specific way, but be mindful that it should not be describing how the code works. That should be expressed clearly in the code itself.
To do a good documentation, Python two main tools, `docstrings` and `annotations`. We can also use code comments in the specific case that we, for example, are doing something using a technical nuance, or an external constraint, such as an argument for a external function, but these comments should be concise and as few as possible.

### Docstrings

Docstrings are a string literal embedded in the code to document the logic of that part. There are meant to include an explanation of what a function, module or class does and how developers should use it, expected inputs and outputs. This last part is important since Python is dynamically typed.

The way to add docstrings to our codebase is simply as adding a string literal at the beginning of the function, module or class, enclosed with `"""`.
Here we can see an example from built-in module `collections`:

```python
def namedtuple(typename, field_names, *, rename=False, defaults=None, module=None):
    """Returns a new subclass of tuple with named fields.

    >>> Point = namedtuple('Point', ['x', 'y'])
    >>> Point.__doc__                   # docstring for the new class
    'Point(x, y)'
    >>> p = Point(11, y=22)             # instantiate with positional args or keywords
    >>> p[0] + p[1]                     # indexable like a plain tuple
    33
    >>> x, y = p                        # unpack like a regular tuple
    >>> x, y
    (11, 22)
    >>> p.x + p.y                       # fields also accessible by name
    33
    >>> d = p._asdict()                 # convert to a dictionary
    >>> d['x']
    11
    >>> Point(**d)                      # convert from a dictionary
    Point(x=11, y=22)
    >>> p._replace(x=100)               # _replace() is like str.replace() but targets named fields
    Point(x=100, y=22)

    """
    ...
```

As we can see, at the beginning of the docstring, it explains what the function is doing, and then it gives several examples of the different ways the function can be called and what type of arguments to use.

There are tools to create automated documentation from docstrings, such as `sphinx`, that has the module `autodoc`. It will create a page containing the documentation for your project that you can later publish.

### Annotations

Annotations are another great tool for documenting our code in Python, it basically allows us to hint in the code what types to expect for arguments, variables and return values in functions. This is really useful, specially for Python being dynamic typed, it reduces the possibility of errors.
It also enables the use of tools to automatic type checking, such as `mypy`.
Let's see this example where you want to calculate the average grades of students:

```python
def print_average_grade(students):
    for student in students:
        average_grade = sum(student['grades']) / len(student['grades'])
        print(f"Average grade for {student['name']}: {average_grade}")
```

In this case, you should know before hand that students is a `list` of `dict` that must contain the keys `grades` and `name`, otherwise you will get an error when trying to use the function.

We can express the same function better using annotations and docstrings:

```python
from typing import Dict, List


def print_average_grade(students: List[Dict[str, Any]]) -> None:
    """
    Calculates and prints the average grade for each student.
    """
    for student in students:
        average_grade = sum(student['grades']) / len(student['grades'])
        print(f"Average grade for {student['name']}: {average_grade}")
```

Now we can have a better understanding of what values the function needs to work properly and what to expect from it.

This could be improved even further, by using `dataclass` decorator:

```python
from typing import Dict, List
from dataclasses import dataclass

@dataclass
class Student:
    name: str
    grades: List[float]

def print_average_grade(students: List[Student]) -> None:
    """
    Calculates and prints the average grade for each student.
    """
    for student in students:
        average_grade = sum(student.grades / len(student.grades)
        print(f"Average grade for {student.name}: {average_grade}")
```

Annotations are not only valid for built-in datatypes, but also for your own classes. In this last example, the code is more readable, without the need to view the function definition in order to use it.

### Tooling

There are some basic tools that lets you automate many of these code quality checks, allowing developers to set a baseline of code layout, format and, if needed, type checking, so each developer of the team can run these checks in their own environment and in continuos integration builds.

#### Automatic formatting

As stated before, you should have a coding style defined in your project and adhere to it, usually it is recommended to enforce it using an automatic formatting tool. In this case I will recommend [`black`](https://github.com/psf/black). This tool will allow you and the team to rely on it to keep the code layout and formatting consistent in the codebase, giving more consistency, readability, and more important, reducing the git diffs when adding new changes, allowing the code reviews to be more effective and having only the most significant changes on it.
The tool is pretty straightforward, because it does not allow much configurations, it can be seen as a rigid and opinionated tool, but once you start using it, formatting is more transparent and not an issue at discussion, so the developers can be focused on more important matters, even when working on different projects.
It is really easy to use, once installed, you can run it using the following command:

```bash
black file.py
```

Let's see how it works, using the following PEP-8 compliant function:

```python
import pandas as pd

def filter_csv(
    file_name
    ):
    """
    Filters a CSV file to exclude test rows. Returns a DataFrame.
    """
    with open(file_name, 'r') as file:
        df = pd.read_csv(file)

    filtered_df = df[df['name'].str.contains('TEST')]

    return filtered_df
```

And this is the result given by the tool:

```python
import pandas as pd


def filter_csv(file_name):
    """
    Filters a CSV file to exclude test rows. Returns a DataFrame.
    """
    with open(file_name, "r") as file:
        df = pd.read_csv(file)

    filtered_df = df[df["name"].str.contains("TEST")]

    return filtered_df
```

We can see a few differences, changing the arguments in a one-liner, adding an additional vertical space between the import and the function definition and changing single quotes with double quotes.

#### Static type checking

Python is dinamycally typed, which means that types are assigned at runtime, but you can use annotations to hint other developers and tools about what types to expect.
There are many tools for static type checking, one of the most used is [`mypy`](https://github.com/python/mypy). You will need to install it and add it as a dependency of your project on the setup file. Once you install it, it will analyze all the files in the project, checking for type inconsistencies. It is really useful for detecting bugs early in the development.
**It is important to note that python still is a dynamic typed language, and that does not changes with the use of `mypy`, so these checks are not enforced.**

Once installed, you can run the static type check running the following command:

```bash
mypy file.py
```

It will report any type inconsistency in the analyzed file, here is an example of that.

```python
def say_hello_to(name: str) -> None:
    print("Hello " + name)

say_hello_to("Peter") # No type inconsistency
say_hello_to(123)     # Argument 1 to "say_hello_to" has incompatible type "int"; expected "str"
```

##### Generic types

 Working with static type checks in Python can be confusing at first, specially if you come from a duck-type style background. You must be really careful to not type hint using generic types, such as `Any` or `Iterable`, because it will silently ignore really obvious errors, missing the point of running the checks. For example:

```python
def say_hello_to(name):
    print("Hello " + name.split() + "!")

say_hello_to("Peter") # No error!
```

In this case, since we haven't hinted the type for argument `name`, it is interpreted as `Any` type, and since `name.split()` is undefined, it will pass the check, always make sure to type hint it.

```python
def say_hello_to(name: str) -> None:
    print("Hello " + name.split() + "!")

say_hello_to("Peter") # error: Unsupported operand types for + ("str" and "list[str]")  [operator]
```

Along the same lines, let's say you have the following function:

```python
from typing import Iterable

def say_hello_to(names: Iterable[str]) -> None:
    for name in names:
        print("Hello " + name + "!")

say_hello_to("Peter") # No error!
```

Even if no errors are reported, this is clearly a mistake, since it will iterate over each character of the string, but since both `list` and `str` are `Iterable`, no error is returned. Instead use more specific types:

```python
from typing import List

def say_hello_to(names: List[str]) -> None:
    for name in names:
        print("Hello " + name + "!")

say_hello_to("Peter") # error: Argument 1 to "say_hello_to" has incompatible type "str"; expected "list[str]"  [arg-type]
```

#### Linters

Besides formatting and type checking, there are some tools that allow you to check a wider range of validations, beyond PEP-8 standard, such as errors in the imports, code smells and in some cases, it can make suggestion on how to refactor the code.
These static code analyzer tools are called **linters**. In this case, I will recommend `pylint`, that is one of the most well-known one, but there are a few more.
`Pylint` is highly configurable and, as said before, it goes beyond the formatting and layout, and all this rules can be parameterized in a configuration file, so the best approach will be to **create a coding standard and enforce it in the configuration file, having it available in the repository**.
Once installed, the tool will run the checks and report the validation result only by running the command `pylint`.

#### Automatic checks

A good and recommended way to automate the execution of these tools is using Makefiles. This file allow you to configure the commands to run in the project, making them easily available for other developers to use and also to be executed in you CI build.
Here is an example of a Makefile that contains the tools we introduced before:

```makefile
.PHONY: format
format:
    black *.py
.PHONY: lint
lint:
    pylint
.PHONY: typehint
typehint:
    mypy
.PHONY: check
check: format lint typehint
```

In this example, we defined commands to run each tool on its own, and the command `check` to run them one after the other.
The command to run, both in local development machine or in CI build would be as follows:

```bash
make check
```

## Conclusion

Clean Code is not only about code formatting, layout and documentation, but this is the baseline expected in a quality codebase, in this part of the series, we have introduced to you a series of practices and tools to start a project with the right foot, setting coding style standard, automatic checks and good documentation practices. In the next part of the series we are going to go deeper on Python idioms and design principles.
