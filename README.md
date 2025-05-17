# Robot-Control DSL

## What is this

A fully self-contained domain-specific language for commanding a simulated robot in a two-dimensional grid world. Defined entirely in a single JavaCC grammar (`Robot.jj`), it supports variable definitions, first-class functions, conditionals, loops, and direct invocation of robot-world APIs without generating an intermediate AST.

## How it works

1. **Lexical and Syntactic Specification**  
   - **JavaCC grammar**: uses `LOOKAHEAD=1` and `IGNORE_CASE=true` to implement an LL(1) parser.  
   - **Token declarations**: keywords (`defvar`, `defun`, `if`, `loop`, `repeat`, `not`), punctuation (`(`, `)`, `:`), identifiers (`NAME`), numeric literals (`NUMBER`), constants (`CONS`), and predicate forms (`facing?`, `blocked?`, etc.) are specified in the `TOKEN` section.  
   - **Skip rules**: whitespace and line breaks are discarded via `SKIP`.

2. **Integrated Parsing and Execution**  
   - **Recursive-descent productions**: each grammar production is annotated with Java code that either records tokens into a buffer (`codeInDefinition`) or issues side-effecting calls to `RobotWorldDec` when not in definition mode.  
   - **No separate AST**: function definitions accumulate their source text in `codeInDefinition` and, upon completion, store a `FunctionValue` (parameters + code string) in the current `Scope`. Invocation re-parses that stored code fragment via a fresh parser instance.

3. **Scope and Static Analysis**  
   - **`Stack<Scope>`**: every block, function definition, or conditional pushes a new `Scope` object.  
   - **Variable resolution**: `Scope` maps names to their string-encoded values (`"NUMBER 5"`, `"CONS dim"`, `"KEYWORD north"`, etc.). On each reference, the parser enforces that the identifier is defined in the current or any enclosing scope.  
   - **Function signatures**: stored as keys of the form `"fname arity"` mapping to `FunctionValue`. Redeclaration or arity mismatches produce `ParseException`.

4. **Control Structures without AST**  
   - **`if` / `loop` / `repeat`**: each evaluates a boolean condition by invoking `COND()`. If outside definition context, the parser toggles a `canExecute` flag and either executes or skips the nested productions.  
   - **Dynamic re-entry**: loops and recursion re-initialize parser state on stored code strings, enabling repeated execution without an explicit AST or bytecode.

5. **Command Semantics**  
   - **Movement and rotation**: high-level commands compute relative facing changes (front, back, left, right) and dispatch to `world.moveForward` and `world.turnRight`.  
   - **Interaction predicates**: methods like `facingNorth()`, `isBlocked()`, `canMove()` query the `RobotWorldDec` API for grid boundaries, obstacles, and resource availability.

All analysis (lexical, syntactic, and static-scope checks) and execution logic reside in `Robot.jj`, leveraging JavaCC’s generated parser framework for a compact yet powerful embedded DSL.
