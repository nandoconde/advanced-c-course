# C Programming Theory

## Variables

### Namespaces

#### Static variables

These are file-limited global variables. This means that they can be accessed 
everywhere throughout the file, but not outside of it. This is different from
**external variables**.

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

### C89

Universally supported, it is safe to use all of its features assuming the 
compiler will support them.

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





