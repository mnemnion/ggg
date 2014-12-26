#Parallel Parsing


Using a GGG style grammar, it should be possible to parse in parallel using a SIMD architecture.

Steps:

##Literal Detection

First we search the whole string in parallel for each possible literal match. These are lifted.

## Regular elimination.

We take the lifted set and apply regular elimination to it. There are contexts where a literal token is collected by a regular rule, rather than by the literal rule which applies. An example is within strings. Strings begin and end with a literal, making them particularly easy to eliminate. 

Note that this is using a restricted version of the regular rule, when possible. For strings, we merely have to check for string escaping, and can eliminate all literals between pairs. The engine knows which literals it can collect, which ones belong to regular expressions, and how to validate that the literals within the expression are well-formed. 

All we do is eliminate literals here, and promote posslbe regular matches.  Note also that this step runs in parallel, requires a separate array for kernels to communicate, and may involve a few (quadratic? to some small number) backtracks across the owned stripe of the string. String escaping remains a good example, we have to cross communicate to figure out what's outside and what's inside, and might be wrong at first, detect a `\"` and have to refix. A pathological collection of strings might backtrack, I hypothesize, by the number of escapes detected, though it may be possible to make this stage single-pass through sheer cleverness. 

This is the step that makes this approach fast, if fast indeed it is. The critical step is converting a regular expression into a negative expression, which is never slower than the regex and is often faster. I am lacking a general transformation for this step at present. 

## Ordering

Many grammars will specify an ordering for some or all matches. A very common and useful subset of this are the balanced literals, such as the classic `"(" form ")"` pairing in a Lisp. We have a sparse array of possible literal matches, and a sparse array of possible regular matches. It is quite possible that the regulars overlap, and even possible for the literals, if a language had say keywords `to`, `toward` and `ward` and no meaningful whitespace whatsoever. 

We use the orderings rule as the next level of constraint on eliminating regular matches. We're starting to have a structured tree, with relatively few ambiguities remaining. If the parse is ambiguous, allowing multiple orderings, we do the least amount of work to satisfy. This can still become quadratic if not handled very carefully. I think we're at less risk though, because we know how long the ?regulars are and can aim for greedy ones. 

We now take a parser, specially designed to eliminate any logical ambiguities we haven't eliminated, and take a linear pass over the regions marked as not-yet-ordered. Those resolved, we have our parse. Unresolved, we have a parse error, one from which we are able to construct the possibilities that the engine is searching for. Maybe. The parse is laying around broken into pieces when it fails. "regular expression bleh does not allow token ';'" is a crappy parse error. 
