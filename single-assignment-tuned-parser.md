#Tuned Parsing With Infinite Recursive Single Assignment

To tune a parser, we must know where we are in the call stack. This is tracked in the typical parser anyway, for clear reasons, but here we need it to be very cheap.

Here's a grammar:

```

foo =  bar | baz | bux

bar = "bar" | "car"

baz = baz | "baz"

bux = "bux"

```

How can we rewrite it?

```

foo = foo_3 | foo_5 | foo_7

foo_3 = bar

foo_5 = baz

foo_7 = bux

bar = bar_3 | bar_5

bar_3 = "bar"

bar_5 = "car"

baz = baz_3 | baz_5

baz_3 = baz_??

baz_5 = "baz"

bux = "bux"

```

We tried single assignment form, and we have a recursive assignment. Hence, the primes. `baz_??` could become `baz_n*3` where n is the number of times we've visited the rule. Or `baz_2*n*3`. 

This obscures the logic, I think, since everything becomes foo_something under the hood. I think. The point is to be able to do some fast math to figure out which weight applies to reasoning the next choice of rule, which matters in intricate, ambiguous, possibly input-generated grammar situations. 
