Generalized Generative Grammar
===
This is an introduction to a representation format, **Generalized Generative Grammar** or `.ggg`.

GGG format is a text encoding of grammars, designed to be general, and focused around binary data. It is designed to scale to task, introduce no unintentional ambiguities, and be computable in practice. The main objective is to provide a tool for validating the type of data, and a reference framework for manipulating the resulting tree. 

Central to the concept is the notion of recognition layers, which have an absolute precedence: literal, regular, deterministic, context free, and functional. Each is a proper superset, and many grammars do not require all layers: for this and other reasons, no algorithm is specified for GGG output. 

The upper bound of the context free layer is provided by the GLL algorithm, which is free of arbitrary restrictions on parse construction. Adding the functional layer is... tricky, and is expected to take awhile. 

Within a recognition context, each possible rule is tried. The precise behavior is determined by the rule level: literals return based on recognition, regulars return based on greed, PEGs return based on order and CFGs and functions return based on success. A success either terminates and returns the rule, or the parser may continue and return all possible trees in the case of ambiguity. The parser *must* provide both behaviors to be compliant, if it provides non-deterministic grammar. 

GGG is an input format. The output is implementation specific: the reference target is an executable program which takes data as input and exits successfully if the data is valid to the specified form. 

Implementations are not required to be fully compliant; they may implement only particular subsets of GGG grammar, the first G being Generalized. They are required to parse the full formatted GGG specification, but may refuse to execute if desired. 

##Rationale

In 19xx, [Niklaus Wirth](http://addme.com) published a paper, which proposed a standardized format for extending BNF format. This grammar format, while excellent, was a) designed around strings and b) predates the GLL algorithm. It is also purely declarative, allowing no execution semantics to be used as part of recognition. 

The [GLL algorithm](http://addme.com), published in 2010, is a towering achievement. The underlying representation structure, a [Graph-Structured Stack](http://en.wikipedia.org/wiki/Graph-structured_stack), is well suited to parallel execution and can conceivably be extended to support a restricted but useful subset of computations. 

The human mind and our current computational machines are optimized differently. The difference in AI achievement between chess and Go is partially one of computational complexity, but partly that the largely static nature of Go favors our finely-honed pattern recognition ability. 

Grammars are a rare meeting of the minds, wherein language, made sufficiently precise, takes on the mathematical character which our machines require. GGG is an EBNF with a binary focused, mix-and-match philosophy, empowered by modern algorithms to specify formats and types in a single, largely declarative, human readable syntax. 


##Encoding.

The format of GGG is unusual: It is to be encoded in Latin-1, ISO 8859-1. I strongly believe that one byte is exactly enough to specify a standard of such importance as GGG. 127 code points is kind of light, though. Latin-1 is exactly as well defined as ASCII, in practice well supported: Emacs, Sublime/TM, Vim have no problem editing Latin-1, and I consider that conclusive. 

Therefore, GGG format will be *specified* in Latin-1. It may be easier to markup a file in UTF-8, if so, Unix systems provide `iconv -f UTF-8 -t ISO-8859-1 filename.txt` for conversion before parsing. 

It is currently a pain in the ass to edit Latin-1 in many contexts. It should be viewed as an internal representation format, to be worked with directly when convenient. A practical GGG implementation will go the extra step to convert from various formats into Latin-1, and pass Latin-1 strings (which *must* be produced in all communications from a compliant GGG implementation) through a converter of some sort to produce a representation convenient to the environment. It might be UTF-8, it might be UTF-16, and the difference is not trivial.

Additionally, a pure ASCII dialect will be available. It's important to stress that this is an input format, and must never be implemented directly or I may just hunt you down. 

The dichotomy begins early: octopress's Markdown parser takes only Unicode, so I will show off these useful codepoints in that format. Latin 1 has such gems as `«`, `»`, `·`, `±`, `¿`, and `¦`. It also has `à`, `á`, `â` and `ä` for all five vowels, including capitals, plus `¨` and `´`, joining `` ` `` and `^` from ASCII to make a full set of bare corresponding marks. 

Given that a rule may be literal, regular, PEG, CFG, or functional, having five levels of markup for names seems just right. Most words have vowels, and if not the bare mark can be prepended. 

We don't plan to go nuts with the extended characters. It's nice to be able to just type, conversationally, and our keyboards support that better in general for the lower 127. Future systems will not be so tightly bound to physical keyboards with a single glyph sprayed onto the surface, as that input format has a limited shelf life. The future of input is through a combination of touchscreen and something like the [Optimus](http://www.artlebedev.com/everything/optimus/).

GGG insists on not using ASCII or Unicode as a representation format. Digraphs have a wearying effect on the brain, on the one hand, and on the other, there are few faster operations than bitmasking bytes. There may be none in fact, it is very, very fast and very, very easy and stupidly reliable. A bitwise XOR for a clean zero. GLL grammar is intentionally as regular (in the sense of lexical) as possible, because GLL grammars, in readable human format, should be treatable as fast executable code. 

Let's write some!

##Generalities

GGG is a tool for representing data. It is written by someone who considers Unicode a poor level of aggregation, as part of a tool set for making it practical to not have to use it. It is not designed around the paradigm of strings, although it makes a few concessions to it. 

GGGs input format is fairly permissive, but *must* be passed through a formatter to standardize it before parsing. This intermediate representation *should* be presented to the user, or may be used to generate warnings. It is fine to write loose GGG, but please publish the tight stuff. 

GGG will be specified in GGG, before which time it is not even vaguely done. GGG in GGG will not contain insignificant characters, thereby enforcing whitespacing and all other matters of format. The formatter/parser combination is not optional, a GGG implementation *must* be permissive in what it accepts. 

This level of validation will require the functional syntax, and will allow the GGG grammar as-written to serve as a unit test for any GGG formatter. It will only be published as an extension of a more restricted version of the syntax with less prominent whitespacing conventions. 

A rule name definition must be followed by one of `=`, `:=`, or `:`. The three are equivalent and the formatter will change all to `=`. Rules are terminated with `;`, which are added when not present. Semicolon insertion may feel scary; fear not, GGG is quite regular stuff.

The character `@` represents epsilon. Epsilon matches no input, returning successfully. It is allowed in any rule. The string `epsilon` and variants have no particular meaning. `¬` is a rule which always fails. In functions, they behave like true and false, respectively. 

Rules are scoped and namespaced. The top level rule is the namespace, if scope is not otherwise specified. Please choose something distinctive; it is good form for the file name to match the top level rule, if the file metaphor is in play. 

Scope is specified explicitly with `{` and `}`, which enclose rules. The first rule after `{` is always the namespace. All rules in a particular scope *must* be reached by the top level rule: all valid GGG constructions are a tree. As many scopes as desired may be provided, and scopes nest. Rule references follow a `namespace.rule` format for single scoping. A nested scope takes the form `namespace.rule.subrule`. 

Note that every other rule, other than the top rule, is in a single level of scope. The names must be distinctive and there is only one level of subclassing per scope. This is in contract to the logical structure of rules, which can become deeply and arbitrarily co nested. 

##Literals

Literal rule names *must* start with an ASCII letter, and may continue with any ASCII letter or digit, a `-`, or an `_`. By convention, dashes are used between words, and the underscore is used before a number meant as an index. Two underscores may used to indicate a generated symbol. This may develop the force of law; it is wise to follow this convention.

Literal values, all values in fact, are always specified in big-endian fashion. This is *not* meant to imply an implementation, on the contrary in fact. Compliant implementations are expected to understand their architecture and do literal value recognition in the correct way for it. This is surely not too much to ask.

The right side contains a literal representation of data. These literals may be specified in recognizable forms: `0b011010101`, `0x234ABG`, and `£QWxsIFlvdXIgQmFzZSBBcmUgU2l4dHkgRm91cg==`. The last is intended to represent MIME Base64. If there is a conventional prefix, I don't know it. 

Let's steer away from things like `1.5`, though. That's not as literal as it looks. Similarly, you can't just up and say `25` in a literal. How many bits is `25`? Nope. You get binary, hex, and base-64.

These literals are **conservative** in all cases. A single numeral in binary specifies one bit, hexidecimal specifies 4, and base-64 specifies six. Naturally, null values in the highest significance are valid: we are doing recognition, not arithmetic.

`0%` is reserved for representing literal floating point data, if we can design a syntax that is both readable and bit-identical in all cases. 

Literal data can be anything, but forsooth it is often strings, and often those strings are ASCII. We provide some sugar for this eventuality. `«a literal string»` may contain anything printable in the lower 127, including line feeds and tabs, because I'm feeling generous. It would be perverse, I feel, to provide the rest of Latin-1 in what is already a weird (if eminently well established) way to map numbers to glyphs on screen. For one thing, a string, as defined here, cannot contain its delimiting characters. For how many languages, dear Reader, can we say the same?

Perverse doesn't begin to describe Unicode. We could allow some demonic variant of GGG in which, while a Unicode is chosen, all characters outside of `«` and `»` are validated for Latinitude. It would need to be clearly marked, as reeking of gin. 

Pardon my laconic sense of humour. You are in fact quite welcome to embed any valid UTF-8 character except the closing delimiter `»` into your literal string. The character we're looking for is 187, or `10111011` in binary. This is not a valid leading byte in UTF-8, which is very fast to parse through if you're just validating. So we'll validate as UTF-8 inside a literal string. It will look very weird in Latin-1, I promise you.

Oh and I promised no need for string escaping since this is a byte-at-a-time parser with zero lookahead. `»` is a perfectly cromulent UTF-8 character, while the byte 187 is not and never shall it be, for it begins with an 10, and is a continuation, verily and always. Aren't you happy I think about these things? 

Let us note: `«a literal string»` is a number, and each character is a byte. ASCII yes, Latin-1, yes, UTF-8, yes, UTF-16, **no** they are **not** the same **at all**. 

No, I truly feel that Unicode code points have unique 26-character-range-limited names for a reason. Let's use those, shall we? It shouldn't take more than several months diligent work to type it all up. We have `/`, we can specify all the different encodings in a single pass! 😈 SMILING FACE WITH HORNS U+1F608 (U+D83D U+DE08), UTF-8: F0 9F 98 88. As my Mac would put it. 

Hey if you don't read my blog with Firefox you might be missing some color. Just sayin'.

The literal representations may be embedded directly, which is why rule names can't start with numbers or the `£` sign, among other restrictions. This is often convenient in regular rules, and sometimes useful/readable elsewhere, though it is a bad habit at higher levels of abstraction. 

Note that any single ASCII character is a perfectly valid literal rule name. Digits are not; a convention would be to name them d0..d9. This allows for readable definitions of (unaccented Latin alphabet based) character encodings, which will prove handy when designing saner readable formats for executable code. 

More than one literal value may be provided on the right side of a literal rule. This is implicit concatenation, and the rule matches only to all values provided, in the order provided. 

##Regular Rules

Regular rules follow the logic of a lexer or a regular expression engine, but on binary data. There is less sugar on the one hand, and the need to specify bit fields on the other. It's a different flavor. 

Regular rule names follow the constraints of literal rule names with this addition: They *must* contain one of `` ÀÈÌÒÙàèìòù` ``. If the rule is regular on the right, and does not contain one of these characters, the first vowel in the rule name is replaced by the substitute, by the formatter. This is repeated after each dash, leaving wèll-fòrmatted names with a consistent look. If there is no vowel in the rule name, `` ` `` is prepended. Canonically this is the only place `` ` `` is found in a rule name.

A linguist should parse these as tone marks, of a sort. Indeed they are; the instinctive funny pronunciations are part of the charm. 

The simplest regular rule consists of multiple literal values separated by `|`, the regular option character. It is an error to find it in anything but a regular rule. Its behavior is greedy: a regular rule will return the most input it can, given the options it has to work with.  `«foobar» | «foo»` will return `foobar` if it's available.

The precedence of `|` is lower than that of concatenation, so `«foo» «bar» | «baz»` will match either `foobar` or `baz`. Parentheses play their usual role in forcing precedence, throughout GGG: `«foo» («bar» | «baz»)` will match either `foobar` or `foobaz`, but not `foo`.

We want to specify ranges and wildcards and repeaters and optionals, because this is what a regular language needs. We must be careful, because we're consuming binary. The range `0x00..0xAF` consumes 8 bits, if you want 16, it's `0x0000..0x00AF` you're looking for. Range is a binary operation of highest precedence. It is an error to place whitespace between the two values, which must be literal, of the same encoding, and such that the left value is less than the right value. 

Wildcards start with `~`, and come in two flavors, bit and byte, which are `,` and `#` respectively. `~##,,,` matches 19 bits of whatever, and you can stick in some fields so `~##,1,` matches 19 bits if the 18th bit is a 1. Binary only, concatenation does most of the heavy lifting here. As mentioned in literals, representation in GGG is big-endian, and no method to specify a little-endian rule is ever provided. Certainly you may write a preformatting script if you need one. 

`!` before a rule matches a number of bits if they do not fit the rule. The match will consume however many bits were needed to verify, so handle with care: this rule is easiest to understand with matches of fixed width. `!~#0000,,,,` will match 16 bits if bits 9-12 are not 0. `!0xFFFFFF` will match three bytes that aren't white. 

`+` means at least one of previous, `*` means zero or more of previous, and `±` means zero or one of previous, but not more than one. This is solid ground, I reckon. `+,*,±` are unary operators in post position, with lower precedence than range and higher precedence than `|` (and hence, higher than concatenation).

`+` and `*` are greedy by default. The thrifty/lazy versions are `\+` and `\*`. `~#\+ 0xFF` will consume bytes up to and including a `0xFF` while `~#+ 0xFF` will eat everything you give it a byte at a time and then fail, as there will be no bytes left that could equal `0xFF`.

There is a multiply operator which matches exactly the number of patterns specified. Although there is an `×` character in the set, `x` is a valid rule name, and confusion is unfortunate. `§` is chosen as at least somewhat mnemonic; there are n sections in `§n`. To illustrate: `«foo»§3` matches exactly and only `foofoofoo`. The precedence is the same as the unary operators, and `n` must be a decimal integer greater than one. The range operator may be used, so that `«foo»§0..3` would match up to three `foo` in a row, or none. It is an error for the value on the right to be less than that on the left.

Regular rules can also be concatenations of other regular rules, with a key restriction: no recursion of any sort is allowed. Recursion promotes your rule to PEG and will change your accent mark accordingly in the formatting pass. This *should* be treated as a non-fatal error, rather than merely warned against. 

Without recursion, concatenation is merely a convenience against writing long, hard to read regular expressions. It also allows for better composability, encouraging pattern reuse.

As a sample of the format, we define the first 32 bits of an IPv4 header thus:

```
ÌPv4-fìrst-32-bìts = version ÌHL `DSCP ÈCP tòtal-lèngth ;                    

version = 0b0100 ; 

ÌHL = 0b0101..0b1111 ;  

`DCSP = ~,,,,,, ; //      

ÈCP = ~,, ;
     
tòtal-lèngth := ~#§2  ;
``` 

In one line:

```
ÌPv4-fìrst-32-bìts = 0b0100 0b0101..0b1111 ~,,,,,, ~,, ~## ;
```

I trust this is fairly readable. `~##` is a more straightforward version of `~#§2`. As a reminder, a subrule may be written as, e.g, `ÌPv4-fìrst-32-bìts.version`. There will be a mechanism for importing and renaming namespaces; the notation will probably use the function braces `[ ]` and there's little sense in providing it until there's an implementation that can use it.  

##Whitespace and Separators Sugar

There is a sugared form for whitespace and other padding. A rule named `_` is treated as whitespace, and must be a regular rule. It *may* be hidden by an implementation, but needn't be; a minimally compiant GGG implementation is pass/fail only and does nothing to the bitstream other than validate it. It behaves like any other regular rule, though it is literal if literally defined.

`·` is a rule which may be defined as a separator. Unlike any other rule, this ends up in the grammar even if not referenced. The conventional use is to define a newline character for error reporting; it may be used for any regular pattern that demarcates boundaries, whether distinct from the logic of the grammar or not. 

##Parsing Expression Grammar Rules

[Parsing Expression Grammars](http://en.wikipedia.org/wiki/Parsing_expression_grammar) are a fairly general class of grammar. The key limitation is that they are deterministic: choices are always evaluated in a particular order, and a grammar defined only in PEG terms cannot be ambigous. It will always parse a given input in exactly one way.

PEG rules names are similar to regular rule names, except the required character set is `ÁÉÍÓÚáéíóú´`. There is no particular mnemonic to the choice of accent. They come in a specific order in Latin-1, and that order is what we use. As always, if your rule is PEG, the formatter will accent the name accordingly. There is no particular reason to type the accents on a definition when composing, and the invocation of a rule must be accented only if there are multiple instances of the name (a regular fòo and a peg fóo, for example).

The accent marks are a key part of the system: they allow one to determine at a glance the precedence of any rule, anywhere within a grammar. GGG has an exact but somewhat convoluted expectation for what will happen at any given point in a parse, and the key to understanding the precedence is to know which kind of rule you're dealing with. 

Rules must be either PEG or CFG, for a variety of reasons. Precedence, for one, paralellization potential for another, ambiguity for a third, and if this is not enough, several operators are overloaded. This is a readability decision, primarily, in that `*` and `+` both have well understood context-free meanings, ambiguity and all. 

Literals may be embedded, and there are times that this might be useful, but do so carefully. Any literal rule that is implicitly defined in a PEG or higher rule is treated for precedence as though it were literally defined directly beneath the rule that creates it. This can bend comprehension of precedence in a CFG context, so handle with care. The formatter may make this rule explicit with a gensym. I haven't decided. 

The basic PEG operator is `/`, and expresses deterministic choice. A PEG will go through the options given, in the order given, and return a match without exploring further. `?` is the lookahead operator, unary and prefix, followed by a literal or regular rule. This rule is verified via lookahead, but not consumed; if it passes, the next rule may be checked against the bitstream. Negative lookahead is `¿`, also unary and prefix, the rule provided must fail, and bits are not consumed. 

`±*+§` all work as in regular rules, as do the lazy variants `\+` and `\*`. PEGs are very much like regular expressions generalized to support explicit lookahead, recursion, and ordered choice. `|` is disallowed to promote separation between regular and PEG rules, and to avoid having to make an arbitrary decision about the precedence of `|` vs `/`.

Therefore, a PEG rule which contains no recursion and doesn't use any of `/¿?` is actually a regular rule, and is reaccented accordingly by the formatter.

A note about this decision: It would be easy, and tempting, to allow mix and match among `|`, `/`, and `¦`, which we shall soon meet. This would be practicable by requiring disambiguating parantheses. I won't, because that means generated rules, which means a significantly different intermediate representation. Just name the rules. Be creative!

An important point about "PEG" syntax: GGG allows all forms of recursion, including indirect left, which is not supported by all PEG parsers. We are implementation agnostic; it is best to avoid left recursion in PEG rules, if writing a grammar that is explictly written to be useable by such a parser.  

GLL is the reference algorithm for non-functional GGG. [Instaparse](https://github.com/Engelberg/instaparse), which implements GLL, allows for flexible mixing of PEG and CFG syntax, as well as all forms of recursion. This suggests to me that the more restrictive syntax of GGG can be implemented performatively using the same algorithm. Functional GGG could plausibly implement its operations on top of the existing stack tree, as the functions have no way to 'call out'. 

##Context-Free Grammar Rules

Explicit order is not always helpful, and ambiguity can be the most natural way to define something. GGG is concerned primarily with validation, and it is often easy to write a grammar that can validate a given atom in a number of ways. 

CFG rule names use `ÂÊÎÔǓàêîôû^` as the disambiguating characters. The key operator is `¦`, the unordered choice operator: any of the options may succeed, in any order, and a compliant implementation *must* be able to provide all possible parses if requested. The broken pipe indicates that this may be run in parallel. `+*±§` are all available, but `\+` and `\*` are not. This is because in a CFG rule, the laziness or thrift of `+` and `*` cannot be assumed. The parser takes "(zero¦one) or more" literally, and can stop munching whenever it likes, as long as the overall rule is fulfilled. Again, it *must* be able to provide all possible parses on command, even if it takes awhile. 

It should be possible to support `&` as a binary operator with the same precedence as `¦`. This would return true only if both rules return true and provide the same piece of the data stream. I haven't seen this in an existing GLL, so it should be considered reserved for now. If provided, it could conceivably be supported at the regular and PEG levels also. They key thing to grasp would be that both rules must return true and move the cursor the same number of bits forward. It would have identical precedence to `¦`, or `/` and `|` in PEG and regular contexts respectively; parenthesis strongly encouraged for the sake of clarity. They are all binary, left associative operators. 

`?` and `¿` may be used in CFG rules, but `/` may not, to promote separation between CFG and PEG rules, as well as to avoid precendence assignment between `/` and `¦`. It is better form not to mix lookahead and CFG syntax, for two reasons: it may imply to the reader a greedy `+` or `*`, and there are many classes of grammar where only CFG or PEG rules, but not both, will be provided. These may be efficiently implemented using their respective techniques: using only enough algorithm for the job is a goal of GGG. CFG rules that use lookahead are contaminated, requiring PEG techniques for realization. This is allowed, but not encouraged. 

##Function Rules

We are not defining a syntax for functions at present. It is likely they will use `[` and `]` where Lisp uses parentheses, in order to avoid collision with `( )`, used in the usual mathematical fashion, and `{ }`, which provides scoping. Overloading of symbols within a functional context is an inevitability, making it all the more important that the outermost enclosure be clean. 

Function rule names are disambiguated with the character set `ÄËÏÖÜäëïöü¨`. In keeping with Unicode, we call this mark a diaresis, not an umlaut. The others are grave, acute, and circumflex respectively. we use `` ` `` for our bare grave accent, and `` ^ `` for our circumflex, but we call them grave and circumflex, not backtick and caret. Thanks.

The functions should be deliberately restricted. [Total Functional Programming](http://en.wikipedia.org/wiki/Total_functional_programming) seems a good approach, as GGG is designed to validate finite data, or infinite data one tree at a time. None of our other rules fail to terminate, and allowing our functions to do so is bad form. 

These functions are proper functions, that is, they do not transform the input although they do conceptually consume it. A function returns two values: true or false, and the amount of the bitstream consumed by a true return. A function says "yes, we have matched the rule. Here is the data which does so". Or it says "Nope".

A function is closed over itself and may maintain state internally between invocations. There is no mechanism for a function to return this state, or share it; though functions may invoke other functions, as well as any other rule, any given function may receive only the cursor and the digest, and return true, false, and bits consumed (which may be zero for a true and must be zero for a false).

This aids parallelization by simply forbidding shared state. No transactional model is needed, and fewer architectural assumptions are made. The functions may be retained after a GGG implementation exits, and inspected for their internal contents, but like any implementation detail that isn't mandatory, this is optional. 

A function declaration, but not a function call, may have arguments. Those arguments must be provided by the implementation, or the function is required to fail whenever invoked. An example use: you may validate a public key signature by providing the public key in question as an argument to the function. This must be done at runtime by necessity and allows for a common type of validation to be performed. If the function is not provided with the public key, it must perforce fail to validate. Another rule might easily validate the public key for format, as an alternative parse. 

The language used to specify functions must perforce be a complete one, within the constraints of TFP. It should quite suffice to implement GGG, and implement it well. It will also be specifiable, and specified, in GGG, using only the literal, regular, and PEG layers.

The reference to the digest points to the major reason why functional GGG is undefined at present: any useful function should have the parse tree thus far constructed and the cursor, which points to the rest of the bitstream. It cannot alter them but may examine them. We haven't specified a representation for a parse tree, and don't intend to do so immediately. Without one, functional notation is pointless.

##Optional Grammar

Optional grammar is mandatory, that is, a GGG implementation *must* recognize it. It is optional in the sense that it specifies behaviors that a GGG implementation *may or may not* provide.

There are times when intermediate rules interfere with a clean parse tree. Other times, there is padding or whitespace in the bitstream that an implementation may not want to deal with. A GGG implementation *must* provide a mode in which it either succeeds or fails in some fashion without modifying the source, but it *may* provide other behaviors. 

To indicate that rules or literals should be hidden, put them in brackets, like `<hidden-rule>` and `<0x3F>`. Again, this may or may not correspond to a behavior in a GGG implementation, and it *must not* compact or change the source atom in validation mode. A validating GGG may as well strip all `<` and `>` encountered, as such behavior *must not* change the result of validation.

Comments begin with `//` and terminate in a newline. Block comments begin with `</` and end with `\>`. They do not nest.  

Metadata starts with `μ{` and ends with `}`. The contents should be EDN format restricted to printable ASCII characters, though I suppose any format that requires balanced angle brackets will work. Compliant GGG must recognize metadata, and may use it, but functions may *not* access it. 

##ASCII input mode.

For maximum keyboard convenience, GGG may be completely composed in ASCII. This format *must* be converted by a separate formatting stage. Please do not publish it or implement it directly, God forbid. 

There are a few common characters we didn't use. `$` may substitute for `£`, sensibly enough, and you may use `%` instead of `±` and `::` instead of `§`. We do use `:` but we don't allow it twice. Good enough for input, but GGG should be faster to parse and cleaner than that. Negative lookahead is `-?`; rules can't start with a dash, so we're ok here. 

The broken bar, `¦`, may be represented with bamboo `||`. The reference is to the shape in Go. Two regular alternators in a row is a syntax error in any mode, as is two `¦¦`, while `//` will be interpreted as a line comment by the formatter and a syntax error by GGG proper. 

For optional whitespace a single `.` will suffice, though it must be surrounded by real whitespace on both sides. Nor may you say it twice; too much like the range operator while being syntactically obtuse. 

Literal strings, well, use `" "`. But keep in mind there is no string escaping, and you'll need to concatenate a literal 0x22 in between two literal strings to get ASCII `"`. That's the price you pay for not arming yourself with guillemets. 

Metadata uses `#{` and `}`. All valid `#` must begin with `~` so again, we're ok. 

You will note that there is herein provided no way to provide accent marks in ASCII. This is because you are discouraged from marking your own rules, since the formatter will only correct you. If you wish to move them around for decoration, or add extras to rules that have more than one logical word in the name, feel free; the formatter will change every mark it finds to the equivalently accented vowel if it decides a rule is of a certain type. If you're doing this, you're using Latin-1 or a Unicode anyway; the ASCII equivalents are supported regardless, mix and match, but formatted GGG will use only one representation.



