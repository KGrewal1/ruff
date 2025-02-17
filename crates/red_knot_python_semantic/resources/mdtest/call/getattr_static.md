# `inspect.getattr_static`

`inspect.getattr_static` is a function that returns the value of an attribute of an object, without
invoking the descriptor protocol (for caveats, see the [official documentation]).

```py
import inspect

class Descriptor:
    def __get__(self, instance, owner) -> str:
        return 1

class C:
    normal: int = 1
    descriptor: Descriptor = Descriptor()

c = C()

reveal_type(c.normal)  # revealed: int
reveal_type(c.descriptor)  # revealed: str

reveal_type(inspect.getattr_static(c, "normal"))  # revealed: int
reveal_type(inspect.getattr_static(c, "descriptor"))  # revealed: Descriptor
```

For non-existent attributes, a default value can be provided:

```py
reveal_type(inspect.getattr_static(C, "normal", "default-arg"))  # revealed: int
reveal_type(inspect.getattr_static(C, "non_existent", "default-arg"))  # revealed: Literal["default-arg"]
```

When a non-existent attribute is accessed without a default value, the runtime raises an
`AttributeError`. We could emit a diagnostic for this case, but that is currently not supported:

```py
# TODO: we could emit a diagnostic here
reveal_type(inspect.getattr_static(C, "non_existent"))  # revealed: Never
```

Attributes of objects of all kind can be accessed:

```py
import sys

reveal_type(inspect.getattr_static(sys, "platform"))  # revealed: LiteralString
reveal_type(inspect.getattr_static(inspect, "getattr_static"))  # revealed: Literal[getattr_static]

reveal_type(inspect.getattr_static(1, "real"))  # revealed: Literal[1]
```

[official documentation]: https://docs.python.org/3/library/inspect.html#inspect.getattr_static
