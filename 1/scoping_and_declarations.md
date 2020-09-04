# Scoping and Declarations

## Variable Declaration

The first time a variable is referenced you must declare it's
type \<types\>:

```python
data: int128
```

In the above example we declare variable `data` with a type of `int128`.

Depending on the active scope, an initial value may or may not be
assigned:

> - For storage variables (declared in the module scope), an initial
>   value **cannot** be set
> - For memory variables (declared within a function), an initial
>   value **must** be set
> - For calldata variables (function input arguments), a default value
>   **may** be given

### Declaring Public Variables

Storage variables can be marked as `public` during declaration:

```python
data: public(int128)
```

The compiler automatically creates getter functions for all public
storage variables. For the example above below, the compiler will
generate a function called `data` that does not take any arguments and
returns an `int128`, the value of the state variable data.

For public arrays, you cna only retrieve a single element via the
generated getter. This mechanism exists to avoid high gas costs when
returning an entire array. The getter will accept an argument to specity
which element to return, for example `data(0)`.

### Tuple Assignment

You cannot directly declare tuple types. However, in certain cases you
can use literal tuples during assignment. For example, when a function
returns multiple values:

```python
@internal
def foo() -> (int128: int128):
    return 2, 3

@external
def bar():
    a: int128 = 0
    b: int128 = 0

    # the return value of `foo` is assigned using a tuple
    (a, b) = self.foo()

    # Can also skip the parenthesis
    a, b = self.foo()
```

## Scoping Rules

Vyper follows C99 scoping rules. Variables are visible from the point
right after their declaration until the end of the smallest block that
contains the declaration.

### Module Scope

Variables and other items declared outside of a code block (functions,
constants, event and struct definitions, ...), are visible even before
they were declared. This means you can use module-scoped items before
they are declared.

An exception to this rule is that you can only call functions that have
already been declared.

#### Accessing Module Scope from Functions

Values that are declared in the module scope of a contract, such as
storage variables and functions, are accessed via the `self` object:

```python
a: int128

@internal
def foo() -> int128
    return 42

@external
def foo() -> int128:
    b: int128 = self.foo()
    return self.a  + b
```

#### Name Shadowing

It is not permitted for a memory or calldata variable to shadow the name
of a storage variable. The following examples will not compile:

```python
a: int128

@external
def foo() -> int128:
    # memory variable cannot have the same name as a storage variable
    a: int128 = self.a
    return a
```

```python
a: int128

@external
def foo(a: int128) -> int128:
    # input argument cannot have the same name as a storage variable
    return a
```

### Function Scope

Variables that are declared within a function, or given as function
input arguments, are visible within the body of that function. For
example, the following contract is valid because each declaration of `a`
only exists within one function's body.

```python
@external
def foo(a: int128):
    pass

@external
def bar(a: uint256):
    pass

@external
def baz():
    a: bool = True
```

The following examples will not compile:

```python
@external
def foo(a: int128):
    # `a` has already been declared as an input argument
    a: int128 = 21
```

```python
@external
def foo(a: int128):
    a = 4

@external
def bar():
    # `a` has not been declared within this function
    a += 12
```

### Block Scopes

Logical blocks created by `for` and `if` statements have their own
scope. For example, the following contract is valid because `x` only
exists within the block scopes for each branch of the `if` statement:

```python
@external
def foo(a: bool) -> int128:
    if a:
        x: int128 = 3
    else:
        x: bool = False
```

In a `for` statement, the target variable exists within the scope of the
loop. For example, the following contract is valid because `i` is no
longer available upon exitting the loop:

```python
@external
def foo(a: bool) -> int128:
    for i in [1, 2, 3]:
        pass
    i: bool = False
```

The following contract fails to compile because `a` has not been
declared outside of the loop.

```python
@external
def foo(a: bool) -> int128:
    for i in [1, 2, 3]:
        a: int128 = i
    a += 3
```

<!-- tabs:start -->

#### ** Template **

[embedded-code](../assets/1/1.1-template-code.vy ':include :type=code embed-template')

<!-- tabs:end -->