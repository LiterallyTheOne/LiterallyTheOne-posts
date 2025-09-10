---
date: '2025-07-14T08:28:09+03:30'
draft: false
title: 'Decorator in Python'
tags: ["Python"]
image: "decorator_python.jpg"
---

# Decorator in Python

## Introduction

You might have seen `@something` before a
function in Python and ask yourself, what
does that mean? These `@something`s are called
`decorator`s. We use a `decorator` to modify a
function without changing it.

In this tutorial, we define a `decorator` using
a function and a class also with give an example
on stacking 3 `decorator`s.

## Decorator with function

```python

from typing import Callable


def check_triangle_validation(func: Callable[[float, float, float], float]) -> Callable[[float, float, float], float]:
    def wrapper(a: float, b: float, c: float) -> float:
        if (a + b <= c) or (b + c <= a) or (a + c <= b):
            raise Exception('Not a valid triangle')
        result = func(a, b, c)
        return result

    return wrapper


@check_triangle_validation
def calculate_triangle_area(a: float, b: float, c: float) -> float:
    return a + b + c


def main():
    a = 2
    b = 2
    c = 3
    result = calculate_triangle_area(a, b, c)
    print(result)


if __name__ == '__main__':
    main()
```

In this example, we have a function that calculates
the area of a triangle (`calculate_triangle_area`).
We want to make sure that the given triangle
is a valid triangle. To do so, we define
a `decorator` called `check_triangle_validation`
that gets a function as an argument and returns a `wrapper`.
Then we have an inner function that is going to run instead
of the `decorated` function that we call it `wrapper`.
in `wrapper`, we check if every vertex is less than
the sum of the other two. Then we execute the
`decorated` function and finally, we return the result.

If we run our function with values of `a=2,b=2,c=3`,
we are going to have `7` as a result. But if we run
our function with values of `a=1,b=1,c=3`, we are
getting an `Exception` that we defined in the `if` statement
in our `wrapper`.

## Decorator with class

```python
from typing import Callable


class CheckTriangleValidation:

    def __init__(self, func: Callable[[float, float, float], float]):
        self.func = func

    def __call__(self, a: float, b: float, c: float) -> float:
        if (a + b <= c) or (b + c <= a) or (a + c <= b):
            raise Exception('Not a valid triangle')
        result = self.func(a, b, c)
        return result


@CheckTriangleValidation
def calculate_triangle_area(a: float, b: float, c: float) -> float:
    return a + b + c


def main():
    a = 2
    b = 2
    c = 3
    result = calculate_triangle_area(a, b, c)
    print(result)


if __name__ == '__main__':
    main()
```

Defining a `decorator` using a class is pretty much the same
as the function. we get our `decorated` function as an argument
in `__init__` function. Then we should define a `__call__` function
that works exactly like our `wrapper`.

## Stacking decorators

```python

from typing import Callable


def change_meter_to_feet(func: Callable[[float, float, float], float]) -> Callable[[float, float, float], float]:
    def wrapper(a: float, b: float, c: float) -> float:
        result = func(a, b, c)
        return result * 3.28084

    return wrapper


def check_positive_vertices(func: Callable[[float, float, float], float]):
    def wrapper(a: float, b: float, c: float) -> float:
        if a <= 0 or b <= 0 or c <= 0:
            raise Exception('Vertices should be natural numbers')
        result = func(a, b, c)
        return result

    return wrapper


def check_valid_vertices(func: Callable[[float, float, float], float]):
    def wrapper(a: float, b: float, c: float) -> float:
        if (a + b <= c) or (b + c <= a) or (a + c <= b):
            raise Exception('Not a valid triangle')
        result = func(a, b, c)
        return result

    return wrapper


@check_positive_vertices
@check_valid_vertices
@change_meter_to_feet
def calculate_triangle_area(a: float, b: float, c: float) -> float:
    return a + b + c


def main():
    a = 2
    b = 2
    c = 3
    result = calculate_triangle_area(a, b, c)
    print(result)


if __name__ == '__main__':
    main()
```

We can stack up multiple `decorator`s. In this example, we
have three `decorators`. `check_valid_triangle` checks if
every vertex is less than the other two. `check_positive_vertices`
check if all vertices are positive. `change_meter_to_feet` changes
the calculated result from meter to feet.

If we run `calculate_trinalge_area` with values of
`a=2, b=2, c=3`, the result will be `22.96588`.

