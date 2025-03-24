---
title: An ANTLR 2.0 Tutorial
aliases:
    - /articles/antlrtut/index.html
    - /articles/antlrtut

categories:
- Articles

date: "2000-03-04"

description: Learning to use ANTLR v2.

tags:
- java
- antlr
- parsing
- language
- dsl

---
A tutorial on ANTLR 2.x

<!--more-->


### _Note: This is the ANTLR 2.x Tutorial. If you're looking for an ANTLR 3.x Tutorial, please see my new [ANTLR 3.x Video Tutorial](../20091221-antlr3tut)_

Introduction
------------

### (the part you'll never read...)

_Note: I've had several requests over the years to make this a single page. Well, here it is. Enjoy!_

I need to warn you up front that I can be a bit long-winded, so bear with me while I try to describe a bit about what ANTLR is, how you can write parsers and compilers using it, what type of code it will produce, the color of the moon on an October evening and why "42" is such an important number (or so says Douglas Adams.)

Ok. Call me nuts. I just sick of all the whining about how hard it is to learn ANTLR. (So the whining is valid -- it's a _bit_ hard to learn. _I_ only had the cat to whine to.) I remembered learning PCCTS (the old C/C++ version of ANTLR) a few years ago using the "Advanced Tutorial" but things have changed significantly since then. Back then, I used PCCTS for a compiler course. I loved it and have loved it ever since. I figured, "Hey, I've got a simple compiler written in PCCTS, so why not revise it and write it as a tutorial?" So here it is.

I want this tutorial to really help people, so I'm asking for input. Email me at  [scott@javadude.com](mailto:scott@javadude.com) with any suggestions or comments you may have, and I just might do some more work on this.

A Simple Language
-----------------

The compiler course I took at Johns Hopkins featured a small language called "XL." I'm not sure who came up with this language, but the course was taught by Dr. John Moore, and he presented XL to us. I wanted to at least give him some credit for getting me really interested in compilers and such. If anyone knows a reason why I shouldn't be presenting this language, please let me know and I'll remove this from the net. I'll be presenting the language almost verbatim from Dr. Moore's writeup of the language.

Note that in this tutorial I will not be implementing the entire XL language. I will only be implementing the features I selected when writing the compiler for the class. Eventually I will try to implement all the features in this document, if I have lots of extra time between reading to the kids and changing their diapers. I will try to mention when features are not implemented.

XL is a small but complete programming language based primarily on Ada and, to a lesser extent on C, C++ and Pascal. XL was designed to be suitable for use as a project language in a course on compiler design and construction. Its features illustrate many of the techniques and problems associated with language translation.

* * *

### General

XL is case sensitive. Upper-case letters and lower-case letters are considered to be distinct in all tokens, including reserved words.

White space (space character, tab character and end-of-line) serves to separate tokens; otherwise, it is ignored. No token can extend past end-of-line. Spaces may not appear inside any token except character and string literals.

A comment begins with two forward slashes and extends to end of line, as in C++.

* * *

### Identifiers

Identifiers start with a letter and contain letters and digits. An identifier must fit on a single line and its first 100 characters are significant.

* * *

### Reserved Words

The following keywords are reserved in XL:

keyword  | keyword | keyword  | keyword
---------|---------|----------|------------
 and     | array   | Boolean  | begin     
 Char    | constant| else     | elsif     
 and     | exit    | function | if        
 Integer | loop    | mod      | not       
 of      | or      | out      | procedure 
 program | record  | return   | String    
 then    | type    | var      | when      
 while   |         |          |           


* * *

### Literals

An integer literal consists of a sequence of one or more digits

A character literal is a single character enclosed by a pair of apostrophes (sometimes called "single quotes".) Examples include `'A'`, `'x'`, and `'''`. A character literal is distinct from a string literal of length one.

A string literal is a sequence of zero or more printable characters enclosed by a pair of quotation marks ("double quotes.") The double quote character itself can be represented in a string as two adjacent double quotes. In XL, string literals are only used as arguments to the predefined I/O procedure "put" (see below)

* * *

### Other Tokens (delimiters and operators)

operators  | note
---------- | -----------
: ; , . ( ) \[ \] | one character
\+ - \* / < = > ' | one character
.. := /= >= <=    | two characters
(end-of-file)         | actual EOF, not words

* * *

### Standard (Predefined) Scalar Types

The type `Boolean` is treated as a predefined enumeration type with elements `FALSE` and `TRUE`

The type `Integer` is a predefined type as in Pascal and FORTRAN (equivalent to type `int` in C)

The type `Char` is a predefined type which is equivalent to Pascal's `CHAR`. The predecessor and successor functions are invoked as attributes `Char'pred` and `Char'succ`. Thus, `Char'pred('D')` is 'C' and `Char'succ('A')` is 'B'. (At this point, these attributes are not implemented. I may add them later.)

* * *

### Type Generators

An enumeration type is defined by listing the identifiers which are the actual values of the type. As with type `Char`, the predecessor and successor functions of an enumeration type `T` as invoked as attributes `T'pred` and `T'succ`. For example:

	type CardSuit = <CLUB, DIAMOND, HEART, SPADE>;       

Then `CardSuit'pred(HEART)` is DIAMOND and `CardSuit'succ(HEART)` is SPADE.

An array type (one dimensional only) is defined by giving a range of indices and the component type. Only integer indices are allowed in XL. For example

	type Table = array[1..10] of Integer;       

A record type is defined by listing the individual component names (fields) and their types. For example:

	type Date =
		record
			day : Integer;
			month: Integer;
			year: Integer;
		end record;


* * *

### Named Constants

Named constants are introduced by declarations of the form

	constant ID, ID, . . ., ID: typeName := expression;       

`typeName` must be an identifier representing a scalar type (`Integer`, `Boolean`, `Char` or a user-defined enumeration type.) The phrase "`:=` expression" is required. For example:

	constant maxIndex : Integer := 100;       

* * *

### Variables

Variables are introduced by declarations of the form

	var ID, ID, ..., ID : typeName := expression;       

`typeName` must be an identifier, not a type constructor such as an `array` or `record`. The phrase "`:= expression`" is optional and can only involve literals and named constants. For example:

	var I : Integer := 1;
	var b1, b2 : Boolean;       

* * *

### Operators and Expressions

#### Operators

The operators, in order of precedence, are

category                | operators           | precedence
------------------------|---------------------|----------
Boolean negation        | not                 | highest      
Unary adding operators  | \+   -              |
Multiplying operators   | \*   /   mod        |
Binary adding operators | \+ -                |
Relational operators    | =  /=  <  <=  >  >= |
Logical operators       | and or              | lowest

#### Expressions

For binary operators, both operands must be the same type. Similarly, for assignment compatibility, both the left and right sides must have the same type. Objects are considered to have the same type only if they both have the same type name. Thus, two distinct type definitions are considered different even if they may be structurally identical. This is known as "name equivalence" of types.

#### Short Circuiting

Logical operators `and` and `or` use short-circuit evaluation.

(If you are not familiar with short-circuiting, this means that as soon as the truth value can be determined, evaluation stops. For example, if the first operand of an `and` evaluates `false`, the expression will evaluate `false` no matter what the second operand is, so the second operand is not even evaluated. If the first operand of an `or` evaluates `true`, the second isn't evaluated either.)

* * *

### Statements

All statements are terminated with a semicolon.

#### Assignment statement

(":=" is the assignment operator). For example

	i := 2*I + 5;     

#### If statement

	if x > MAX then 			if x < 10 then
		MAX := x; 					x := x + 1;
	elsif x < MIN then 				y := 2*x;
		MIN := x; 				end if;
	end if;

#### Loop and exit statements

	loop 						while I < n loop
		get(x);						sum := sum + a[I];
		exit when x = SIGNAL;		i := i + 1;
		process(x);				end loop;
	end loop;

#### I/O Statements (for text I/O only)

XL defines only sequential I/O for two basic character streams, standard input and standard output. All I/O is provided by the following procedures:

	procedure put(item : String); // for string literals
	procedure put(item : Char);
	procedure put(item : Integer);
	procedure newLine;
	procedure get(var item : Char);
	procedure get(var item : Integer);
	procedure skipLine;       

* * *

### Subprograms

#### Procedures

Procedures are similar to those in Pascal except that an explicit return is allowed. The program is essentially the outermost procedure (with no parameters) and serves as a starting point for the program.

#### Functions

Functions can return scalar types only, not arrays or records. Only value parameters are allowed. A function returns a value by executing a statement of the form

	return expression;       

#### Parameters

There are two parameter modes in XL: value parameters and variable parameters. Value parameters (passed by copy) are the default. Variable parameters (passed by reference) must be explicitly declared as in

	procedure p(var x : Integer) ...       

#### Subprogram end

The name of the procedure must be repeated at the end of its declaration. For example

	procedure proc1;
		...
	end proc1;


* * *

### Specification Conclusion

OK, so it's sketchy. I know, I know. But that's what I got. Maybe I'll improve it someday, but I think if you're a programmer you'll probably understand it pretty well anyway...

Next we'll discuss... 

Our Strategy
------------

### Structure of Our Compiler

So, now that we have a so-so idea of the language we are compiling, let's design our compiler and think about how we want to implement it.

A basic compiler consists of several pieces. These pieces can be very independent, or very intertwined. I'll try an independent approach here...

The compiler will take an input source file, and output some sort of executable. Let's break it down into tasks:

*   Lexical Analysis (scanning)
*   Semantic Analysis (parsing)
*   Tree Generation
*   Code Generation
*   Interpretation

The approach I'll take is to generate a pseudo-machine code, and we'll write a simple interpreter at the end. This approach is similar to the Pascal P-Code machines and the current Java Virtual Machine approach...

#### Lexical Analysis

First, the compiler will read the input file and lump the characters together into tokens. Using ANTLR 2.x, this job is done using a scanner definition.  One of the big differences between ANTLR 2.x and PCCTS 1.33 is how you specify your scanner.  ANTLR 2.x uses a syntax that is nearly identical to the _parser_ syntax.  You create a recursive-descent scanner as well as a recursive-descent parser.

#### Semantic Analysis

Next, we write grammar rules to pump into ANTLR. These rules will have action code (Java code) attached to them to specify what to do when we see certain patterns of tokens in the input file.

#### Tree Generation

This actually occurs along with the semantic analysis. We'll generate an Abstract Syntax Tree (AST) using ANTLR's built-in tree generation routines. This tree will act as the communication device between the parser and the code generator.

#### Code Generation

Once we have a tree, we'll walk it and write out code.  More on this if I ever finish this tutorial...

#### Interpretation

The interpreter for XL is really simple, and will allow us to test our compiled output without needing to learn specific machine code.

* * *

### Structure of this Tutorial

Now how will we build this compiler? Should we try to do everything at once so you get good and confused? Of course not!

We'll break the work up into steps:

1.  Build a Recognizer
2.  Add a symbol table
3.  Add type checking
4.  Build an AST
5.  Write a Tree Walker to generate code
6.  Write an Interpreter
7.  Test the output code

Let's start our recognizer!

The Scanner
-----------

### Introduction

We'll look at each language element and put together some scanner rules for it. In some cases, I may look at how the same construct would be implemented for other languages, showing some difficulties and how to overcome them. At least that's my plan. We'll see where I end up...

You can download the code for the tutorial or type along. I recommend typing along as it seems to help folks "get" what's going on...

Source code: [antlrtut.zip](antlrtut.zip)

This zip file is also an Eclipse project. If you have the ANTLR-Eclipse plugin installed, you can edit and test the code within eclipse. Simply unzip the antlrtut.zip and use **Import->Existing Project Into Workspace** from within Eclipse.

### Lexical Elements of XL

#### Whitespace

XL declares that blank spaces, tabs, and end-of-line constitute whitespace. Oh, and don't forget the ever (un)popular carriage-return/line-feeds that DOS is so fond of. You should always handle this, especially if you will be running on UNIX and there is even a _chance_ that someone will run a file that has been touched by DOS through your parser. And _just_ to make things more interesting, let's consider the Macintosh as well; it uses a lonely carriage-return to specify a new line.

Most scanner-generation tools force you to specify regular expressions for scanner rules.  ANTLR 2.x simplifies this significantly.  Not only is the ANTLR tool itself significantly simpler, but it makes the job of a grammar writer much easier as well: you only need to learn one format of specification.

In ANTLR 2.x, you specify the scanner grammar in the same manner in which you specify the parser grammar.  We'll discuss some minor differences a bit later, but overall, they are virtually identical.

For our whitespace rule, we specify the following:

```antlr
WS
	:	(	' '
		|	'\t'
		|	'\f'

		// handle newlines
		|	(	"\r\n"  // DOS/Windows
			|	'\r'    // Macintosh
			|	'\n'    // Unix
			)

			// increment the line count in the scanner
			{ newline(); }
		)
		{ $setType(Token.SKIP); }
;
```

The `WS` rule defines what will happen when the scanner sees space (`' '`), tab (`'\t'`), formfeed (`'\f'`) or one of the platform-specific manners of indicating "end of line."  First, we match the character(s) that we see.  If those characters represent "end of line", we call the built-in `newline()` method, which tells the parser to increase its current line number.  (The line number will be used in error messages, so it's very important that you call the `newline()` method.)

After we match the whitepace (whatever it ends up being) we set the token type to Token.SKIP.  `$setType` is a special syntax for setting the type and will be translated by ANTLR into a Java statement.  This is a special token indicator that tells the scanner _not_ to return a token, but to proceed on to the next token it might encounter.

Rules in an ANTLR 2.x scanner may be prefixed by the keyword `protected`, or by no keyword at all.  If a rule is prefixed by `protected`, it means that the rule will _only_ be used as a _helper_ rule.  Perhaps it is the definition of a "decimal digit" which could be used in other rules to help build a number.

Any scanner rules that are _not_ prefixed with the `protected` keyword will be candidates for being called to match "the next token."   ANTLR will create a `nextToken()` method that will pick the appropriate non-`protected` scanner rule to match the next token.  `nextToken()`, in turn, is called by a _parser_ to fetch the next token it will try to match.

Note that ANTLR's syntax allows you to specify _subrules_. You can nest decisions within other decisions by enclosing the lower-level decision in another set of parenthesis.  Think of it as follows:

```antlr
someRuleName
	:	'A' 'B' 'C'		// a top-level alternative
	|	'D' 'E' 'F'		// a top-level alternative
	|	'Q' (			// starting a subrule!
				'R' 'S' // a subrule alternative
			|	'W' 'X'	// a subrule alternative
			)			// end the subrule
	;					// end the rule   
```

The above rule could match the following "words"

   * ABC
   * DEF
   * QRS
   * QWX   

* * *

#### Comments

We got lucky in XL (it uses C++-style comments), but I'll bore you with C-style comments in a minute...

XL defines comments as "all text from // to the end of the current line." So, we get a rule like

```antlr
COMMENT
	:	"//" (~('\n'|'\r'))*
		{ $setType(Token.SKIP); }
	;
```

A few things to notice about this rule:

*   Note that we _didn't_ match the end-of-line character.  This will be matched by our `WS` rule that we have already defined.  What we _do_ need to do is specify which characters are _ok_ to keep in the comment.  As soon as we see `'\n'` or `'\r'`, both of which start an end-of-line sequence, we stop.
*   Those of you familiar with lex will notice that we can't just say ".\*" to represent all non-newline characters. `(~('\n'|'\r'))*` is needed to explicitly state "any character except end-of-line".

I threatened to do C-style comments, so I will.

Let's assume we are using a C compiler that does not keep track of nested comments -- "/\*" starts a comment, and "\*/" ends a comment. It doesn't matter if it supports the C++ style comments or not -- they'll get gobbled just as well as the rest of a comment.

One of the advantages to the new scanner definition in ANTLR 2.x is that a full EBNF grammar is much stronger at representing a language description than regular expressions.   If you take a look at the old PCCTS tutorial, you can see what a mess defining C-style comments can be.  In contrast, a C-style comment in ANTLR 2.0 is a bit simpler:

```antlr
// multiple-line comments
ML_COMMENT
	:	"/*"
		(	/*	'\r' '\n' can be matched in one alternative or by matching
				'\r' in one iteration and '\n' in another. I am trying to
				handle any flavor of newline that comes in, but the language
				that allows both "\r\n" and "\r" and "\n" to all be valid
				newline is ambiguous. Consequently, the resulting grammar
				must be ambiguous. I am shutting this warning off.
			*/

			options {
				generateAmbigWarnings=false;
			}
			:	{ LA(2)!='/' }? '*'
			|	'\r' '\n' {newline();}
			|	'\r' {newline();}
			|	'\n' {newline();}
			|	~('*'|'\n'|'\r')
		)*
		"*/"

		{$setType(Token.SKIP);}
	;
```
   

This rule says the following:

*   Match `/*` to start a comment
*   Keep looping through stuff inside a comment:
    *   match `'*'` as long as it's not followed by `'/'`
        
        NOTE: This subrule uses a _semantic predicate._ We'll talk about these more later on, so don't worry too much about what the `{LA(2) != '/'}?` means.  For now, just read it as "if the _second_ character down the input stream is _not_ a `'/'`, the following alternative (`'*'`) is eligible to match.)
        
    *   match any of the end-of-line indicators.  Note that we must perform the _same_ action we would when we see end-of-line in any other context.  In our recognizer, that action is to simply call the `newline()` method.
    *   match _anything_ other than a newline() or `\*`.  This may _seem_ redundant, but if you _don't_ specify this explicitly, ANTLR will compare it to the previous two rules and report an ambiguity.  This is necessary because based on looking at the next character in the input stream, the scanner wouldn't be able to decide whether to match an end-of-line sequence in the second or third alternative.
*   Match `*/` to end the comment

Note the use of the `( )*` construct.  This is called a _star closure_, and is used to perform looping in a grammar rule.  The _star closure_ will match _zero or more_ of whatever is contained within its parenthesis.

* * *

#### Literals

In XL, there are three types of literals: `Integer`, `Character` and `String`.

**Integer literals** are simple -- they are just a series of digits.  Just so we can demonstrate a `protected` scanner rule, we'll start out by defining what a `DIGIT` is:

```antlr
protected DIGIT
	:	'0'..'9'
	;
```

Because this rule is marked `protected`, we will not get a `DIGIT` token returned to the parser;  `DIGIT` is never directly called from the `nextToken()` method.

Now that we have a helper method that defines what an integer digit is, we'll define an `INTLIT` (integer literal):

```antlr
INTLIT 
	:	(DIGIT)+
	;
```

This rule means "an `INTLIT` is composed of one or more `DIGIT`s".   Note that we do not set the token type to `Token.SKIP` this time. This means that is an `INTLIT` scanner rule is matched while looking at the characters in the input, an `INTLIT` token will be returned to the parser, via the `nextToken()` method.

Note that this will result in the generated INTLIT scanner rule _calling_ the generated DIGIT scanner rule.  A slightly more efficient approach would be to define the INTLIT rule as follows:

```antlr
INTLIT
	:	0'..'9')+
	;   
```

Our final code will use the first, less-efficient method, just to show use of the `protected` keyword.

**Character literals** as fairly simple as well:

```antlr
CHARLIT
	:	'\''! . '\''!
	;
```

Basically, an apostrophe followed by any character, followed by another apostrophe. In the grammar, we will use `CHARLIT` to refer to a character literal.  The "any character" is represented by an unquoted period/dot in the rule.

But notice the exclamation marks (`!`) after the single quotes?  This means, toss these characters away when building the token's string value.  When a CHARLIT token is returned to the parser, its text will _only_ be the single character that was contained within the single-quotes.

**String literals** are very similar.  The trick is to decide what ends a string literal.  We defined string literals as being all text within double-quotes, _but_ the entire literal must be on one line in the source file.  This results in the following scanner rule:

```antlr
STRING_LITERAL
	:	'"'!
		(	'"' '"'!
		|	~('"'|'\n'|'\r')
		)*

		(	'"'!
		|	// nothing -- write error message
		)
	;
```

First, we enter a string when we see a double quote.

What can we see inside a string? Remember, from the spec, two double-quotes is a literal double quote. So we match them.  However, we only want _one_ of them in the resulting string literal.  So we suffix on of the two double-quote characters with another exclamation mark (`!`). This says "match, but toss out the second double-quote".)

But what if we see an end-of-line inside a string? This is illegal, so we'll assume it ends the string.  It will _not_ be matched as part of the contained characters, and will cause an exit of the `( )*` subrule.

After we get out of the `( )*` string body, we have a subrule that checks to see if we can match a closing double-quote or not.  If we match it, we have a value string.  If we don't match it, it can only be because we found end-of-line within the string.  We'll allow this (the empty alternative commented "nothing") but we'll probably want to write an error message saying "String termninated by end-of-line".

Finally, note that we use exclamation marks after the opening and closing quotes of the string.  This removes them from the string literal text represented by the token.

* * *

#### Keywords

XL reserves all keywords -- I may add later what to do with unreserved keywords (for evil languages like PL/1 -- hey! I _like_ to program in PL/1, but I'd hate to write that compiler!)

There are several ways to implement keywords. The simplest is to just directly specify them in the parser grammar.  ANTLR 2.x provides better support for string literals in a grammar than PCCTS 1.33 did.  If you specified a string literal in a PCCTS 1.33 grammar, there was no way to refer to that token when walking through a generated AST (Abstract Syntax Tree).  ANTLR 2.x solves that problem by defining a special token name for each literal in the parser grammar.  For example, a string literal `"end"` specified in the grammar will be represented by a token named `LITERAL_end`.   You can use the name `LITERAL_end` in a tree-walker grammar to refer to that matched literal `"end"`.

Any literals you specify in the parser will be inserted into a nifty little hash table.   You'll need to make the scanner aware that the hash table is being used so it can perform lookups in that table for keyword matches.  We'll discuss this when we talk about indentifiers.

* * *

#### Operators

XL defines several operators, and we'll define them as well... These definitions are _very_ straightforward.  Just assign a scanner rule name to match the text.

```antlr
DOT        : '.'   ;
BECOMES    : ":="  ;
COLON      : ':'   ;
SEMI       : ';'   ;
COMMA      : ','   ;
EQUALS     : '='   ;
LBRACKET   : '['   ;
RBRACKET   : ']'   ;
DOTDOT     : ".."  ;
LPAREN     : '('   ;
RPAREN     : ')'   ;
NOT\_EQUALS : "/=" ;
LT         : '<'   ;
LTE        : "<="  ;
GT         : '>'   ;
GTE        : ">="  ;
PLUS       : '+'   ;
MINUS      : '-'   ;
TIMES      : '*'   ;
DIV        : '/'   ;
```

Note that some of these tokens have common prefixes.  For example, `GT` and `GTE` both start with character `'>'`.  In a Deterministic Finite Automata (DFA)-based scanner, this would not be an issue, as it would try to match the longest possible token.

In a recursive-descent scanner, this is a big issue.  There are two ways to handle this situation:

*   Left-factor the rules in question.
*   Increase the lookahead characters.

Taking the first approach, left factoring, would require that we define a rule to deal with `GT` and `GTE` as follows:

```antlr
GT
	:	'>'  ('=' {$setType(GTE);})?
	;   
```

This rule says "match the `'>'` character.  Then, if I see `'='` as the next character. match it."  The "trick" here is that the "normal" completion of the rule defines the resulting token as the _name of the rule_, which is `GT` in this case.  If we match the _optional subrule_ `('=')?`, where the `( )?` defines its contents as optional, we execute the `$setType(GTE);` statement, which _changes_ the type of the token to `GTE`.

Taking the second approach only involves setting the lookahead amount to a higher number.  This means that we can have the scanner check the next two characters to make a decision, instead of just the first character.  Checking the next two characters in the input stream allows the scanner to correct decide if it should choose to match the `GT` rule or the `GTE` rule.  You can specify a higher lookahead value in the scanner as follows:

```antlr
options {
	k=2; // two characters of lookahead
}   
```

immediately after defining the scanner.  We'll see how the scanner is defined in a moment.  It is important to think carefully whether you want to increase the lookahead amount.  While it may make the grammar simpler, it could also slow it down.   In addition, if you do not carefully study the conflicts and understand them, and simply raise the lookahead amount, you may be masking a real problem.  Try to keep lookahead as low as possible, and _always_ try generating your compiler with `k=1` first.  Make sure you understand what is causing the conflicts, and if you feel certain, and the conflicts are all similar to the `GT`/`GTE` conflict above, raise the lookahead value.

* * *

#### Identifiers

Finally, we talk about identifiers. Basically, they are any sequence of letters and digits that's left. So we get a regular expression like:

```antlr
IDENT
	:	('a'..'z'|'A'..'Z') ('a'..'z'|'A'..'Z'|'0'..'9')*
	;   
```

Very simple. Note that there is no action code here, especially not any that would perform symbol table lookup. I'm very adamant in pushing my view that scanner to parser communication should be a one-way street. This is especially important in multiple-lookahead parsers. If your parser decides when to push and pop scope (a likely scenario), and your scanner is reading two or more tokens ahead, you may be looking at a token in the scanner before its proper scope was pushed or popped. In addition, things like semantic predicates let you make more pointed decisions about how to parse identifiers in the grammar, rather than changing the token type in the scanner.

Think about the set of tokens that can be returned by the scanner we've defined.   Where would keywords like `begin` and `end` be matched?   In our IDENT rule!

We will be defining keywords within our parser, which will end up generating tokens called `LITERAL_begin` and `LITERAL_end`, for example.  Recall that these literals will be inserted in a special hash table.  To utililize this hash table, modify the IDENT rule as follows:

```antlr
IDENT
	options {testLiterals=true;}
	:	('a'..'z'|'A'..'Z') ('a'..'z'|'A'..'Z'|'0'..'9')*
	;
```

Note the `testLiterals=true` option.  This generates code that will lookup a name in the literals hash table _if_ the IDENT rule matches a word.   This implies that every literal you specify in the parser grammar _must_ be matchable by the IDENT rule.

If a word such as `"end"` is found in the literals table, the token type will be changed to be something like `LITERAL_end` before being returned to the parser.

* * *

#### Putting the Lexical Elements Together

What do we have so far? Let's look at it in one lump, shall we?

But first, we need to tell ANTLR that we're creating a scanner and specify a new options (such as lookahead):

```antlr
//---------------
// The XL scanner
//---------------

class XLLexer extends Lexer;

options {
	charVocabulary = '\0'..'\377';
	testLiterals=false;    // don't automatically test for literals
	k=2;                   // two characters of lookahead
}   
```

First, we specify that we are creating a scanner called `XLLexer`, subclassing the `Lexer` class.  `Lexer` is an abstract class that defines the basis for an ANTLR-generated scanner.  (You can define scanners that subclass other scanners that you have defined, but that is outside the scope of this tutorial.  Please see the ANTLR documentation at [http://www.antlr.org/](http://www.antlr.org/) for details.)

Next we specify four options.  These are:

*   `charVocabulary = '\0'..'\377';`  
    This defines the valid set of characters than can be recognized by the ANTLR scanner.   This is necessary to match expresions like `~'a'` in a scanner rule.   We need to know what the other possible characters are...  
    Also - the characters mentioned in the charVocabulary are the _only_ characters that will be allowed in the source input.  Any character encountered that is not in the charVocabulary will be flagged as invalid.  The range we specify above is the entire UNICODE character set.

*   `testLiterals=false;`  
    Specifies that by default we do not want all rules to check the literals table for possible matches.  Only rules that have their own testLiterals option will perform that lookup.
*   `k=2;`  
    This specifies that the scanner will look at the next two characters of the input stream to determine which alternative to choose.  The extra lookahead is only used when actually necessary to disambiguate.

After specifying the name and options for the scanner, we specify the rules.  The following is our entire scanner:

```antlr
//----------------------------------------------------------------------------
// The XL scanner
//----------------------------------------------------------------------------

class XLLexer extends Lexer;

options {
	charVocabulary = '\0'..'\377';
	testLiterals=false;    // don't automatically test for literals
	k=2;                   // two characters of lookahead
}

// Single-line comments
COMMENT
	:	"//" (~('\n'|'\r'))*
		{ $setType(Token.SKIP); }
	;

// Literals
protected DIGIT
	:	'0'..'9'
	;

INTLIT 
	:	(DIGIT)+
	;

CHARLIT
	:	'\''! . '\''!
	;

// string literals
STRING_LITERAL
	:	'"'!
		(	'"' '"'!
		|	~('"'|'\n'|'\r')
		)*

		(	'"'!
		|	// nothing -- write error message
		)
	;

// Whitespace -- ignored
WS
	:	(	' '
		|	'\t'
		|	'\f'

		// handle newlines
		|	(	"\r\n"  // DOS/Windows
			|	'\r'    // Macintosh
			|	'\n'    // Unix
			)

			// increment the line count in the scanner
			{ newline(); }
		)

		{ $setType(Token.SKIP); }
	;

// an identifier.  Note that testLiterals is set to true!  This means
// that after we match the rule, we look in the literals table to see
// if it's a literal or really an identifer

IDENT
	options {testLiterals=true;}
	:	('a'..'z'|'A'..'Z') ('a'..'z'|'A'..'Z'|'0'..'9')*
	;

// Operators
DOT        : '.'   ;
BECOMES    : ":="  ;
COLON      : ':'   ;
SEMI       : ';'   ;
COMMA      : ','   ;
EQUALS     : '='   ;
LBRACKET   : '['   ;
RBRACKET   : ']'   ;
DOTDOT     : ".."  ;
LPAREN     : '('   ;
RPAREN     : ')'   ;
NOT_EQUALS : "/="  ;
LT         : '<'   ;
LTE        : "<="  ;
GT         : '>'   ;
GTE        : ">="  ;
PLUS       : '+'   ;
MINUS      : '-'   ;
TIMES      : '*'   ;
DIV        : '/'   ;
```

Let's move on to the parser grammar.  

The Parser
----------

Now we'll look at a parser grammar for XL. Note that we are building just a recognizer at this state, and the grammar may (and will!) change.

Let's start at the top, shall we?

### Program Specification

XL has one program per file, so this is really our starting rule.

```antlr
program
	:	"program" IDENT EQUALS 
			subprogramBody 
		DOT
		// end-of-file
	;
```

Pretty straightforward. Note that so far, we don't care that we are **definitely** declaring that `IDENT` here. We will later, though...

* * *

### Subprogram Bodies

A subprogram body is a little (or big) block of code that makes up the "what I do" part of a program, procedure, or function. It's not bad to define, but it will get a bit tricker later. Well, not too much so.

	subprogramBody
		:	(basicDecl)*
			(procedureDecl)*
			"begin"
				(statement)*
			"end" IDENT
		;

So what is it? Basically, define your local variables, types and constants (`basicDecls`). There can be zero or more of these, hence the use of the `( )*` closure. Then, define any nested procedures or functions. Again, zero or more of these. Finally, we get to the definition of what the current program/procedure/function does. This starts with a "begin", has zero or more statements in it, and is ended by "end" `IDENT`. Note that the XL spec stated that the identifer that ends a subroutine must match the beginning one. Right now, we have no way of doing that, as the name for the subroutine is outside the scope of this rule. We'll handle this later, though, and in a pretty neat way I must say. Yacc can't hold a candle to it, you'll see!

Note that the XL spec said nothing about "procedures must be declared after vars, consts and types." This was one of the many things that the language designer told us during midnight interrogation... Similar to Pascal's definition order (`CONST` `TYPE` `VAR` `FUNCTION`/`PROCEDURE`) but not quite that rigid.

* * *

### Basic Declarations

XL has three main declarations: variables, constants, and types:

```antlr
basicDecl
	:	varDecl
	|	constDecl
	|	typeDecl
	;
```

Just so this section isn't so short, I'll define `varDecl` and `constDecl` here.

A variable declaration looks like:

```antlr
varDecl
	:	"var" identList COLON typeName
		(BECOMES constantValue)?
		SEMI
	;
```

Unlike Pascal, each declaration must start with "var"; there is no "VAR section" that starts with the keyword VAR. The `varDecl` states that you can defined any number of idents at once, and you can optionally initialize the variable(s).

A constant declaration is similar to a variable declaration, except that you use the keyword "constant" and must assign a value:

```antlr
constDecl
	:	"constant" identList COLON typeName
		BECOMES constantValue SEMI
	;
```

In the above rules, subrules `identList` and `constantValue` are mentioned. These are:

```antlr
identList
	:	IDENT (COMMA IDENT)*
	;
```

which says "one or more `IDENT`s separated by `COMMA`s", and

```antlr
constantValue
	:	INTLIT
	|	STRING_LITERAL
	|	IDENT
	;
```

which is pretty self-explanatory. One thing to note, though, there there is nothing right now that prevents us from using a variable `IDENT` instead of a constant `IDENT`. That's handled later...

* * *

### Type Declarations

XL defines three user-defined types: arrays, records and enumeration types. In my project, I only defined arrays and records, so for now, I'll skip enumeration types. I may add them at another time.

A type declaration is either an array declaration or a record declaration:

```antlr
typeDecl
	:	"type" IDENT EQUALS
		(	arrayDecl
		|	recordDecl
		)
		SEMI
	;
```

I got a bit fancier here by using a subrule to say "array or record" instead of defining a new rule for it. This is also a bit more efficient than the extra function call created by another rule there. Basic rule of thumb -- if a subrule is clear, and not deeply nested, feel free to use it. However, watch out for using several nested subrules, as the meaning can get hidden quickly.

Arrays are defined as `ARRAY [x..y] OF type`. Only one-dimensional, pretty simple:

```antlr
arrayDecl
	:	"array" LBRACKET integerConstant DOTDOT integerConstant RBRACKET
		"of" typeName
	;

integerConstant
	:	INTLIT
	|	IDENT // again, a constant...
	;
```

Records are defined as `RECORD x,y,z:typename; END RECORD.` Again, simple until we have to know what it means:

```antlr
recordDecl
	:	"record" (identList COLON typeName SEMI)+ "end" "record"
	;
```

So what is this `typeName` I keep bringing up? Well, it's either one of the predefined types, `Integer` or `Boolean`, or it's a user-defined type (which means its an `IDENT`), so:

```antlr
typeName
	:	IDENT
	|	"Integer"
	|	"Boolean"
	;
```

Enough about types, now, on to

* * *

### Procedure Declarations

A procedure in XL is similar to a program, so basically, it's a small heading followed by a `subprogramBody`:

```antlr
procedureDecl
	:	"procedure" IDENT (formalParameters)? EQUALS
		subprogramBody
		SEMI
	;
```

At the time I originally did this project, I only did procedures, not functions. Perhaps I'll add them later... (You may think "boy he left a lot out," but Dr. Moore made a large subset of the project required, and I did a few extra point things, but not the whole thing. I _was_ working full-time you know...)

Notice that the `formalParameters` are optional... Let's define what they look like:

```antlr
formalParameters
	:	LPAREN parameterSpec (SEMI parameterSpec)* RPAREN
	;
```

Again we see the familiar `x (COMMA x)` notation -- the ()\* closure is very handy and efficient for matching lists of things...

```antlr
parameterSpec
	:	("var")? identList COLON typeName
	;
```

You'll notice that this is quite a bit like a variable declaration, except that `VAR` is optional, and there's no semicolon after it. We'll handle it a bit differently as well, once we add action code.

Next, we'll look at...

Statements
----------

There are seven statements in XL (I told you it was a small language): assignment, exit, procedure call, return, if-then, loop and I/O.

```antlr
statement
	:	exitStatement
	|	returnStatement
	|	ifStatement
	|	loopStatement
	|	ioStatement
	|	procedureCallStatement
	|	assignmentStatement
	|	endStatement
	;
```

### Assignment Statement

The assignment statement is pretty much like Pascal's assignment statement, except that there are no type conversions allowed.

```antlr
assignmentStatement
	:	variableReference BECOMES expression SEMI
	;
```

Very simple, but the pieces (variable and expression) are somewhat complex -- we'll look at them after all the statements are done. We'll look at `variableReference` later...

* * *

### Exit Statement

The exit statement tells a loop it's done. It looks like

```antlr
exitStatement
	:	"exit" "when" expression
	;
```

### Procedure Call Statement

A procedure call takes the form

```antlr
procedureCallStatement
	:	IDENT (actualParameters)? SEMI
	;

actualParameters
	:	LPAREN (expression (COMMA expression)*)? RPAREN
	;
```

I extended XL just a bit for this. The original XL compiler I wrote had

```antlr
actualParameters
	:	LPAREN expression (COMMA expression)* RPAREN
	;
```

which basically means that if you have parens, you **must** have at least one expression in them. I thought it would be nice to allow `myfunc()` as a valid procedure call.

* * *

### Return Statement

The XL return statement tells a procedure or function to return immediately. Keeping in mind that I did not implement functions (yet, at least), `return` cannot take an expression argument. Therefore, it's just

```antlr
returnStatement
	:	"return" SEMI
	;
```

### If Statement

The XL "if-then-elsif-else-end if" statement is a fairly simple `if` type statement. This is because it is defined as a closed `if` statement; that is, there is a definite "I'm done" part to the `if` statement. Those of you who are familiar with the "dangling `else`" conflict present in the C language may realize that if you close the `if` statement, the conflict disappears. First, I'll present the rule for XL, then I'll talk a bit about how to do the `if` statement for C.

XL's `if` statement looks like:

```antlr
ifStatement
	:	"if" expression "then"
		( statement )*
		(	"elsif" expression "then"
			( statement )*
		)*
		( "else" ( statement )* )?
		"end" "if" SEMI
	;
```

Which is the most straightforward way to write the rule. However, you may notice some redundancy in the rule -- there are two separate places where we'll need to deal with `expression` evaluation. Not a big deal now, but we'd need to duplicate code later. After staring at the rule for a bit, we might try splitting out the "`expression` THEN possible ELSE" stuff into a separate rule, and call it recursively if we get an ELSIF. In other words:

```antlr
ifStatement
	:	"if" ifPart "end" "if" SEMI
	;

ifPart
	:	expression "then"
		(statement)*
		(	"elsif" ifPart
		|	"else" (statement)*
		)?
	;
```

Which is much less redundant.

Earlier, I threatened to look at the "dangling else" problem. In ANTLR, this is an easy problem to solve. Basically, the problem is that in a statement like

	if (x)
		if (y)
			myfunc();
		else
			yourfunc();
             

It isn't terribly apparent (to the compiler at least) which `if`-condition owns the "else" clause. The compiler can't tell if the inner `if` is an "if-without-an-else" inside an "if-with-an-else," or vice-versa.

On the other hand, the language designers decided that the else will be owned by its most recent `if` that can possibly own it. In this case, the `else` would clearly belong to the `(y)` condition. However, the compiler doesn't know this.

In a yacc grammar, an if-statement for C might be written

```
ifStatement
	:	IF LPAREN expression RPAREN statementList
	|	IF LPAREN expression RPAREN statementList
		ELSE statementList
	;
```

which causes yacc to report a "shift/reduce" conflict -- yacc can't tell if you intended to have an "if-without-else" (the first alternative) be nested inside an "if-with-else" (the second alternative.) It doesn't know if it should reduce the first alternative when it sees the ELSE, or whether it should keep looking (shift the ELSE.) Fortunately, yacc's default is to shift the else, basically grouping the ELSE with its closest IF, which is exactly what we want.

To get yacc to stop reporting this (and explicitly specify which alternative we intend) requires the use of either some very messy (and hard-to-follow) rules, or explicit precedence assignment. Neither is pretty, or very readable for someone else.

In ANTLR, the C if statement would look like

```antlr
ifStatement
	:	"if" LPAREN expression RPAREN (statement)*
		( "else" (statement)* )?
	;
```

which, of course, causes ANTLR to report a conflict. To get rid of this conflict, we can use a syntactic predicate (aka "guess" mode) which basically says "try this -- if it works, use it. Otherwise, try the next alt."

Applying a nice syntactic predicate to the `ifStatement` rule above yields

```antlr
ifStatement
	:	"if" LPAREN expression RPAREN (statement)*
		(	("else")=> "else" (statement)* )?
	;
```

which can a bit easier to read if you break the `( )?` subrule into a "this-or-nothing" subrule.

```antlr
ifStatement
	:	"if" LPAREN expression RPAREN (statement)*
		(	("else")=> "else" (statement)*
		|	( ) // nothing
		)
	;
```

The predicate says "try matching an ELSE. If you can, then use the first alt. Otherwise, use the second alt (the "nothing" alternative.) Note that we will be using the more compact ``( )?`` form for our compiler.

* * *

### Loop Statement

The loop statement in XL is pretty simple. (Anyone sick of the word "simple" yet? But it's true, really it is.) It's a set of statements enclosed by `loop` and `end loop`, with an optional `while` clause in front of it. It looks like

```antlr
loopStatement
	:	("while" expression)? "loop"
		( statement )*
		"end" "loop" SEMI
	;
```

Not too bad, eh? Well, just to confuse matters a bit, what would happen if XL allowed an `END` statement, like in BASIC? (BASIC's `END` statement exits the program immediately.) Come to think of it, what the heck, let's add one!

```antlr
endStatement
	:	"end" SEMI
	;
```

Now why would I be mentioning this here? Let's look at some XL sample code:

	program sponge =
		var x : integer := 1;
		begin
			while x < 10 loop
				x := x + 1;
			end;
		end loop
	end sponge.

Kind of contrived (and useless, I know), and simple, until you look closer...

What happens when the parser sees the `end` inside the loop? Let's look at the `loopStatement` rule again (repeated here 'cause you're a programmer which means you're too lazy to turn back a page):

```antlr
loopStatement
	:	("while" expression)? "loop"
		( statement )*
		"end" "loop" SEMI
	;
```

Assuming k=1 lookahead, we have a problem: how does the parser determine that the `end` in loop is an `endStatement` and not the `end` that is the first part of `end loop`? The answer is simple: with k=1 lookahead, it can't. It will always try to make it an `endStatement`, and will get a syntax error on `loop`. (Try this in the final recognizer, just for kicks.) You'll also get a conflict warning in ANTLR. (Note that you'll have the same conflict with `end if` in `ifStatement` -- our solution should, and will, handle both situations.)

So how do we solve this? Two possibilities: use k=2 lookahead, or use a syntactic predicate.

To use k=2 lookahead, you'd just need to specify `k=2` as a parser option. This will make the ANTLR run take a bit more time, but should only affect checks that require two-token lookahead.

In our grammar, we only have one place (this one) that causes a conflict. (Remember that our `ifStatement` is closed with an `end if`, so there's no dangling `else` conflict.) So let's fix the problem in that spot using syntactic predicates. (I also want to cover syntactic predicates a bit, so I won't go with the easier k=2 fix... In real life, I think it might make a slightly faster parser to use k=2 for this case. Syntactic predicates should probably be reserved for cases when you don't know how much lookahead you'll really need, or the known lookahead is lengthy.)

We can tell ANTLR exactly how to distinguish an `endStatement` from the end of a loop. How? Simple (for me at least, 'cause I already know...):

```antlr
statement
	:	("end" SEMI)=> endStatement
		assignmentStatement
		. . .
	;
```

Now what the heck does that mean? Basically, it says "try to match `END`, then `SEMI`." If it works, then we really want the `endStatement`. If we didn't get a match, try the next alternative, and so on...

There's one problem that I have with the above statement. I don't like spelling out `END SEMI` in a rule other than `endStatement`. It just seems like bad isolation of rules. I'm a bit picky at times, though. What I'd rather do is say to myself, hmmm, `END SEMI` is an `endStatement`, so why not put that inside the `( )=>` -- like this:

```antlr
statement
	:	(endStatement)=> endStatement
	| assignmentStatement
	. . .
	;
```

(Note: This will be a bit less efficient than spelling it out -- the "guess" that ANTLR will make will need to make a function call to `endStatement` to check for `END SEMI` rather than just directly check for `END SEMI` in the statement rule. However, as long as you don't do this too much, the penalty for a function call is minimal on most machines nowadays. Just keep it to a minimum!)

What if `endStatement` were a bit more involved, such as

```antlr
endStatement
	:	"end" SEMI expression SEMI
	;
```

(Kinda useless, but so are most examples.) All we need to predict it are the `END` and `SEMI`. If we keep the predicate as it is, `(endStatement)?`, the "guess" would keep going through expression (which could be pretty long) and the next SEMI. That would be mighty wasteful. So, in a case where we want to limit the lookahead, it's probably better to sacrifice readability a bit and limit the lookahead to just END SEMI:

```antlr
statement
	:	("end" SEMI)=> endStatement
	|	assignmentStatement
	. . .
	;
```

(Look familiar?) You may ask "why not just put the `(END SEMI)?` in the `endStatement` rule itself?" Unfortunately, ANTLR complains about a rule like

```antlr
endStatement
	:	("end" SEMI)=> "end" SEMI expression SEMI
	;
```

basically telling you "why bother with a predicate -- there's only one alternative!" I think I'd prefer to be able to specify it like this, and have ANTLR hoist the predicate up into the statement rule (and possibly optimize it out of the `endStatement` rule altogether.) But let's work with the great tool we currently have, shall we?

So, we end up with two new rules:

```antlr
loopStatement
	:	("while" expression)? "loop"
		(statement)*
		"end" "loop" SEMI
	;

endStatement
	:	"end" SEMI
	;	
```

and we modify the `statement` rule to look like

```antlr
statement
	:	(endStatement)=> endStatement
	|	assignmentStatement
	|	exitStatement
	|	procedureCallStatement
	|	returnStatement
	|	ifStatement
	|	loopStatement
	|	ioStatement 
	;
```

But is the order important in the `statement` rule? Of course, but it's not so obvious until you look at what the generated code will say. The following isn't actual generated code, but pseudocode that will show what happens in the statement rule:

	statement {
		if next token is "end" and guess(endStatement) worked
			do endStatement
		else if next token is IDENT
			do assignmentStatement
		else if next token is "exit"
			do exitStatement
		else if next token is IDENT
			do procedureCallStatement
		else if next token is "return"
			do returnStatement
		else if next token is "if"
			do ifStatement
		else if next token is ("while" or "loop")
			do loopStatement
		else if next token is ("put" or "get")
			do ioStatement (defined later)
	}

A few things to notice here:

*   Most of the conditions in this code are simple, one-test comparisons
*   The condition for `endStatement` is much more complex
*   Oooops! I see a conflict between `assignmentStatement` and `procedureCallStatement` -- hold this thought for a minute...
*   Notice that `loopStatement`'s lookup condition had two parts to it: WHILE or LOOP -- this is because the WHILE section is optional, making the FIRST set for `loopStatement` be {WHILE, LOOP} (`ioStatement` will be defined in a bit as a GET or PUT call...)
*   All of the conditions are evaluated in the order that the rules are specified
*   All of the conditions except `assignmentStatement` and `procedureCallStatement` have a unique test condition; their FIRST sets are disjoint

Our goal when parsing is to make things as fast as possible. Therefore, based on the above information, we should

*   Order the rules such that the most likely/most used rules come first -- that means less conditions to test
*   Order the rules such that rules with nasty conditions come later. Note that if the nasty condition is required to disambiguate two alternatives, it needs to be in the first of the two alternatives

If possible, determine which statements are more likely to be used. Do some statistical sampling of user code (if possible) and order the rules accordingly. In this case, I think it's less likely that the user would use an END statement, and on top of that, it's more complex, so let's put it at the bottom of the list.

We also need to disambiguate `assigmentStatement` and `procedureCallStatement`. Recall that they look like:

```antlr
assignmentStatement
	:	variableReference BECOMES expression SEMI
	;

procedureCallStatement
	:	IDENT (actualParameters)? SEMI
	;
```

So, how do we tell them apart? First, let's look at the FIRST sets of each (the sets that contain all tokens that can be the first terminal in a rule.) Impossible until we define `variableReference`, so let's do that now (sorry about the jumping around here..)

```antlr
variableReference
	:	IDENT
		(	LBRACKET expression RBRACKET
		|	DOT IDENT
		)*
	;
```

which says a variable reference is an IDENT followed by any number of array or field dereferences.

Without going into a long discussion about FIRST sets, what is the FIRST set of `assignmentStatement`? By inspection, it's whatever the FIRST set is of `variableReference`, which is {IDENT}. Again, by inspection, the FIRST set of `procedureCallStatement` is also {IDENT}. There, the conflict becomes very clear. On one token alone, IDENT, the parser cannot determine which rule is proper. Looking at the pseudocode above, it becomes obvious that the parser will, in fact, always choose whichever rule is specified first with a FIRST set of {IDENT}, which is bad. So, we need to add a predicate to one of the rules. But which one? Lets look at what would be needed for each. What makes each unique? `assignmentStatement` could start out with

	IDENT BECOMES . . .      // just ident := expression
	IDENT LBRACKET . . .     // starting with array reference
	IDENT DOT . . .          // starting with field reference

and `procedureCallStatement` could start out with (after looking at `actualParameters`)

	IDENT SEMI         // No parameters (or parens)
	IDENT LPAREN . . . // parameters (or just parens)

Obviously, they are unique with two tokens of lookahead. So, we're back to the decision of "do we bump lookahead to k=2, or use predicates." k=2 would resolve the conflict, but my gut says we should use a syntactic predicate here. But which one?

The predicates would look like:

```antlr
statement
	:	. . .
	|	(IDENT (BECOMES|LBRACKET|DOT))=> assignmentStatement
	|	(IDENT (SEMI|LPAREN))=> procedureCallStatement
```

Well, the predicate for `procedureCallStatement` is actually a bit simpler. We probably should use it and put its alternative ahead of the one for `assignmentStatement`.\\ If your research proves that `assignmentStatements` are found in code more often than `procedureCallStatements`, you may want to reconsider, though.

So what do we end up with? Using our basic rule of "keep the simple stuff near the top" and adding syntactic predicates for `endStatement` and `procedureCallStatement`, we get (without taking into account statistical frequency of each statement):

```antlr
statement
	:	exitStatement
	|	returnStatement
	|	ifStatement
	|	loopStatement
	|	ioStatement
	|	(IDENT (LPAREN|SEMI))=> procedureCallStatement
	|	assignmentStatement
	|	(endStatement)=> endStatement
	;
```

Now, on to our final statement...

* * *

### I/O Statement

The I/O statement is really a procedure call to four predefined functions, `get`, `put`, `skipLine` and `newLine`. These really should be handled just like any other procedure call, but I'm putting them here just because that's how I did it the first time around, and because I don't want to mess with special pre-defined identifiers in the symbol table later...

```antlr
ioStatement
	:	"put" LPAREN expression RPAREN SEMI
	|	"get" LPAREN variableReference RPAREN SEMI
	|	"newLine" (LPAREN RPAREN)? SEMI
	|	"skipLine" (LPAREN RPAREN)? SEMI
	;
```

Note that I was nice and allow the user to put "()" after `skipLine` and `newLine`, just like we did for procedure calls in general.

Easy enough. Now onto the sticky stuff...

Expressions
-----------

XL expressions are pretty similar to any old expressions you'd find in any old language (or in any new language, for that matter.) The stuff we do here will be pretty much the same in any parser or compiler you write in ANTLR, so pay attention! I know you're falling asleep after reading thirty-odd pages of drivel, but hang with me for a little longer!

There are a few things to notice about using ANTLR when you write expressions. The big one is that ANTLR is an LL(k) parser-generator. That means several things, but the biggest (with regard to expressions) is that left recursion is **not** allowed. See some compiler books for the details; in a nutshell

```antlr
a : a PLUS a ; 
```

would generate code like

	a() {
		a();
		match(PLUS);
		a();
	} 

which would obviously recursively call itself until you run out of stack space. There are several ways around this. You have probably seen the following type of code used to describe parts of expressions:

```antlr
a
	:	a PLUS a
	|	t
	; 
```

To get rid of the left-recursion that renders LL(k) grammars helpless, you can use some wonderfully-mechanical algorithms to turn it into

```antlr
a
	:	a1 a_tail
	;
a1
	:	t
	;
a_tail
	:	PLUS t a\_tail
	|	// nothing
	; 
```

Yuck! It works, but how long did you have to think about it to understand what it means?

Notice that the mechanically-generated code utilizes tail-recursion (recursion as the last element of a rule, also known as right-recursion) to keep adding (PLUS t) to the end of the "a" thing we are creating. Eventually, it ends with the empty (epsilon) production in place of `a_tail`

Thinking about right-recursion, recall that right recursion can always be replaced very easily by a loop. (From where do you recall this? I dunno. Some programming class.) In ANTLR, we have the advantage of being able to use ()\* and ()+ closures using EBNF notation. In other words, we can easily represent loops. So, a rule like

```antlr
a_tail
	:	PLUS t a_tail
	|	// nothing
	; 
```

can be re-written as

```antlr
a_tail
	: (PLUS t)*
	; 
```

which is much better so far. Now look what we have:

```antlr
a
	:	a1 a_tail
	;
a1
	:	t
	;
a_tail
	:	(PLUS t)*
	; 
```

Notice something about `a_tail` now? Without the tail recursion, it's only used in one place, so we can substitute it. And while we're at it, let's substitute `a1` in place as well, yielding

```antlr
a
	:	t (PLUS t)*
	; 
```

When we look at that, we see that that's exactly what we wanted. Basically, PLUS is associative, so the order in which you add things really doesn't matter. You can keep saying "PLUS something else" at the end of our PLUS expression. The above notation is actually very good, because it matches what your brain does as it adds new parts to the expression as it reads left-to-right.

The moral of the above nonsense is that an expression-like rule such as

```antlr
a 
	:	a OPERATOR a
	|	t
	; 
```

can be re-written

```antlr
a
	:	t (OPERATOR t)*
	; 
```

Now that we're armed, lets start organizing for our attack on XL's expression rules!

* * *

### Precedence

In the description of XL above, we defined that the operators have the following precedences:

category                | operators           | precedence
------------------------|---------------------|----------
Boolean negation        | not                 | highest      
Unary adding operators  | \+   -              |
Multiplying operators   | \*   /   mod        |
Binary adding operators | \+ -                |
Relational operators    | =  /=  <  <=  >  >= |
Logical operators       | and or              | lowest

What exactly does this mean? Well, it means that if we see a `not` somewhere in an expression, we should resolve it before we resolve any other operators (unless, of course, there are specific precedences specified by using parentheses.) So if we had an expression like

	a and x or not y and 2 + -q * s < 10 

we would first evaluate the "not" (let's use parentheses to specify evaluation grouping)

	a and x or (not y) and 2 + -q * s < 10 

then take care of the unary -

	a and x or (not y) and 2 + (-q) * s < 10 

then the \*

	a and x or (not y) and 2 + ((-q) * s) < 10 

then the binary +

	a and x or (not y) and (2 + ((-q) * s)) < 10 

then the relational <

	a and x or (not y) and ((2 + ((-q) * s)) < 10) 

	then the boolean `and` and `or`

	(((a and x) or (not y)) and ((2 + ((-q) * s)) < 10)) 

Whoa! Is that right? In most languages, `and` has a higher precedence than `or`. But in our precedence chart for XL, `and` and `or` have the same precedence. So while we might _expect_ to see the evaluation proceed like

	((a and x) or ((not y) and ((2 + ((-q) * s)) < 10))) 

that's not how it goes, because `and` and `or` have the same precedence, we have to just read them left to right. (In this situation, XL's design is less intuitive because of the preponderance of other languages that give `and` a higher precedence than `or`. But that was the language designer's choice, and we'll live with it.)

So what do we do with these precedences? Basically

*   each level of precedence represents a rule in the grammar
*   Each operator in that precedence level is one alternative in that rule
*   The rules will nest with the highest precedence operators being the most-deeply nested rules
*   Each precedence level will only imbed the next higher precedence level.
*   The highest precedence level will only use "primitive" expression elements, such as variable references and literals.

So lets start at the top of the precedence chart. I'm going to go out on a limb and avoid the terms `term`, `factor`, `simple expression`. I could never remember which was supposed to be which from a mathematics point of view. Instead, I'll follow Sun's lead in their Java grammar, and use more descriptive names.

We need to start with our primitive expression elements. In XL, these are constant values, variable references, and parenthesized expressions. Remember, parenthesizing an expression basically makes it the most important thing that is currently being evaluated.

```antlr
primitiveElement
	:	constantValue
	|	variableReference
	|	LPAREN expression RPAREN
	;
```

So far so good. Now, let's just walk down the precedence chart. First, boolean negation:

```antlr
booleanNegationExpression
	:	("not")* primitiveElement
	;
```

Notice that the `NOT` is optional here, and you can have as many of them as you like. That is because we want to be able to just pass any expression element through a rule without modifying it, enabling us to have an expression just be a `primitiveElement` if that's what the user wants. Next, we have unary PLUS and MINUS:

```antlr
signExpression
	:	(PLUS|MINUS)* booleanNegationExpression
	;
```

Notice that the `( )*` can evaluate to "no operators," meaning that `booleanNegationExpression` can be passed straight through this precedence level without any modification.

Let us have as many plusses or minuses in front of our expression so far. Next, we have our multiplying operators, \*, / and `mod`:

```antlr
multiplyingExpression
	:	signExpression ((TIMES|DIV|"mod") signExpression)*
	;
```

Which lets us keep tacking on multiplying operators ad nauseum.

Notice that this precedence rule imbedded the previous rule, and again, can let it just pass `signExpression` right on through without modifying it. By now, things should be seeming a bit more clear. Next, we move onto the binary adding operations:

```antlr
addingExpression
	:	multiplyingExpression ((PLUS|MINUS) multiplyingExpression)*
	;
```

Will we have a conflict between an (PLUS|MINUS) acting as a binary add operator and one acting as a unary operator? No. The reason is that if we see one in between two partial expressions, it must be a binary operator. If we see another right after it (and any number of them after that) they must be working (covertly) as unary operators. Next we have the relational operators:

```antlr
relationalExpression
	:	addingExpression ((EQUALS|NOT_EQUALS|GT|GTE|LT|LTE) addingExpression)*
	;
```

Finally, we get to the operators with the lowest precedence. At this level, we can call the result a full expression. This level encompasses `and` and `or` operators:

```antlr
expression
	:	relationalExpression (("and"|"or") relationalExpression)*
	;
```

Tack on as many and/or partial expressions as you like...

Now that we've gotten through expressions, it really doesn't seem so bad. The precedence table actually made it easier, rather than more confusing.

At this point, we have specified the entire grammar for an XL parser. You've already seen the entire scanner. The following is our parser:

```antlr
//-----------------------------------------------------------------------------
// Define a Parser, calling it XLRecognizer
//-----------------------------------------------------------------------------
class XLRecognizer extends Parser;

options {
	defaultErrorHandler = true;      // Don't generate parser error handlers
}

// Define some methods and variables to use in the generated parser.
{
	// Define a main
	public static void main(String[] args) {
		// Use a try/catch block for parser exceptions
		try {
			// if we have at least one command-line argument
			if (args.length > 0 ) {
				System.err.println("Parsing...");

				// for each directory/file specified on the command line
				for(int i=0; i< args.length;i++)
					doFile(new File(args[i])); // parse it

			} else {
				System.err.println("Usage: java XLRecogizer <directory name>");
			}

		} catch(Exception e) {
			System.err.println("exception: "+e);
			e.printStackTrace(System.err);   // so we can get stack trace
		}
	}

	// This method decides what action to take based on the type of
	//   file we are looking at

	public static void doFile(File f) throws Exception {
		// If this is a directory, walk each file/dir in that directory
		if (f.isDirectory()) {
			String files[] = f.list();
			for(int i=0; i < files.length; i++) {
				doFile(new File(f, files[i]));
			}

		// otherwise, if this is a java file, parse it!
		} else if ((f.getName().length()>5) &&
			f.getName().substring(f.getName().length()-3).equals(".xl")) {
				System.err.println("-------------------------------------------");
				System.err.println(f.getAbsolutePath());
				parseFile(new FileInputStream(f));
		}
	}

	// Here's where we do the real work...
	public static void parseFile(InputStream s) throws Exception {
		try {
			// Create a scanner that reads from the input stream passed to us
			XLLexer lexer = new XLLexer(s);

			// Create a parser that reads from the scanner
			XLRecognizer parser = new XLRecognizer(lexer);

			// start parsing at the compilationUnit rule
			parser.program();

		} catch (Exception e) {
			System.err.println("parser exception: "+e);
			e.printStackTrace();   // so we can get stack trace 
		}
	}
}

// the following tag is used to find the start of the rules section for
//   automated chunk-grabbing when displaying the page

program
	:	"program" IDENT EQUALS 
		subprogramBody 
		DOT
		// end-of-file
	;

subprogramBody
	:	(basicDecl)*
		(procedureDecl)*
		"begin"
		(statement)*
		"end" IDENT
	;

basicDecl
	:	varDecl
	|	constDecl
	|	typeDecl
	;

varDecl
	:	"var" identList COLON typeName
		(BECOMES constantValue)?
		SEMI
	;

constDecl
	:"constant" identList COLON typeName
		BECOMES constantValue SEMI
	;

identList
	:	IDENT (COMMA IDENT)*
	;

constantValue
	:	INTLIT
	|	STRING_LITERAL
	|	IDENT
	;

typeDecl
	:	"type" IDENT EQUALS
		(	arrayDecl
		|	recordDecl
		)
		SEMI
	;

arrayDecl
	:	"array" LBRACKET integerConstant DOTDOT integerConstant RBRACKET
		"of" typeName
	;

integerConstant
	:	INTLIT
	|	IDENT // again, a constant...
	;

recordDecl
	:	"record" (identList COLON typeName SEMI)+ "end" "record"
	;

typeName
	:	IDENT
	|	"Integer"
	|	"Boolean"
	;

procedureDecl
	:	"procedure" IDENT (formalParameters)? EQUALS
		subprogramBody
		SEMI
	;

formalParameters
	:	LPAREN parameterSpec (SEMI parameterSpec)* RPAREN
	;

parameterSpec
	:	("var")? identList COLON typeName
;

statement
	:	exitStatement
	|	returnStatement
	|	ifStatement
	|	loopStatement
	|	ioStatement
	|	(IDENT (LPAREN|SEMI))=> procedureCallStatement
	|	assignmentStatement
	|	(endStatement)=> endStatement
	;

assignmentStatement
	:	variableReference BECOMES expression SEMI
	;

exitStatement
	:	"exit" "when" expression
	;

procedureCallStatement
	:	IDENT (actualParameters)? SEMI
	;

actualParameters
	:	LPAREN (expression (COMMA expression)*)? RPAREN
	;

returnStatement
	:	"return" SEMI
	;

ifStatement
	:	"if" ifPart "end" "if" SEMI
	;

ifPart
	:	expression "then"
		(statement)*
		(	"elsif" ifPart
		|	"else" (statement)*
		)?
	;

loopStatement
	:	("while" expression)? "loop"
		(statement)*
		"end" "loop" SEMI
	;

endStatement
	:	"end" SEMI
	;

variableReference
	:	IDENT
		(	LBRACKET expression RBRACKET
		|	DOT IDENT
		)*
	;

ioStatement
	:	"put" LPAREN expression RPAREN SEMI
	|	"get" LPAREN variableReference RPAREN SEMI
	|	"newLine" (LPAREN RPAREN)? SEMI
	|	"skipLine" (LPAREN RPAREN)? SEMI
	;

primitiveElement
	:	constantValue
	|	variableReference
	|	LPAREN expression RPAREN
	;

booleanNegationExpression
	:	("not")* primitiveElement
	;

signExpression
	:	(PLUS|MINUS)* booleanNegationExpression
	;

multiplyingExpression
	:	signExpression ((TIMES|DIV|"mod") signExpression)*
	;

addingExpression
	:	multiplyingExpression ((PLUS|MINUS) multiplyingExpression)*
	;

relationalExpression
	:	addingExpression ((EQUALS|NOT\_EQUALS|GT|GTE|LT|LTE) addingExpression)*
	;

expression
	:	relationalExpression (("and"|"or") relationalExpression)*
	;
```

Now we need to discuss...

ANTLR Glue
----------

I like to call the code that wraps the scanner specification and grammar specification together into an ANTLR "grammar" file the "ANTLR glue." It's a bit messy at first, but if we really think about what it does, step-by-step, at least some of the mystery should disappear.

First, let's look at the overall design of a ANTLR-based parser.

Let's consider what information we already have. We have already defined how we want to group input characters into tokens in our scanner definition. We have defined the syntax of our language in our grammar definition. But what gets generated when these definitions are processed by ANTLR?

We need the startup code (a "main" function), a shell class to hold the parser definition, and references to all the fun definitions that ANTLR needs to create an executable. Without saying much more (I may later), we have the following. Note that we'll be changing some of it later when we add trees, symbols and other compiler goodies...

```antlr
header {
	...definitions that need to go at top of all generated files...
}

{
	...Stuff at the top of generated files based on this file...
	...Note that stuff in these { } will appear after anything 
	...in the header { } code...
}

class **_ParserNameGoesHere_** extends Parser;
options {
	...parser options...
}

{
	...your parser method/variable definitions...

	// a sample main

	public static void main(String[] args) {
		// Use a try/catch block for parser exceptions
		try {
			InputStream input = new FileInputStream(args[0]);
			XLLexer lexer = new XLLexer(s);
			XLRecognizer parser = new XLRecognizer(lexer);
			parser.program();
		
		} catch (Exception e) {
			System.err.println("parser exception: "+e);
			e.printStackTrace();   // so we can get stack trace		
		}
	}
}

...parser rules go here...

class _**ScannerNameGoesHere**_ extends Lexer;

options {
	...scanner options...
}

{
	...your scanner method/variable definitions...
}

...scanner rules go here...
```

Now we know everything that needs to go into our recognizer.
 

Understanding the ANTLR Compilation Process
-------------------------------------------

### The Files Involved

At some point I will expand on this a bit more to help understand how all the pieces fit together. For now, it may be a bit sketchy...

The following parts are involved when building your ANTLR parser. (Assume that you save your parser as `xl.g`):

file | description
-----|-------------
xl.g | Your grammar source file
XLParser.java | _generated by ANTLR_ \-- contains the class definition for the generated parser
XLLexer.java | _generated by ANTLR_ \-- contains the class definition for the generated scanner
XLTokenTypes.java | _generated by ANTLR_ \-- contains an interface definition that lists constant values for the token vocabulary.  Note that the name of this interafce is determined by the name of the tokenVocabulary option.

* * *

### Generating and Compiling your Parser

To build this tutorial, you can use Apache Ant. You can download Ant from [http://ant.apache.org/](http://ant.apache.org/). Ant provides an optional task that runs ANTLR. To build a grammar with this task, you add

```xml
<antlr target="src/com/javadude/xl1/xl1.g">
	<classpath path="lib/antlr.jar" />
</antlr>
```

The tutorial download includes an ANTLR.jar in its lib directory. You can choose a different ANTLR.jar by specifying it in the classpath tag.

If you've been editing your own grammar, you can tweak the "xl1" rule in build.xml to specify your grammar name. Otherwise, you can build the provided grammar, xl1.g.

To run the first build, open a command prompt, navigate to the tutorial directory, and enter

	ant xl1

What happened? It looks like we have a few grammar problems. Let's look at them.

Remember our discussion about adding `END` as a statement. Looks like it came back and bit us on the butt. But why?

The rules that are giving us trouble are rules like

```antlr
subprogramBody
	:	(basicDecl)*
		(procedureDecl)*
		"begin"
		(statement)*
		"end" IDENT
	;
```

The problem is that the `( )*` closure needs to determine when to exit. ANTLR checks to see if the "next" thing in the closure is also the first thing _after_ the closure.  This makes the closure ambiguous.  Did you really mean to stay in the loop when it sees "end" as the next token, or jump out of the loop?

Remember that we disambiguated the "end" in the `statement` rule.   This was intended to force us to use endStatement when we saw an end statement.   However, it is the wrong place to do it.  The conflict is being caused at the level of the `(statement)*` loop.

If we just move the predicates up to the `(statement)*` closures, it may work, but now we have extra predicates all over the grammar. Our goal was to fix the lookahead problem in a single place. So let's create a single place. Instead of using `(statement)*` to represent a statement list, we'll create a rule called statementList:

```antlr
statementList
	:	statementList statement
	|	// nothing
	;     
```

Some of you may be saying "but..." but I'll cut you off because I've spent too much time in yacc-land recently. Silly me coded the rule this way. Yup, left-recursion, an LL no-no, but I'm not too proud to admit I make these mistakes sometimes. ANTLR is very nice about it and reports

	error: line 197: infinite recursion to rule statementList from
	                 rule statementList     

So I shake my head in disbelief that I typed it (10 minutes before writing this) and change it to

```antlr
statementList
	:	statement statementList
	|	// nothing
	;     
```

and compile again...

Better; we're still getting the message about the `END` statement. But now the conflict is isolated to one spot. The conflict is that we can choose statement when we see "END" in the input stream, or say "we're done with the statementList" because it can be followed by "END." It is in this rule that we need to make the decision. So...

...we _could_ put a syntactic predicate around the call to `statement`. But that would be incrediby wasteful. A "guess" parse would take place for every statement. Not good. But we still need to make a guess at this point. So what do we do?

We move the `endStatement` out of the statement rule and into `statementList`:

```antlr
statementList
	:	statement  statementList
	|	(endStatement)=> endStatement statementList
	|	// nothing
	;
```

And now, the conflict disappears. We have applied the syntactic predicate to the point of the conflict.

After we compile this, we have one last conflict.

	warning: line 265: nondeterminism upon
	                   k==1:IDENT
	                   between alts 1 and 2 of block     

In `primitiveElement` we have a choice between `constantValue` and `variableReference` _which could both be an `IDENT`_! Why do we have `IDENT` in both places? Because the _semantics_, (the _meanings_) of the references are different. This should be a hint that we will need a _semantic predicate_ to properly resolve this (without mangling the grammar.) The semantic predicate we will be using will say "if the IDENT is a constant, match `constantValue`; if it's not, match `variableReference`." That type of check requires a symbol table, which we don't have yet. So, we leave this conflict in the grammar. This will produce code for `primitiveElement` like

	primitiveElement() {
		if next token is IDENT || STRING_LITERAL || INTLIT
			call constantValue
		else if next token is IDENT
			call variableReference
		else if next token is LPAREN
			match LPAREN
		call expression
			match RPAREN
		else
			report error

which means that an IDENT that could come in a `primitiveElement` context will be matched through `constantValue`. This is a bit of a problem, because things like a\[4\] cannot be matched, as they are handled by `variableReference`. Since `constantValue` only uses a single IDENT by itself, we won't lose anything by swapping it with `variableReference`, so swap 'em!

To compile the new version of the grammar, you can run

	ant xl2

* * *

### Building the java code and running tests.

If you run ant without any options, it will perform a full build on the new xl2.g file and run some sample tests against it.

* * *

### Wrapup

This wraps up the recognizer section of this tutorial. I hope it's been helpful at some level for you. As always, please let me know your comments (good or bad -- helps the ego ya know) at [scott@javadude.com](mailto:scott@javadude.com).

