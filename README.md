# JASON Compiler

A front-end compiler for JASON, a small imperative teaching language: a hand-written lexical
analyser and a recursive-descent parser that produces a parse tree, with a WinForms UI for
inspecting both stages.

## Overview

This project implements the two front-end phases of a compiler without a parser generator. Given a
JASON source file it produces a token stream, reports lexical and syntax errors with the line
numbers that caused them, and builds a parse tree for programs that are well-formed.

Writing the scanner and parser by hand — rather than generating them from a grammar — was the point
of the exercise: it forces you to deal with maximal-munch tokenizing, operator precedence, and
error recovery explicitly instead of delegating them to a tool.

Built as the Compiler Design coursework at Ain Shams University.

## Architecture

```
Source file (.txt)
  → ScannerPhase          tokenizes; builds the symbol table; records line numbers
  → Parser                recursive descent over the token stream; builds Node tree
  → Errors.Error_List     lexical + syntax errors, collected rather than thrown
  → WinForms UI           token table (Form1) and parse tree (Form2)
```

| File | Responsibility |
|---|---|
| `Scanner.cs` | Token classes, symbol table, tokenizer, line tracking |
| `Parser.cs` | Recursive-descent parser, one method per grammar production |
| `Errors.cs` | Shared error accumulator |
| `Form1.cs` / `Form2.cs` | Token output and parse-tree display |
| `JasonTestCase.txt` | Sample JASON program used to exercise the compiler |

## Language features parsed

The grammar is covered by roughly 40 mutually-recursive production methods in `Parser.cs`:

- **Declarations** — `integer` / `real` / `string` / `char`, single and comma-separated
- **Assignment** — `set x = expr;`
- **Control flow** — `if` / `elseif` / `else` / `endif`, and `repeat … until`
- **I/O** — `read`, `write`, `write endl`
- **Functions** — declarations, parameter lists, bodies, and `return`
- **Conditions** — `and` / `or`, relational operators (`<`, `>`, `<=`, `>=`, `=`, `<>`)
- **Expressions** — arithmetic with correct precedence, via the
  `expression → term → factor` production chain

## Error handling

Errors are accumulated into `Errors.Error_List` rather than aborting on the first failure, so a
single run reports every problem it can find. Each token carries its `lineNumber` from the scanner
through to the parser, which is what makes the reported diagnostics point at real source lines.

## Tech Stack

C# · .NET Framework · Windows Forms · Visual Studio

## Running Locally

Requires Visual Studio with .NET Framework support (Windows only — the UI is WinForms).

```
1. Open JASON_Compiler.sln in Visual Studio
2. Build the solution (Ctrl+Shift+B)
3. Run (F5)
4. Load JasonTestCase.txt from the UI to see the token stream and parse tree
```

## Engineering Decisions

**Recursive descent over a parser generator.** The grammar was written LL(1)-friendly so it could be
parsed by hand. This makes the parser directly readable — each production is a method with the same
name — at the cost of not handling left recursion, which the grammar avoids by design (see the
`expressionDash` / `conditionTermDash` continuation productions).

**Errors collected, not thrown.** A compiler that stops at the first error is frustrating to use.
Accumulating into a shared list means one compile pass surfaces everything.

**Symbol table built in the scanner.** Identifiers are registered during tokenizing so the parser
works against classified tokens rather than raw strings.

## Limitations

This is a front end only. There is no semantic analysis (no type checking or scope resolution), no
intermediate representation, and no code generation — the pipeline stops at the parse tree.

## Future Improvements

- Semantic analysis: type checking and scope/symbol resolution
- Panic-mode error recovery so the parser can resynchronize and continue after a syntax error
- Automated tests over a corpus of valid and invalid JASON programs
- Port the UI off WinForms so the compiler can run cross-platform
