# Mini-Compiler (Fortran-like Dialect)

A front-end compiler pipeline developed in **C**, **Flex**, and **Bison (Yacc)** for an educational Fortran-like imperative programming language. It performs lexical scanning, syntax analysis, comprehensive semantic verification, symbol table management, and intermediate code generation (3-address code / Quadruplets).

---

## 📌 Table of Contents
- [Overview](#-overview)
- [Architecture & Pipeline](#-architecture--pipeline)
- [Language Specification](#-language-specification)
- [Semantic Checks](#-semantic-checks)
- [Intermediate Code (Quadruplets)](#-intermediate-code-quadruplets)
- [Project Structure](#-project-structure)
- [Prerequisites](#-prerequisites)
- [Build & Run](#-build--run)
- [Sample Input & Output](#-sample-input--output)

---

## 🚀 Overview

This compiler translates a custom Fortran-style dialect into intermediate quadruplets. It implements:
1. **Lexical Analysis (Flex):** Token recognition, line/column tracking, identifier validation, and numeric range checks.
2. **Syntactic Analysis (Bison):** LALR(1) parsing for declarations, control structures, subroutines, and nested boolean/arithmetic expressions.
3. **Semantic Analysis:** Static checks for types, bounds, initializations, and signatures.
4. **Symbol Table:** Dynamically managed linked lists for identifiers, types, dimensions, scopes, and keywords.
5. **Code Generation:** Quadruplet generation with backpatching for conditional and loop jump targets.

---

## 🧩 Architecture & Pipeline

```text
 Source File (.txt / .f)
         │
         ▼
  [ Lexical Analyzer ] (Flex: lexical.l)
         │ Tokens + Location (Line, Column)
         ▼
  [ Syntactic Parser ] (Bison: syntaxique.y)
   ├───► [ Symbol Table Manager ] (ts_liste.c)
   ├───► [ Semantic Validator ]   (semantique.c)
   └───► [ Quadruplet Generator ] (quad.c)
         │
         ▼
 Execution Report:
  ├── Syntax & Semantic validation
  ├── Symbol Tables (Keywords & Identifiers)
  └── Quadruplet Table
```

---

## 📜 Language Specification

### 1. Data Types
- `INTEGER`: 16-bit integer values `[-32768, 32767]`.
- `REAL`: Floating point values `[-32768.32768, 32767.32767]`.
- `LOGICAL`: Boolean constants (`TRUE`, `FALSE`).
- `CHARACTER`: Fixed-size strings (e.g., `CHARACTER str *5` or single-char `CHARACTER c`).

### 2. Supported Structures
- **Programs & Subroutines:**
  - `PROGRAM <name> ... END`
  - `<type> ROUTINE <name>(<args>) ... <name> = <val>; ENDR`
- **Data Structures:**
  - 1D Arrays: `INTEGER TAB DIMENSION (100)`
  - 2D Matrices: `REAL MAT DIMENSION (10, 10)`
- **Control Flow:**
  - Conditional: `IF (<cond>) THEN ... ELSE ... ENDIF`
  - Loops: `DOWHILE (<cond>) ... ENDDO`
- **I/O & Memory:**
  - `READ(<var>)`
  - `WRITE(<expr>, "literal", ...)`
  - `EQUIVALENCE (<var1>, <var2>)`
  - Function invocations via `CALL`

### 3. Operators
- **Arithmetic:** `+`, `-`, `*`, `/`, unary minus (`-`)
- **Comparison:** `.LT.`, `.LE.`, `.GT.`, `.GE.`, `.EQ.`, `.NE.`
- **Logical:** `.AND.`, `.OR.`
- **Comments:** Single-line comments start with `%`

---

## 🔍 Semantic Checks

The semantic analyzer enforces safety rules during AST traversal:

| Check | Description |
|---|---|
| **Declaration Safety** | Detects double declarations and undeclared identifiers. |
| **Initialization** | Flags usage of uninitialized variables in expressions. |
| **Type Compatibility** | Restricts mixing incompatible types in arithmetic, assignments, and returns. |
| **Routine Consistency** | Validates number/types of arguments (`CALL`), ensures routine return variable matches routine name. |
| **Array/Matrix Bounds** | Enforces positive dimensions on declaration and bounds-checks constant/variable indices. |
| **Division by Zero** | Statically detects division by literal `0` and known zero-valued variables. |
| **Character Length** | Ensures assigned string literals do not exceed the declared variable length. |
| **Identifier Constraints** | Issues warnings for identifiers exceeding 10 characters. |

---

## ⚙️ Intermediate Code (Quadruplets)

The compiler generates standard 4-tuples: `(Operator, Operand 1, Operand 2, Result)`

- **Arithmetic & Logic:** `(+, A, B, temp1)`, `(LT, A, B, temp2)`
- **Assignments:** `(:=, source, vide, target)`
- **Conditionals & Branching:**
  - Unconditional Jump: `(BR, target, vide, vide)`
  - Branch if Zero: `(BZ, target, cond, vide)`
  - Branch if Non-Zero: `(BNZ, target, cond, vide)`
- **Arrays & Memory:** `(BOUNDS, 0, size, vide)`, `(ADEC, idf, vide, vide)`
- **Function/Subroutine Calls:** `(Parametre, P, vide, vide)`, `(Argument, A, vide, vide)`, `(CALL, funcName, nbArgs, vide)`

---

## 📁 Project Structure

```text
mini-compiler/
├── include/
│   ├── quad.h            # Quadruplet generator declarations
│   ├── semantique.h      # Semantic validation interfaces
│   ├── tools.h           # Utility stack & string operations
│   └── ts_liste.h        # Symbol table definitions
├── src/
│   ├── lexical.l         # Flex lexical scanner specification
│   ├── syntaxique.y      # Bison grammar & semantic actions
│   ├── semantique.c      # Semantic routines implementation
│   └── modules/
│       ├── quad.c        # Quadruplet generation & backpatching
│       ├── tools.c       # Dynamic stack & string formatters
│       └── ts_liste.c    # Linked-list Symbol Table implementation
└── build/
    ├── test_Quadruplets.txt  # Test suite for control flow & quads
    └── test_semantique.txt   # Test suite for semantic error handling
```

---

## 📦 Prerequisites

- **GCC** (MinGW recommended on Windows due to console coloring via `<windows.h>`)
- **Flex** (Fast Lexical Analyzer Generator)
- **Bison** (GNU Parser Generator)

---

## 🛠️ Build & Run

### 1. Compile with Flex and Bison

Run the following commands from the root of the project:

```bash
# Navigate to the build directory
cd mini-compiler/build

# 1. Generate parser C code and header
bison -d ../src/syntaxique.y

# 2. Generate scanner C code
flex ../src/lexical.l

# 3. Compile everything together
gcc syntaxique.tab.c lex.yy.c -o compiler.exe
```

> **Note for Linux / macOS Users:**  
> The file `src/modules/tools.c` includes `<windows.h>` for terminal colors. If building on Linux/macOS, remove `#include <windows.h>` and stub out `setConsoleColor()`, or replace it with ANSI escape codes (`\033[0;31m`).

### 2. Execute a Program

Pass your source code file as an argument:

```bash
./compiler.exe test_Quadruplets.txt
```

To run semantic verification tests:

```bash
./compiler.exe test_semantique.txt
```

---

## 🖥️ Sample Input & Output

### Input (`example.txt`)
```fortran
PROGRAM Demo
INTEGER A, B = 10;
REAL C;

A = 5;
IF (A .LT. B) THEN
    C = A * 2.5;
ELSE
    C = B;
ENDIF
END
```

### Compiler Output
```text
syntaxe correcte

/***************Liste des symboles IDF*************/
__________________________________________________________________________________
	| Nom_Entite |    Code_Entite |  Type_Entite |  Val_Entite/taille/nbArg | scope
__________________________________________________________________________________
	|          A |       VARIABLE |      INTEGER |                        - |  main
	|          B |       VARIABLE |      INTEGER |                       10 |  main
	|          C |       VARIABLE |         REAL |                        - |  main

*********************Les Quadruplets***********************

 0 - ( :=  ,  10  ,  vide  ,  B )
--------------------------------------------------------
 1 - ( :=  ,  5  ,  vide  ,  A )
--------------------------------------------------------
 2 - ( LT  ,  A  ,  B  ,  temp1 )
--------------------------------------------------------
 3 - ( BZ  ,  7  ,  temp1  ,  vide )
--------------------------------------------------------
 4 - ( *  ,  A  ,  2.5  ,  temp2 )
--------------------------------------------------------
 5 - ( :=  ,  temp2  ,  vide  ,  C )
--------------------------------------------------------
 6 - ( BR  ,  8  ,  vide  ,  vide )
--------------------------------------------------------
 7 - ( :=  ,  B  ,  vide  ,  C )
--------------------------------------------------------