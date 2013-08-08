#A Bit of ECL.

These are some quick checks on the underlying implementation of ECL:

```lisp
> (defun foo(x) (1+ x))

FOO
> (compiled-function-p #'foo)

T
> (defun bar(x) (eval '(1+ x)))

BAR
> (compiled-function-p #'bar)
```
