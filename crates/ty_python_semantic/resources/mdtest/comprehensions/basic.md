# Comprehensions

## Basic comprehensions

```py
# revealed: int
[reveal_type(x) for x in range(3)]

class Row:
    def __next__(self) -> range:
        return range(3)

class Table:
    def __iter__(self) -> Row:
        return Row()

# revealed: tuple[int, range]
[reveal_type((cell, row)) for row in Table() for cell in row]

# revealed: int
{reveal_type(x): 0 for x in range(3)}

# revealed: int
{0: reveal_type(x) for x in range(3)}
```

## Nested comprehension

```py
# revealed: tuple[int, int]
[[reveal_type((x, y)) for x in range(3)] for y in range(3)]
```

## Comprehension referencing outer comprehension

```py
class Row:
    def __next__(self) -> range:
        return range(3)

class Table:
    def __iter__(self) -> Row:
        return Row()

# revealed: tuple[int, range]
[[reveal_type((cell, row)) for cell in row] for row in Table()]
```

## Comprehension with unbound iterable

Iterating over an unbound iterable yields `Unknown`:

```py
# error: [unresolved-reference] "Name `x` used when not defined"
# revealed: Unknown
[reveal_type(z) for z in x]

# error: [not-iterable] "Object of type `int` is not iterable"
# revealed: tuple[int, Unknown]
[reveal_type((x, z)) for x in range(3) for z in x]
```

## Starred expressions

Starred expressions must be iterable

```py
class NotIterable: ...

# This is fine:
x = [*range(3)]

# error: [not-iterable] "Object of type `NotIterable` is not iterable"
y = [*NotIterable()]
```

## Async comprehensions

### Basic

```py
class AsyncIterator:
    async def __anext__(self) -> int:
        return 42

class AsyncIterable:
    def __aiter__(self) -> AsyncIterator:
        return AsyncIterator()

async def _():
    # revealed: int
    [reveal_type(x) async for x in AsyncIterable()]
```

### Invalid async comprehension

This tests that we understand that `async` comprehensions do *not* work according to the synchronous
iteration protocol

```py
async def _():
    # error: [not-iterable] "Object of type `range` is not async-iterable"
    # revealed: Unknown
    [reveal_type(x) async for x in range(3)]
```

## Comprehension expression types

The type of the comprehension expression itself should reflect the inferred element type:

```py
from typing import TypedDict

# revealed: list[int]
reveal_type([x for x in range(10)])

# revealed: set[int]
reveal_type({x for x in range(10)})

# revealed: dict[int, str]
reveal_type({x: str(x) for x in range(10)})

# revealed: list[tuple[int, Unknown | str]]
reveal_type([(x, y) for x in range(5) for y in ["a", "b", "c"]])

squares: list[int | None] = [x**2 for x in range(10)]
reveal_type(squares)  # revealed: list[int | None]
```

Inference for list comprehensions takes the type context into account:

```py
reveal_type([x for x in [1, 2, 3]])  # revealed: list[Unknown | int]

xs: list[int] = [x for x in [1, 2, 3]]
reveal_type(xs)  # revealed: list[int]

ys: dict[int, str] = {x: str(x) for x in [1, 2, 3]}
reveal_type(ys)  # revealed: dict[int, str]

table = [[(x, y) for x in range(3)] for y in range(3)]
reveal_type(table)  # revealed: list[list[tuple[int, int]]]

# TODO: no error here
# error: [invalid-assignment]
table_with_content: list[list[tuple[int, int, str | None]]] = [[(x, y, None) for x in range(3)] for y in range(3)]
reveal_type(table_with_content)  # revealed: list[list[tuple[int, int, str | None]]]

class Person(TypedDict):
    name: str

persons: list[Person] = [{"name": n} for n in ["Alice", "Bob"]]
reveal_type(persons)  # revealed: list[Person]

# TODO: This should be an error
invalid: list[Person] = [{"misspelled": n} for n in ["Alice", "Bob"]]
```
