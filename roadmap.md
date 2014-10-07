#Roadmap

GGG is a format for representing grammars. It strongly implies a reference implementation that exposes itself as a command-line tool and as an ANSI C library. Let's call that gg, and of course, libgg. 

##gg

Any performant parser must be very close to the hardware. We are providing an abstraction, and can't afford to employ them as a result. Our options are C and Lisp. Guess which one I'd rather use? #ifdef or defmacro.... decisions, decisions... 

###nitty gritties

The algorithm of gg is roughly as so: Read a .ggg source file, format it, parse the formatted version, perform inference, construct a parsing engine. 


