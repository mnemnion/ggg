##Tuned Alternates in GLL

GLL revolves around `alt`, which looks through alternates. It's a fifty-fifty, alternating kind of thing, at least conceptually. 

The thing is, one branch might be 90% likely to succeed for a given class of input, the other branch, 10%. We could specify that in a grammar but it would be ugly, and worse, it would imply an implementation.

A better approach, if data is available, is to allow the grammar to tune itself by noting which alternates tend to succeed from any given context. There are many data tricks which can be used to rapidly dispatch in a more granular way, like consuming 6 bits of entropy and masking it against a ceiling. 

Markov babbling the grammar and running it in reverse will produce some useful biases, but they need to be demarcated so they can be weighed against actual validating input and eventually discarded.

We might call this Bayesian Extended GLL, if we ever get it working. 

A practical example of tuning might be a symbol category, with three regular rules that match Latin, Greek and Cyrillic characters. In the wild, it's like that that we might find all Latin, Latin with a little Greek, all Greek, all Cyrillic, and a few mixes. Weighting the `alt`s accordingly would speed things up: if you're finding Greek, stick with Greek as the first choice for awhile. Even simply trying the first regular expression from the last rule match is a good cache for many cases. 

### What it might look like

Let's take a simple grammar:

```text

A : B | C

B : d | e

C : f | e
```

where `#{A,B,C}` are nodes and `#{d,e,f}` are leaves. `e` may be reached by `A -> B -> e` or `A -> C - e`. 

Let's train our parser. What happens if we feed two `f`s in a row?

The parser has to chose a combinator to consume a string. It flips a coin, gets `C`, flips a coin again, comes up lucky `e`. `A>C>f` is promoted from 128 to 128+64. I need to chew up some Bayesian statistics to get this stuff down, in the meantime I'll be charmingly naive. 

How much should we promote the `C` path? It succeeded too. I say we bitshift for each success, so `A>C` earns 32 for this.

Note that we weight entire paths, not single productions. This is a byte of overhead per path through the grammar, of which there can be many. Grammars at least have the virtue of being acyclic. 

We have impose one boundary: we break inference on namespaces, which is probably what's wanted. If we're parsing Python inside Markdown, the inferences for general Python should suffice, we needn't keep a separate set of weights for the Python-inside-Markdown chains. This lets us train up little languages that will be very fast for their purpose and embed them in other contexts safely. 

