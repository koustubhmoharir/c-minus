# c-minus
A toy language for learning programming. It provides a minimal set of abstractions. The idea is that the working of mainstream programming languages may be better understood by translating their syntax to this minimal set. The syntax is designed to be very easy to parse.

## Builtin data types
- `int`:  8 byte signed integer
- `float`: IEEE754 double precision floating point number
- `byte`: Unsigned byte

## Pointers
The `*` suffix makes a type a pointer type. For example, `int*` is the type of a pointer to an integer. Whitespace before `*` is not allowed, i.e, `int *` is not valid.

## Arrays
The `[size]` suffix makes a type an array type. `size` must be a positive integer constant. For example `int[5]` is the type of an array of 5 integers. Whitespace before `[` is not allowed, i.e, `int [5]` is not valid.

## Constant Values
- Integer constants are written in decimal without a decimal point. For example, `0`, `10`, `-20` are integer constants.
- Floating point constants are written in decimal with a decimal point or in scientific notation. For example, `0.0`, `-23.0`, `1e5` are floating point constants.
- Byte constants are written as decimal integers with a `b` suffix or as ASCII characters in single quotes with `\` as an escape character. For example, `0b`, `255b`, `'A'`, `'\n'` are byte constants.

## Identifiers
Identifiers must match the regular expression `[a-zA-Z][a-zA-Z0-9_]*`

## Named Constants
A named constant is written as `const identifier value` where `value` can be any constant value. For example
```
const one 1
```

## Variables
A variable is declared as `var identifier type`. `type` can be any built-in, array, pointer, or composite structured type. For example
```
var n int
var p int*
const five 5
var a int[five]
```

## Composite Structured Types
Types can be composed as `struct identifier [comma_separated_list_of_variable_declarations]`. The example below defines a type called `Point` that is composed of two `float`s with the names `x` and `y`.
```
struct Point [var x float, var y float]
```

## Assignments
An assignment to a variable is written as `addressable_value := expression`. `addressable_value` is any value that has a defined location / address. Most commonly, it is a variable. For example, the code below declares an integer variable `x` and assigns the value 1 to it.
```
var x int
x := 1
```
The code below defines a `Point` structure, a constant `SIZE`, a variable `pts` that is an array of `Point`s, sets the value of array variable to the coordinates of a square of length 1, and then modifies the y coordinate of the last point in the array to the y coordinate of the first point.
```
struct Point [var x float, var y float]
const SIZE 4
var pts Point[SIZE]
pts := [[0.0, 0.0], [1.0, 0.0], [1.0, 1.0], [0.0, 1.0]]
pts[3][y] := pts[0][y]
```

## Functions
A function is written as `fun identifier return_type(comma_separated_list_of_parameters)` followed by a block of statements enclosed in braces. For example, the code below defines a function that takes two pointers to integers and swaps the values they point to but does not return anything. Whitespace immediately before `(` is not allowed, neither in a function definition, nor in a call to a function.
```
fun swap_ints void(var x int*, var y int*)
{
    var t int
    t := x[0]
    x[0] := y[0]
    y[0] := t
}
```
Within a function whose return type is not void, `result` is an automatically defined variable of the return type. For example, the code below defines a function that just returns a constant value and does not take any arguments
```
fun constant_one int()
{
    result := 1
}
```

## Pointers to functions
The type of a function pointer is written as `return_type(comma_separated_list_of_parameter_types)*`. For example

```
var f1 void(int*, int*)*
f1 := _addr(swap_ints)
var f2 int()*
f2 := _addr(constant_one)
```
The _addr builtin function returns the address of a function or a variable.

## Control Flow

### if
An `if` statement is written as `if expression` followed by a block of statements enclosed in braces. The closing braces can be immediately followed by optional `else if expression` or `else` blocks. The code in the `if` block is executed if the expression evaluates to a non-zero value. For example, in the code below with `i` assumed to be an integer variable, `i` is set to `3 * i + 1` if it is odd and halved if it is even. `_rem_i`, `_add_i`, `_div_i` are builtin functions. There are no arithmetic or logical operators.
```
if _rem_i(i, 2)
{
    i := _add_i(_mult_i(i, 3), 1)
}
else
{
    i := _div_i(i, 2)
}
```

### while
A `while` statement is written as `while expression` followed by a block of statements enclosed in braces. For example, the code below counts the number of steps taken for the sequence formed by repeated operation of the `3 * i + 1 if odd, i / 2 if even` logic on an integer variable `i` to converge to 1. The `break` and `continue` statements can be used to break the loop or to jump to the beginning of the loop.

```
var n int
n := 0
while _neq_i(i, 1)
{
    if _eq_i(_rem_i(i, 2), 0)
    {
        i := _add_i(_mult_i(i, 3), 1)
    }
    else
    {
        i := _div_i(i, 2)
    }
    n := _add_i(n, 1)
}
```

### Labels and goto
A label marks a point that can be targeted with a goto statement within the same function that the label appears in. A label is written as `label identifier`
A goto statement is written as `goto label identifier`. For example the while loop above can be easily written with a goto as 

```
var n int
n := 0
label loop
if _neq_i(i, 1)
{
    if _eq_i(_rem_i(i, 2), 0)
    {
        i := _add_i(_mult_i(i, 3), 1)
    }
    else
    {
        i := _div_i(i, 2)
    }
    n := _add_i(n, 1)
    goto label loop
}
```
Thus the `while` statement is not necessary. It is a departure from the minimalism of the language. It is provided to distinguish normal looping from more complex control flow.

## Scope
Identifiers outside a function have global scope. Identifiers within a function have a scope limited to it and hide identifiers in the global scope. Braces for statement blocks as in `if`, `else`, `while` do not create a new scope, i.e, all identifiers within a function must be unique.

## Builtin functions

### Special functions
- The `_addr` function takes a single argument that can be a variable, a function, or any addressable value and returns a pointer of an appropriate type. The return type depends on static analysis.
- The `_alloc` function takes a `type` as its first argument and an integer `count` as its second argument. It allocates memory that can hold `count` items of `type`. It returns a pointer to the first item. Memory allocated by calling `_alloc` must be freed by calling `_free`.

### Integer functions
- `fun _add_i int(var a int, var b int)`: returns a + b
- `fun _sub_i int(var a int, var b int)`: returns a - b
- `fun _neg_i int(var a int)`: returns -a
- `fun _mult_i int(var a int, var b int)`: returns a * b
- `fun _div_i int(var a int, var b int)`: returns integer division a / b
- `fun _rem_i int(var a int, var b int)`: returns remainder when a is divided by b
- `fun _shl_i int(var a int, var b int)`: returns a << b
- `fun _shr_i int(var a int, var b int)`: returns a >> b
- `fun _flip_i int(var a int)`: returns ~a
- `fun _eq_i int(var a int, var b int)`: returns a == b
- `fun _neq_i int(var a int, var b int)`: returns a != b
- `fun _lt_i int(var a int, var b int)`: returns a < b
- `fun _lte_i int(var a int, var b int)`: returns a <= b
- `fun _gt_i int(var a int, var b int)`: returns a > b
- `fun _gte_i int(var a int, var b int)`: returns a >= b
- `fun _and_i int(var a int, var b int)`: returns non-zero if both a and b are non-zero
- `fun _or_i int(var a int, var b int)`: returns non-zero if either one of a and b are non-zero
- `fun _not_i int(var a int)`: returns non-zero if a is zero and zero if a is non-zero

Functions with a suffix of _u instead of _i treat their integer arguments as if they were unsigned integers.

### Floating point functions
Similar functions where applicable are available for floating point values with a `_f` suffix instead of the `_i` suffix.

## Significance of whitespace
Other than the cases mentioned above where whitespace is not allowed, whitespace is not significant.
For example
```
const
a
5 const b
6
// is identical to
const a 5
const b 6
// but should be avoided

struct Point
[
    var x float,
    var y float
]
// is identical to
struct Point [var x float, var y float]
// and may be preferred
```
