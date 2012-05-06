optima - Optimized Pattern Matching Library
===========================================

optima is a very fast pattern matching library
which uses optimizing techniques widely used in a functional
programming world. See the following references for more detail:

* [Optimizing Pattern Matching](http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.6.5507) by Fabrice Le Fessant, Luc Maranget
* [The Implementation of Functional Programming Languages](http://research.microsoft.com/en-us/um/people/simonpj/papers/slpj-book-1987/) by Simon Peyton Jones

Pattern Language
----------------

A pattern specifier, or a pattern for short unless ambiguous, is an
expression that describes how a value matches the pattern. Pattern
specifiers are defined as follows:

    pattern-specifier ::= constant-pattern
                        | variable-pattern
                        | constructor-pattern
                        | derived-pattern
                        | as-pattern
                        | guard-pattern
                        | not-pattern
                        | or-pattern
    
    constant-pattern ::= t | nil
                       | atom-except-symbol
                       | (quote VALUE)
    
    variable-pattern ::= SYMBOL | (variable SYMBOL)
    
    constructor-pattern ::= (NAME PATTERN*)

    derived-pattern ::= (NAME PATTERN*)
    
    as-pattern ::= (as PATTERN NAME)
    
    guard-pattern ::= (guard PATTERN TEST-FORM)
    
    not-pattern ::= (not PATTERN)
    
    or-pattern ::= (or PATTERN*)

### Constant-Pattern

A constant-pattern matches the constant itself.

Examples:

    (match 1 (1 2)) => 2
    (match "foo" ("foo" "bar")) => "bar"
    (match '(1) ('(1) 2)) => 2

### Variable-Pattern

A variable-pattern matches any value and bind the value to the
variable. _ and otherwise is a special variable-pattern (a.k.a
wildcard-pattern) which matches any value but doesn't bind.

Examples:

    (match 1 (x x)) => 1
    (match 1 (_ 2)) => 2
    (match 1
      (2 2)
      (otherwise 'otherwise))
    => OTHERWISE

### Constructor-Pattern

A constructor-pattern matches not a value itself but a structure of
the value. The following constructors are available:

* (cons car cdr)
* (vector &rest elements)
* (simple-vector &rest elements)

Examples:

    (match '(1 . 2) ((cons a b) (+ a b))) => 3
    (match #(1 2) ((simple-vector a b) (+ a b))) => 3

In addition to constructor patterns above, there is one special
constructor pattern which matches any value of type of STANDARD-CLASS.
The form of the pattern looks like

    (class-name &rest slots)

where CLASS-NAME is a class name of the value, and SLOTS is a list of
the form of (SLOT-NAME PATTERN). You can also specify the element like
SLOT-NAME, which is a shorthand for (SLOT-NAME SLOT-NAME).

Examples:

    (defstruct person name age)
    (defvar foo (make-person :name "foo" :age 30))
    (match foo
      ((person name age) (list name age)))
    => ("foo" 30)
    (match foo
      ((person (name "bar")) 'matched)
      (_ 'not-matched))
    => NOT-MATCHED

### Dervied-Pattern

A derived-pattern is a pattern that is defined with DEFPATTERN. There
are some builtin dervied patterns as below:

#### LIST

Expansion of LIST derived patterns=

    (list a b c) => (cons a (cons b (cons c nil)))

#### LIST*

Expansion of LIST* derived patterns:

    (list a b c) => (cons a (cons b c))

### As-Pattern

An as-pattern matches any value that is matched with sub-PATTERN, and
binds the value to variable named NAME.

Examples:

    (match '(1 . 2) ((as (cons 1 _) cons) cons))
    => (1 . 2)

### Guard-Pattern

A guard-pattern restricts a matching of sub-PATTERN with a post
condition TEST-FORM. See also MATCH documentation.

Examples:

    (match 1 ((guard x (evenp x)) 'even))
    => NIL

### Not-Pattern

A not-pattern matches a value that is not matched with sub-PATTERN.

Examples:

    (match 1 ((not 2) 3)) => 3
    (match 1 ((not (not 1)) 1)) => 1

### Or-Pattern

An or-pattern matches a value that is matched with one of
sub-PATTERNs. There is a restriction that every pattern of
sub-PATTERNs must have same set of variables.

Examples:

    (match '(2 . 1) ((or (cons 1 x) (cons 2 x)) x))
    => 1

[Package] optima
----------------

## [Macro] match

    match arg &body clauses

Matches ARG with CLAUSES. CLAUSES is a list of the form of (PATTERN
. BODY) where PATTERN is a pattern specifier and BODY is an implicit
progn. If ARG is matched with some PATTERN, then evaluates
corresponding BODY and returns the evaluated value. Otherwise, returns
NIL.

If BODY starts with a symbol WHEN, then the next form will be used to
introduce a guard for PATTERN. That is,

    (match list ((list x) when (oddp x) x))

will be translated to

    (match list ((guard (list x) (oddp x)) x))

## [Macro] multiple-value-match

    multiple-value-match values-form &body clauses

Matches the multiple values of VALUES-FORM with CLAUSES. Unlike
MATCH, CLAUSES have to have the form of (PATTERNS . BODY), where
PATTERNS is a list of patterns. The number of values that will be used
to match is determined by the maximum arity of PATTERNS among CLAUSES.

Examples:

    (multiple-value-match (values 1 2)
     ((2) 1)
     ((1 y) y))
    => 2

## [Macro] smatch

    smatch arg &body clauses

Same as MATCH, except SMATCH binds variables by SYMBOL-MACROLET
instead of LET.

## [Macro] multiple-value-smatch

    multiple-value-smatch values-form &body clauses

Same as MULTIPLE-VALUE-MATCH, except MULTIPLE-VALUE-SMATCH binds
variables by SYMBOL-MACROLET instead of LET.

## [Macro] ematch

    ematch arg &body clauses

Same as MATCH, except MATCH-ERROR will be raised if not matched.

## [Macro] multiple-value-ematch

    multiple-value-ematch values-form &body clauses

Same as MULTIPLE-VALUE-MATCH, except MATCH-ERROR will be raised if
not matched.

## [Macro] esmatch

    esmatch arg &body clauses

Same as EMATCH, except ESMATCH binds variables by SYMBOL-MACROLET
instead of LET.

## [Macro] multiple-value-esmatch

    multiple-value-esmatch values-form &body clauses

Same as MULTIPLE-VALUE-EMATCH, except MULTIPLE-VALUE-ESMATCH binds
variables by SYMBOL-MACROLET instead of LET.

## [Macro] cmatch

    cmatch arg &body clauses

Same as MATCH, except continuable MATCH-ERROR will be raised if not
matched.

## [Macro] multiple-value-cmatch

    multiple-value-cmatch values-form &body clauses

Same as MULTIPLE-VALUE-MATCH, except continuable MATCH-ERROR will
be raised if not matched.

## [Macro] csmatch

    csmatch arg &body clauses

Same as CMATCH, except CSMATCH binds variables by SYMBOL-MACROLET
instead of LET.

## [Macro] multiple-value-csmatch

    multiple-value-csmatch values-form &body clauses

Same as MULTIPLE-VALUE-CMATCH, except MULTIPLE-VALUE-CSMATCH binds
variables by SYMBOL-MACROLET instead of LET.

## [Macro] xmatch

    xmatch (the type arg) &body clauses

Same as MATCH, except XMATCH does exhaustiveness analysis over
CLAUSES with a type of ARG. If the type is not covered by CLAUSES, in
other words, if the type is not a subtype of an union type of CLAUSES,
then a compile-time error will be raised.

You need to specify the type of ARG with THE special operator like:

    (xmatch (the type arg) ...)

Examples:

    (xmatch (the (member :a :b) :b) (:a 1) (:b 2) (:c 3))
    => 2
    (xmatch (the (member :a :b) :b) (:a 1))
    => COPMILE-TIME-ERROR

## [Macro] defpattern

    defpattern name lambda-list &body body

Defines a derived pattern specifier named NAME. This is analogous
to DEFTYPE.

Examples:
    ;; Defines a LIST pattern.
    (defpattern list (&rest args)
      (when args
        `(cons ,(car args) (list ,@(cdr args)))))

Authors
-------

* Tomohiro Matsuyama

License
-------

LLGPL