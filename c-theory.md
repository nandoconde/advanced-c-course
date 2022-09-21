# C Programming Theory

- [C Programming Theory](#c-programming-theory)
  - [Control flow](#control-flow)
    - [`goto`](#goto)
    - [Null statement](#null-statement)
    - [Comma operator](#comma-operator)
    - [Error control - `setjmp` and `longjmp`](#error-control---setjmp-and-longjmp)
  - [Variables](#variables)
    - [Bit fields](#bit-fields)
    - [Complex numbers](#complex-numbers)
    - [Type alias – `typedef`](#type-alias--typedef)
    - [Type union - `union`](#type-union---union)
    - [Storage-class specifiers](#storage-class-specifiers)
      - [`auto`](#auto)
      - [`extern`](#extern)
      - [`static`](#static)
      - [`register`](#register)
    - [Type qualifiers](#type-qualifiers)
      - [`const`](#const)
      - [`volatile`](#volatile)
      - [`restrict`](#restrict)
    - [Variable-length arrays](#variable-length-arrays)
    - [Flexible array members](#flexible-array-members)
  - [Functions](#functions)
    - [Variadic functions - `varargs`](#variadic-functions---varargs)
    - [Inlined functions - `inline`](#inlined-functions---inline)
  - [No-return functions - `_Noreturn`](#no-return-functions---_noreturn)
  - [Preprocessor directives](#preprocessor-directives)
    - [`#define`](#define)
    - [`#undef`](#undef)
  - [Memory access](#memory-access)
    - [Static](#static-1)
    - [Stack](#stack)
    - [Heap](#heap)
  - [Code organization](#code-organization)
    - [Source files](#source-files)
    - [Header files](#header-files)
    - [Compiler toolset - Makefile](#compiler-toolset---makefile)
  - [C Standards](#c-standards)
    - [C89/C90](#c89c90)
    - [C99](#c99)
    - [C11](#c11)

## Control flow

### `goto`

Jump directly from current execution point to label. It cannot jump outside the
current function. Avoid it at all costs, except for breaking directly out of 
deeply nested loops.

```c
// Some things
goto t;
// More things
t:
int i = 8;
// Etc
```

### Null statement

Literally, non-existent statement: just a semicolon. Used when all the logic
can be wrapped within a `for`/`while` condition block, and there is no need to 
provide a body. Also used to leave parts of the `for` conditions blank, such as
the initialization.

### Comma operator

It allows multiple evaluation within one line, returning the result of the 
leftmost expression. Used in one-liners to visually compress statements.

### Error control - `setjmp` and `longjmp`

C does not have embedded mechanisms for exception/error management. Instead, it
uses the library `setjmp.h` to control the execution and return to a previously
fixed point within the program. Not only it returns to that point, but it also
restores the memory stack as it was when that point was reached.

```c
#include <setjmp.h>
jmp_buf j;
int i = setjmp(j);
// Control if an error happens when coming back
// Execution happens in-between
// An error is raised, and execution must return to exception control
longjmp(j, -1);
```

## Variables

### Bit fields

Support for structs with fixed bitlength fields. The difference between bit 
fields and bit arrays is that the former fits within a system word, while the 
latter exceeds the usual word size.

They can be coded just as structs are, following each field by a colon and the
number of bits they occupy. This allows accessing each field by name and having
a more compact memory representation. However, they are limited in size to that
of a system word. They are more performant if their size is aligned to the 
memory of the system (16, 32 or 64 bits).

The following syntax is used to declare bit field structs. Nameless fields can 
be used as padding.

```c
struct packed_bits_struct_name{
  unsigned int :3;
  unsigned int f1:5;
  unsigned int t2:7;
  unsigned int :1;
};
```

### Complex numbers

Support for complex numbers comes from compiler or dedicated library 
`complex.h`.

They define functions and macros to work with them depending on the type of its
cartesian arguments:
- Real part: `crealf`, `creal`, `creall`
- Imaginary part: `cimagf`, `cimag`, `cimagl`
- Conjugate: `congf`, `conj`, `conjl`
- Arithmetic: `+`, `-`, `*`, `/`
- Unitary and binary bit arithmetic

Casting an imaginary number to real produces 0. Casting to `_Bool` results in:
- `0` if imaginary number was zero
- `1` otherwise

**Support through library:**
- `float complex`
- `double complex`
- `long double complex`
- `float imaginary`
- `double imaginary`
- `long double imaginary`

It also provides `I` as the imaginary unit (to build complex or imaginary 
literals) and more functions (`cpow`).

**Support through compiler:**
- `float _Complex`
- `double _Complex`
- `long double _Complex`
- `float _Imaginary`
- `double _Imaginary`
- `long double _Imaginary`

### Type alias – `typedef`

`typedef` aliases a known type to a specific keyword. Sometimes this is good 
to add readability of code, but mostly it is useful in case a type is 
system-dependent (`int`, `long`, etc.). In those cases, they can be aliased to
descriptive names such as `uint8`, or `int8`, etc. It is also used for 
complicated or long aliases in casting.

This can be accomplished also through the `#define` directive. However, since
`typedef` is handled by the compiler, it can be assigned also to derived data 
types (structs). This is commonly used because it removes the necessity of 
using the word `struct` when defining a struct.

The best practice is using `typedef` for system-dependent aliases and casting, 
and only for structs in case the style guide followed supports this practice.

### Type union - `union`

It defines a new type that may contain either one of its members. It is defined
through the types of its members and a name for each. Since it is similar to a 
type, it must be placed in header or separate files for inclusion.

```c
union Range{
  unsigned short u;
  signed short i;
};
```

Since it is defined as structs, the member of a union is accessed the same way 
as struct members. Through `.` dot access for objects or `->` access for 
pointers.

### Storage-class specifiers

When a variable is declared, it can be accompanied by special keywords that 
modify some aspects of it, such as its scope and its lifespan. There are four
specifiers for this:
- `auto`
- `extern`
- `static`
- `register`

The pattern to declare variables with them is:
```c
// st_cl_spec var_type var_name;
auto int t = 0;
```

When choosing among them, the most restrictive specifier that works must be 
chosen. Otherwise, the namespace will be unnecesarily polluted, leading to 
possible human errors.

#### `auto`

Specifies a *local* variable. Rarely used, since the C compilers automatically 
assume that any function declared within a function is an automatic variable.
Since C++ has repurposed the keyword `auto` for logic relating return types, 
**it is better not to use it at all**.

However, it can be used to make it clear to another programmer that the scope of
the variable is local, maybe even overriding another global variable of the same
name, or stating that it must not scape the local scope in any way.

In any case, it is bad programming practice.

#### `extern`

Specifies that the variable is defined elsewhere, not within the same block it 
is used. It is a *global* variable initialized with a legal value where it was
declared and ready to be used elsewhere. It extends the concept of global 
variable.

Functions in separate files can communicate through external variables. This 
variable will be visible to **all** program files. Functions can also be 
`extern`, which means that they will not have to be included as headers before 
using them; by default, all functions are `extern`. These variables can be 
declared as `extern` within a block of code, or simply declared in the global 
scope, outside of every function.

**Using them is considered not very good practice**, and it must be avoided 
unless it is unfeasible to accomplish the same otherwise.

#### `static`

This modifier can be used on local and global variables and functions, but its
behaviour differs between them.

When applied to local variables, the compiler will keep them in existence as 
long as the program runs, but they can only be accessed within the scope of the
original variable. This is particularly useful for local variables whose value
may depend on the last call to that function, and thus will avoid recalculation.
Initialization of these variables will only happen the first time theay are 
reached. After that, they will have the same value the last time that function 
was accessed. They must be initialized to a constant the first time. By default,
they are initialized to 0.

When applied to global variables, the compiler will restrict the scope of the 
variable to the current file.

When applied to functions, the compiler will restrict calling the function to 
the same file. Since all functions are `extern` by default, this really 
accomplishes limiting the scope of functions to be very local.

Static variables should not be declared within a structure, since the compiler
requires the memory allocated for a struct to be contiguous. However, a struct 
can be declared as static, making it thus possible to access the desired file
from anywhere inside the file.

#### `register`

It suggests the compiler to store a variable within the CPU registers. These 
registers are very fast to access, and thus variables that are heavily used 
throughout a function should be stored there. It is not always possible to do 
this, but the compiler will decide by itself.

The scope of a `register` variable is local (`auto`), and its maximum size is 
determined by the size of the CPU registers. Since they live outside normal 
memory, they cannot be accessed through pointers nor indirection operators can
be used on them.

### Type qualifiers

Keywords that provide the compiler with additional information about the 
intended use of the variable. It helps the compiler optimize the location of the
variable. The supported type qualifiers are:
- `const`
- `volatile`
- `restrict`

Type qualifiers are idempotent.

#### `const`

Intended for variables whose content will not change after initialization. The 
compiler may raise an error. It allows the compiler placing the constant into 
read-only memory slots, which avoids fragmentation and increases speed.

If a pointer is declared with the `const` keyword it can mean two things:
- `const int *pt` means that it points to constants of said type; the direction 
  itself may be changed, but the contents may not.
- `int * const pt` means that the pointer itself is constant.

The difference between `#define` and `const` is that the first is a preprocessor
directive and may be used by any file, while the second is a scope-controlled
qualifier.

#### `volatile`

It informs the compiler that the variable is going to change values at some 
point in the program. This prevents it from "caching". This helps in real-time 
programming, many threads or scarce resources, because prevents the compiler 
from storing the variable somewhere and access its value from there instead of
the real place, where it mght have chaged.

The meaning it has depends on where it is placed, as in `const`. 

It can be used paired with `const`, for example, for variables whose value is 
not changed by the program, but by another software that has access to that 
slot of memory.

#### `restrict`

Only for pointters, it hints the compiler that this is the only reference to a
variable thorughout the scope. This means that the variable it points to will 
only be referenced through it, and thus the compiler may avoid watching it from 
everywhere, and rather assume that as long as it has the source controlled, it 
will only be accessed in a controlled manner. It also means that if two 
restricted pointers are defined, they will always point to different directions.

`int * restrict pInt`

The syntax is only valid for variables! Without this keyword, the compiler must 
assume the worst case. With it, some optimizations might be triggered.

### Variable-length arrays

These are arrays **whose size is fixed** but unknown at compilation time. 
Although they are easy to declare, they are consciously unsupported by many C
compilers. The best option is try to code assumming they are not supported. 

The reason for the lack of support is that `malloc`-allocated arrays are easier
to check and safer.

### Flexible array members

Flexible array members are arrays as fields of a struct and whose size is not 
stated at source definition. They must obey some rules:
- They can be declared only as the last field of a struct
- Thus, there can only be one per struct
- They must no be the only field of the struct either
- A struct with a flexible array member cannot be a member of other struct
- A struct with a flexible array member cannot be statically initialized
  - It must be allocated dynamically
  - The size cannot be fixed at compile time

It is debatable whether they are a good practice.

## Functions

### Variadic functions - `varargs`

A variadic is a function of indefinite arity, i.e., with an undefined number
of arguments. This is used in mathematics also.

It needs `<stdarg.h>`. To support varargs, the function definition must have 
both:
- **Mandatory arguments**
  - at least one,
  - order matters
- **Optional arguments**

Common practice is using the first extra variable to tell the callee how many
arguments will there be. The access to the variable arguments is performed 
through the macros `va_list`, `va_start`, `va_arg`, `va_end` and `va_copy`.

Declaration of varargs is done through ellipsis in the function prototype.
Ellipsis must be the last argument, and must always have a first non variadic
argument. See documentation online on how to use them (cumbersome).

### Inlined functions - `inline`

Prepending `inline` before a function definition encourages the compiler to 
inline the function wherever it is called, i.e., to substitute the whole source
code into the caller in order to avoid calling overhead. 

- It is compulsory that the function is defined in the same file as the caller,
  or in a header file included by it.
- It does not *force* the compiler to perform the inlining, only encourages it.
- The inlining performance impact is negligible if the body of the inlined 
  function takes a lot of time or if it is too long.

## No-return functions - `_Noreturn`

Some functions never return to their callers for various reasons. If a function
with `_Noreturn` specifier is called, the compiler is promised that it will not
go back on the stack to the callee, and it may perform optimizations such as 
discarding the previous stack.

- Violating this promise is undefined behaviour
- Usually invoked through `noreturn` macro instead (it needs `stdnoreturn.h`)

## Preprocessor directives

Preprocessor directives begin with `#`, which may be preceded or followed by
spaces or tabs. They do not end with a semicolon.

Preprocessor directives must be placed in a single line and occupy the whole 
line by themselves. If a directive is visually too long it can be split through
`\` and continuing on the next line. The first parsing pass will combine this 
into a single line.

### `#define`

The `#define` directive defines symbolic or manifest constants in the source 
code of the program. They are not variables, just special words that are 
substituted before compilation. They are used mostly as constants that are 
defined at source code level (they will not change during the execution) in 
order to have a more flexible program, whose values are automatically updated
if the name value is changed. This also helps with system-related constants

The syntax does not include any equality name, simply the name followed by the 
value. By convention, all names defined by `#define` directive must be in 
capital letters with underscores as separators. It is good practice that the
value itself is a constant (must not be calculated).

Although they can appear anywhere, it is best practice to place them all at the
beginning of files, moreso at the entry points to the program or a subroutine. 
They can also be declared within an `include` file. This is specially 
appropriate, since it enables other programs to include this file in order to 
access them.

Although `NULL` is already defined in `stddef.h`, if it is not included it can
be defined through `#define NULL 0`.

### `#undef`

Removes a constant from definitions in the preprocessing stage. For example, 
to reuse `I` as something other than the imaginary unit if the `complex.h` 
library is included.

## Memory access

### Static

- Persists during the entire execution of the program
- Used to store things like global variables
- Created through the `static` clause

### Stack

**Characteristics**
- Region of the memory that stores temporary variables
- Stores variables created inside a function
- LIFO: last-in, first-out
  - Contiguous allocation
  - Reduces memory fragmentation
- Automatic management of space:
  - Every time a function declares a new variable it is pushed to the stack
  - Every time a function exits variables pushed to the stack are freed
  - Once a variable has been freed the memory is up for grabs
- Allocation
  - Stack space is limited
  - Stack overflow error if exceeded
  - A stack frame is allocated at each function call
- Freeing
  - Not possible to access the stack of a function from outside
- Easy to keep track of them
  - They are only available to the function
  - No management needed

**Use cases**
- Very fast access
- Local variables
- Small variables
- Static-size variables
- Worry-free allocation
- Reduced memory fragmentation

### Heap

**Characteristics**
- Hierarcal structure
  - Pool that can be used dynamically
- Global accessibility
- Slower to read/write than stack
- Non-automatic management of space:
  - Explicit allocation (`malloc`)
  - Explicit freeing (`free`)
- Allocation
  - No restrictions on size of the heap
  - No restrictions on size of the variable
    - Just the machine's own memory
- Freeing
  - Unfred memory may cause memory leaks

**Use cases**
- Allocating a large block of memory, such as
  - large arrays,
  - big structs
- Variables that will be accessed somewhere else
- Variables whose size changes dynamically
  - Arrays that grow or shrink as needed
- Reallocation of objects with same size (`realloc`)

## Code organization

### Source files

### Header files

In order to group together the implementations of several files into one module,
a single header file can be created that 'unifies' the complete funcitonality.
This is the file that will be included by others in order to access the module.

### Compiler toolset - Makefile

Projects are usually split into organized folders and small files. In order to 
keep track of all files that must be included in compilation and perform 
incremental compilations (avoiding unnecesary repetitions in the process), 
external utilities are used. The most common is `make` (also `ninja`).

`make` keeps track of all files in a project using a file called `makefile`, 
which contains the source code names and their relations. If called from the 
terminal, it performs the compilation of the the whole program but avoids 
unnecesary recompilation of files that have already been turned to object files
and whose source code has not been modified.

However, `make` has very complicated syntax, and almost no one uses it as-is. It
is usually generated automatically with another tool, such as `cmake` or 
`meson`, which are used to generate `make` files or `ninja` files for 
compilation.

Newer IDEs take care of everything, and this process is transparent to the user.

## C Standards

### C89/C90

First C standardized by ISO. Universally supported, it is safe to use all of its
features assuming the compiler will support them.
Features:
- New type qualifiers: `const` and `volatile`

### C99

Supported by most compilers now, introduces new features that improve coding
experience and ease the burden of some tasks. New features include:
- `stdbool` library for `true` and `false`
- `complex` library for `_Complex` and `_Imaginary`
- Designated initializers
- Restricted pointers
- Single-line comments
- "Inline function" compiler hints
- Variable-length arrays
- Macros with variable number of arguments
- Flexible array members
- Variable declarations allowed everywhere
- New type qualifier: `restrict`
- Idempotency of type qualifiers

### C11

Most modern standard with relatively wide adoption by compilers, advanced 
concepts introduced to the language:
- Multithreading support through `threads` library
  - Threads
  - Mutexes
  - Condition variables
  - If unavailable, use POSIX-compliant `pthread` library
- Safer standard libraries
  - Bound checking safety
  - Multithreading safety
- Better compliance with other standards
- Anonymous structs and unions
- Static assertions
- No-return functions

Optional features include:
- Variable-length arrays
  - If defined, `_STDC_NO_VLA = 1`
- Complex arithmetic
  - If not implemented, `__STDC_NO_COMPLEX_`





