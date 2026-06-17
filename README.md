What is Light?

Light is a newly conceived programming language positioned as a hybrid between the expressiveness of modern scripting languages and the structural robustness of compiled languages.
The ultimate goal of the project is to eliminate the visual noise typical of traditional programming, offering an ecosystem that is both fluid and rigorous.


Inspiration and Design Philosophy

The language fuses the best traits of two distinct worlds:

    The cleanliness of Python and Lua: It rejects the abuse of curly braces and redundant punctuation for control flow, preferring clean constructs and explicit textual delimiters (end) to close code blocks.

    The safety of strict typing: Unlike purely dynamic scripting languages, Light introduces an explicit type annotation system. This allows the developer—and the compiler—to know exactly what data flows through variables, parameters, and functions, drastically reducing runtime failures.



Key Language Features

    Optional Explicit Typing: Allows you to prototype rapidly in a "scripting" fashion, while offering the power to lock down production code by specifying native types for enhanced stability.

    Flexible Out-of-the-Box Data Structures: Built-in support for indexed sequences (lists) and highly versatile associative structures (tables/dictionaries) makes data manipulation intuitive and immediate.

    Simplified Object-Oriented Programming: Supports OOP through well-defined classes, featuring default field values, instance methods, and a dedicated initialization constructor.



Compiler Architecture (The Frontend)

The current infrastructure implements the foundational frontend of the compiler, split into three decoupled components operating in a strict pipeline:
1. The Lexical Analyzer (Lexer)

The Lexer acts as the entry point for raw source code. It consumes the raw text (a flat string of characters) and slices it into atomic, meaningful units called Tokens.
The Lexer identifies what constitutes a keyword, a number, a string, or a mathematical operator, while discarding comments and irrelevant whitespace.

2. The Abstract Syntax Tree (AST)

The AST is the internal, logical, and hierarchical representation of the program. Instead of treating code as a linear sequence of text, Light maps it as a structural tree of concepts.
For example, a loop is represented as a specialized "node" containing sub-trees for both its condition and its body.

3. The Syntactic Analyzer (Parser)

The Parser acts as the grammar enforcement officer of the language. Utilizing a recursive descent design,
it analyzes the token stream produced by the Lexer to verify it complies with Light’s grammatical rules.
If valid, the Parser models and instantiates the AST nodes; if it encounters a structural violation (such as an unclosed function),
it halts and reports the exact line of the failure.
