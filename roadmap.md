#Roadmap

GGG is a format for representing grammars. It strongly implies a reference implementation that exposes itself as a command-line tool and as an ANSI C library. Let's call that gg, and of course, libgg. 

##gg

Any performant parser must be very close to the hardware. We are providing an abstraction, and can't afford to employ them as a result. Our options are C and Lisp. Guess which one I'd rather use? #ifdef or defmacro.... decisions, decisions... 

###nitty gritties

The algorithm of gg is roughly as so: Read a .ggg source file, format it, parse the formatted version, perform inference, construct a parsing engine. 

Inference is conceptually straightforward. We want to construct as little parser as possible. If a rule is a simple literal like ` sixteen = 0x10 `, we want it to be as close to `if ( first_two_bytes == 16 ) { return 1 ; } ` as it can be. 

As good a place to note as any: parsing over unstructured data is the degenerate use case. gg needs to be able to parse several times over already-structured data, and keep track of the parses orthogonally: We might want to treat a single file as Markdown and as line-based source, while treating some of the contents as e.g. Python. A .jpg might be a part of a .pdf, and so on. We might even want a parser with a separate tokeniser, for some reason. Tradition?

So we simply do analysis to find the level each rule operates at, and build the parser accordingly. We have to do this anyway, to properly format the input. We should be able to get it fast enough that a regular-only set of rules benchmarks similarly to the good regexp engines. 

The PEG and CFG levels are conceptually built on parser combinators. The only literal implementation of this I even vaguely understand uses a graph-structured-stack to store the continuations and their associated memos. 

