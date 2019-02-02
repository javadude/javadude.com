---
title: Converting a Grammar from LALR to LL
url: /articles/lalrtoll.html

categories:
- Articles

date: "2000-03-04"

description: How to convert a grammar from LALR to LL.

tags:
- parsing

---

Converting from LALR to LL can be challenging... Hopefully this article will make it a weeeeee bit simpler...

<!--more-->

This is from a post of mine in a thread about automated tools to convert from LALR grammars to LL grammars... NOTE: These are fairly general (but pointed to PCCTS 1.33 -- note that PCCTS 2.0 does not implement syntactic predicates with setjmp/longjmp)

Toolwise, I think you're pretty much SOL. But there are several common algorithms available that you could probably use to write a simple tool.

HOWEVER, I don't think you'd really want the code they'd generate, and hand-written ANTLR rules using its EBNF notation are much more maintainable.

There are two "big" issues to deal with in the conversion (and some minor headaches as well):

## Problem: Common Prefixes
    
An LL(k) parser cannot choose between two rules that have the same k-token lookahead. For example, if k=1 (LL(1)), the following is ambiguous:
    
	a
		:	A B C
		|	A D E
		;   
    
The above ambiguity will disappear if you specify k=2 -- the parser will look at the "B" or "D" after the "A" to determine which way to go. But, this can be inefficient, as the parser must look 2 tokens ahead in every prediction expression!
    
An LALR(1) parser doesn't have trouble with this because it is essentially "trying all the alts at once" and deciding which to use when it has enough info.
    
An LL(k) parser predicts which alt to take based on the next k tokens it sees in the input stream.
    
So what to do in a general case? There are a few possibilities:

### Up the value of k (lookahead)
This may work, but will cause a performance hit and only covers an certain set of cases

### Left factor

In the above example (in ANTLR), left-factoring this would result in:
        
	a
		:	A (B C | D E)
		;
        
Now the parser just matches "A", then makes another decision on which alternative based on "is the next token B or D?" Note that this could also be written
        
	a
		:	A a_rest
		;
	a_rest
		:	B C
		|	D E
		;
        
if you prefer. However, this creates another function call (ANTLR generates recursive-descent parsers.) Use your judgement as to when to create separate rules as opposed to subrules. Ask "is it readable? Does it make more sense to keep stuff in one rule?" and so on...
        
### Add syntactic predicates

There may be cases when it is very difficult/impossible to left factor, or you really need "infinite lookahead" (C++ is a great example of this with decl vs. expr) 

Syntactic predicates are a feature of ANTLR that says "if my first k tokens match, try me -- if I work, use me. Otherwise, try the next possibility"
        
Say you may have a silly rule like
        
	a
		:	A A A A A A B
		|	(A)+ B
		;
        
Basically, we want to do a certain action if we see 6 A's then a B. Any other number of A's followed by a B does something else. (Yes -- you could just count the A's in the (A)+ part of the rule, but suppose I was stubborn and insisted you follow me...)
        
This rule would need 7 token lookahead, or some really nasty splitting up that would make the meaning unclear (or, don't say it, a counter in the (A)+...) By changing the rule to
        
	a
		:	(A A A A A A B)?
		|	(A)+ B
		;
        
You tell ANTLR to construct a parser that will "try" the first alt, and if it works, use it. Otherwise, \_don't\_ display a message and try the next alt and so on. Note that the "guess" it is taking doesn't execute action code, and if the "guess" says use that rule, it needs to re-run the match to run the actions.
        
Syntactic predicates are your friend, but they are costly if overused. Also, be warned that they are implemented using setjmp/longjmp, which could cause problems if you create objects and need them destructed -- be careful!
        
   
## Problem: Left Recursion
    
This one requires a bit of thought, but usually not too much.
    
Suppose we have a rule
    
	a
		:	a B C
		|	//nothing
		;
    
Think about what that means -- any number of "B C" pairs. Just rewrite it as
    
	a
		:	(B C)*
		;
    
be careful as to the use of \* vs + -- in the above, a can be empty, so \* was used. this could also be written as a tail-recursive rule, but the first method is more efficient as it just generates a match loop, not extra function calls. (Use the (...)\* and (...)+ constructs when possible to avoid recursive calls -- they save on performance!)
    
Left recursion is also prevalent in expression rules. Things like
    
	expr
		:	expr "+" expr
		|	expr "-" expr
		|	expr "\*" expr
		;
    
and so on.
    
To rewrite the expression rules in LL(k) form, you'll need to break them up by the precedence of the operators. Start with a rule like "primary" that just lists the "atomic" elements of an expression, like variable, constant and (expr):
    
	primary
		:	variable
		|	constant
		|	LPAREN expr RPAREN
		;
    
then work your way _down_ the precedence table, starting with the highest-level precedence operators, like NOT:
    
	unary_expr
		:	(unary_operator)* primary
		;
    
then go to the next level (suppose it's multiplicative operators)
    
	mult_expr
		:	unary_expr (mult_op unary_expr)*
		;
    
then the next level
    
	add_expr
		:	mult_expr (add_op mult_expr)*
    
and so on. The idea is that things at each level of the precedence table reference the things at the next higher precedence level, _possibly_ adding their own operators. In general
    
	level_n_expr
		:	level_n-1_expr (level_n_op level_n-1_expr)*
		;
    
making sure to use `(...)*` and NOT `(...)+`
    
Most (if not all) yacc grammars can be converted into ANTLR grammars, but I suggest you use thought to do it rather than mechanical force (unless there's a mechanical force that smart enough, which I doubt.) It doesn't take too long to do it, either. A few other things to remember about ANTLR:

Action code can be placed _anywhere_ in rules without adding conflicts (although placing them at the start of a rule makes it and "init action" rather than a normal action -- see the docs

You can pass/return values to/from other rules as parameters to the rules. This is a GREAT feature that you just can't do in yacc due to the nature of LALR parsing try to use label _names_ instead of label _numbers_ (as in yacc) this'll save tons of headache when you need to make a mod like adding a rule reference to a rule.

	yacc :    a: B C {$$ \= do($1, $2);}

	ANTLR:    a: b:B c:C <<$0 \= do($b, $c);>>

(keep in mind that you can only label terminals -- if you need a result from a subrule, pass it back as a return value)

Good luck! 
