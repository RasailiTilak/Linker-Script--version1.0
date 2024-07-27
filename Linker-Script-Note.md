

# Linker Script

**Version: 1.0**

- If the linker cannot recognize the format of the object file, it will assume that it is a Linker Script.
- This script will augment the original script.
  - Default one or the one manually provided using the `-T`.

```plaintext
SECTIONS
MEMORY
```

These can appear only once.

- `--oformat srec` & `--oformat=srec` are equivalent.
- `-M`, `--print-map` : Print map.
- `-s`, `--strip-all` : Omit all symbol information.
- `-S`, `--strip-debug` : Omit debugger symbol information.
- `-t`, `--trace` : Print the names of the input files as `ld` processes them.
- `-static` : Do not link against shared objects.
- `--cref` : Print a symbol table.
- `--defsym symbol=expression` : Defines a symbol.
- `-Map mapfile` : Map mapfile.

---

## The Linker Command Language

1. Input files
2. File formats
3. Output file layout
4. Address of sections
5. Placement of common blocks

If not `{object | archive | linker script}`, then print "error".

---

## Linker Script

- Collection of statements
  - Setting a particular option
  - Select group input files
  - Name output file
- Two statements have fundamental & pervasive impact on the linking process:
  - `SECTIONS`
  - `MEMORY`
  - (optional)

```plaintext
/* Comment. Equivalent to whitespace. */
```

---

## Expressions

- All expressions evaluated as integers and are of `long` or `unsigned long` type.
  - All constants are integers.
  - All of the C arithmetic operators are provided.
  - You may reference, define, and create global variables.
  - You may call special purpose built-in functions.

---

## Integers

- Octal, Decimal, Hex, negative numbers.
  - Octal: `0`
  - Hex: `0x`
  - Example: `--as-octal = 0o123;`
- Suffixes like:
  - k = 10^3
  - M = 10^6
  - may be used to scale a constant.

---

## Symbol Names

- Unless quoted, symbol names start with a:
  - Letter
  - Underscore
  - Point
- May include any letters, underscores, digits, points, and hyphens.
- Unquoted symbol names **must not** conflict with any keywords.

```plaintext
" SECTION " = 9;   // Valid
SECTION = 9;      // Invalid
```

```plaintext
" with a space " = " also with a space " + 16;
```

---

## Location Counter

- Called the location counter.
- Always contains the current output location.
- It always appears in the `SECTIONS` command.
- Appears anywhere on ordinary symbol allowed in an expression.
- Assignments have a side effect:
  - This moves the location counter.
  - Can be used to create holes in the output section.
- Should never be moved backwards.

---

## Operators

- Support all standard C operators with the standard bindings and precedence levels.

---

## Evaluation

- Calculates an expression when absolutely necessary.
  - "Lazy Evaluation" for expressions.
  - Needs the start address, and the lengths of memory regions, in order to do any linking of all.
  - These values are computed as soon as the command file is read.
  - Values like "symbol values" are not known until after storage allocation.

---

## Assignment: Defining Symbols

```plaintext
Symbol = expression;
Symbol &= expression;
Symbol += expression;
Symbol -= expression;
Symbol *= expression;
Symbol /= expression;
```

- Two things distinguish assignments from other operators in `ld` expressions:
  - Assignment may only be used at the root of an expression.

```plaintext
'a = b + 3;' // is allowed but
'a += 3' // is not [error]
```

- You may place a trailing semicolon (";") at the end of an assignment statement.
- Assignment statements may appear:
  1. As commands in their own right in an `ld` script; or
  2. As independent statements within a `SECTIONS` command; or
  3. As part of the contents of section definition in a `SECTIONS` command.
     - Define a symbol with an absolute address.
     - Define a symbol whose address is relative to a particular section.

---

## Absolute vs Relocatable Type

- When a linker expression is evaluated and assigned to a variable, it is given either an absolute or a relocatable type.
  - Absolute:
    - The value is fixed offset from the base of the section.
  - Relocatable:
    - The value is same as it will be in the output file.

- Type of expression is controlled by:
  - Position in the script file.
  - Relocatable:
    - Fixed offset from the section, relative to base of section.
  - Absolute:
    - Anywhere else, it is created as an absolute value. Even when assigned within the section using `ABSOLUTE()` function.

```plaintext
PROVIDE (Symbol = expression)
```

---

## Arithmetic Functions

- `ABSOLUTE (expr)`
  - Returns absolute, non-relocatable value of expression `expr`.
- `ADDR (section)`
  - Returns absolute address of named section.
- `LOADADDR (section)`
  - Returns absolute load address of the named section.
- `ALIGN (expr)`
  - Returns the result of the current location counter (`.`) aligned to the next `expr` boundary.
    - `expr` must be an expression whose value is a power of 2.
- `DEFINED (symbol)`
  - Returns 1 if the symbol is in the linker global symbol table and defined, 0 otherwise.
  - Example:

    ```plaintext
    begin = DEFINED (begin) ? begin : . ;
    ```

- `NEXT (expr)`
  - Returns the next unallocated address that is a multiple of `expr`.
  - Closely related to `ALIGN (expr)` command.
  - Unless you use the `MEMORY` command to define discontinuous memory for the output file, the two functions are equivalent.
- `SIZEOF (section)`
  - Returns size in bytes of the named section.
- `SIZEOF_HEADERS`
- `max (expr1, expr2)`
- `min (expr1, expr2)`

---

## Semicolons

- Required in the following places:
  1. Assignments
  2. PHDRS

- In all other places they can be used for aesthetics but are otherwise ignored.

---

## Memory Layout

- Contain at most one use of `MEMORY` command.
- You can define as many blocks of memory within it as you wish.
- The syntax is:

```plaintext
MEMORY
{
  name (attr) : ORIGIN = origin, LENGTH = len
}
```

- Attributes that suggest the property of the segment/block:
  - Used internally by the linker to refer to the region.
  - Start address
  - Length
  - `"ALIRWX"`
    - Allocated Section
    - Initialized Section
    - Read Only
    - Read/Write Section
    - Executable Code
    - ! (inverts sense of any attribute)

---

## Specifying Output Sections

- `SECTIONS` command controls exactly where input sections are placed into output sections, their order in the output file, and to which output section they are allocated.
- You may use at most one `SECTIONS` command in a script file. There can be many statements within the `SECTIONS` command.
  - Statements within the `SECTIONS` command can do one of three things:
    1. Define the entry point.
    2. Assign a value to a symbol.
    3. Describe the placement of a named output section and which input sections go into it.

- Can also be done outside of the `SECTIONS` command.

```plaintext
SECTIONS
{
  section_name
  {
    contents
  }
  ...
  name_of_output_section
}
```

- Special section name `/DISCARD/` may be used to discard input sections.
- Linker will not create output sections which do not have any contents.

- `SECTIONS` command specifies the following:
  1. Location
  2. Alignment
  3. Contents
  4. Fill pattern
  5. Target memory region

```plaintext
.section_name : { *(.foo) }
```

- `section_name` in the output file will only be created if `.foo` section exists in at least one input file.

---

## Section Placement

- You can specify the contents of an output section by listing:
  1. Particular input files
  2. Particular file sections
  3. Combination of the two.

- Can place arbitrary data in the section.
  - Define symbols relative to the beginning of the section.

- The contents may include any of the following kinds of statements:
  - Can include as many of these as you like in a single definition, separated from one another by whitespace.

1. `filename`

```plaintext
.data { aFile.o bFile.o cFile.o }
```

2. `filename (section)`

```plaintext
filename (section, section, ...)
filename (section section, ...)
filename (section section ...)
``

`

- Can specify one or more sections from an input file - use parentheses.
- If more than one are specified, then separate them in the parentheses by a whitespace.

3. `* (section)`

```plaintext
* (section section)
* (section, section, ...)
```

- Use `*` to refer to all files.
- If a file is already referred to by its name - `*` refers to all remaining files.

4. `filename (common)`
  - Uninitialized data from given file.

```plaintext
* (common)
```

- Uninitialized data from all input files.

---

- In any place where you use a filename, you may also use a wildcard pattern.
  - Handled like they are by the Unix shell.

```plaintext
*      matches any number of characters.
?      matches single character.
[chars]  matches single instance of any of the chars.
-      matches characters in the range.
```

- The linker does not search directories to expand wildcards.

---

