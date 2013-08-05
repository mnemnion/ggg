##Tuned Alternates in GLL

GLL revolves around `alt`, which looks through alternates. It's a fifty-fifty, alternating kind of thing, at least conceptually. 

The thing is, one branch might be 90% likely to succeed for a given class of input, the other branch, 10%. We could specify that in a grammar but it would be ugly, and worse, it would imply an implementation.

A better approach, if data is available, is to allow the grammar to tune itself by noting which alternates tend to succeed from any given context. There are many data tricks which can be used to rapidly dispatch in a more granular way, like consuming 6 bits of entropy and masking it against a ceiling. 

Markov babbling the grammar and running it in reverse will produce some useful biases, but they need to be demarcated so they can be weighed against actual validating input and eventually discarded.
