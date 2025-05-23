AST and Source Location
=======================

## Literals

### Singletons

Format:

~~~
(true)
"true"
 ~~~~ expression

(false)
"false"
 ~~~~~ expression

(nil)
"nil"
 ~~~ expression
~~~

### Integer

Format:

~~~
(int 123)
"123"
 ~~~ expression

(int -123)
"-123"
 ^ operator
 ~~~ expression

(int 1)
"__LINE__"
 ~~~~~~~~ expression
~~~

### Float

Format:

~~~
(float 1.0)
"1.0"
 ~~~ expression

(float -1.0)
"-1.0"
 ^ operator
 ~~~~ expression
~~~

### Complex

Format:

~~~
(complex (0+1i))
"1i"
 ~~ expression

(complex (0+(1/1)*i))
"1ri"
 ~~~ expression
~~~

### Rational

Format:

~~~
(rational (2/1))
"2.0r"
 ~~~~ expression
~~~

### String

#### Plain

Format:

~~~
(str "foo")
"'foo'"
 ^ begin
     ^ end
 ~~~~~ expresion

(string "foo.rb")
"__FILE__"
 ~~~~~~~~ expression
~~~

#### With interpolation

Format:

~~~
(dstr (str "foo") (begin (lvar bar)) (str "baz"))
'"foo#{bar}baz"'
 ^ begin      ^ end
 ~~~~~~~~~~~~~~ expression
     ^^ begin (begin)
          ^ end (begin)
     ^^^^^^ expression (begin)
~~~

#### Here document

Format:

~~~
(str "foo\nbar\n")
'<<HERE␊foo␊bar␊HERE'
 ~~~~~~ expression
        ~~~~~~~~ heredoc_body
                ~~~~ heredoc_end
~~~

### Symbol

#### Plain

Format:

~~~
(sym :foo)
":foo"
 ~~~~ expresion

":'foo'"
  ^ begin
      ^ end
 ~~~~~~ expression
~~~

#### With interpolation

Format:

~~~
(dsym (str "foo") (lvar bar) (str "baz"))
':"foo#{bar}baz"'
  ^ begin      ^ end
 ~~~~~~~~~~~~~~~ expression
~~~

### Execute-string

#### Plain

Format:

~~~
(xstr (str "foo") (lvar bar))
"`foo#{bar}`"
 ^ begin   ^ end
 ~~~~~~~~~~~ expression
~~~

#### Here document

Format:

~~~
(xstr (str "foo\nbar\n"))
"<<`HERE`␊foo␊bar␊HERE"
 ~~~~~~~~ expression
          ~~~~~~~~ heredoc_body
                  ~~~~ heredoc_end
~~~

### Regexp

#### Options

Format:

~~~
(regopt :i :m)
"im"
 ~~ expression
~~~

#### Regexp

Format:

~~~
(regexp (str "foo") (lvar :bar) (regopt :i))
"/foo#{bar}/i"
 ^ begin   ^ end
 ~~~~~~~~~~~ expression
~~~

### Array

#### Plain

Format:

~~~
(array (int 1) (int 2))

"[1, 2]"
 ^ begin
      ^ end
 ~~~~~~ expression
~~~

#### Splat

Can also be used in argument lists: `foo(bar, *baz)`

Format:

~~~
(splat (lvar :foo))
"*foo"
 ^ operator
 ~~~~ expression
~~~

#### With interpolation

Format:

~~~
(array (int 1) (splat (lvar :foo)) (int 2))

"[1, *foo, 2]"
 ^ begin    ^ end
 ~~~~~~~~~~~~ expression
~~~

### Hash

#### Pair

##### With hashrocket

Format:

~~~
(pair (int 1) (int 2))
"1 => 2"
   ~~ operator
 ~~~~~~ expression
~~~

##### With label (1.9)

Format:

~~~
(pair (sym :answer) (int 42))
"answer: 42"
       ^ operator (pair)
 ~~~~~~ expression (sym)
 ~~~~~~~~~~ expression (pair)
~~~

#### With local variable

Format:

~~~
(pair (sym :foo) (lvar :foo))
"{foo:}"
     ^ operator (pair)
  ~~~ expression (sym)
  ~~~ expression (lvar)
~~~

#### With constant

Format:

~~~
(pair (sym :foo) (const nil :foo))
"{FOO:}"
     ^ operator (pair)
  ~~~ expression (const)
  ~~~ expression (lvar)
~~~

#### With method call

Format:

~~~
(pair (sym :puts) (send nil :puts))
"{puts:}"
      ^ operator (pair)
  ~~~~ expression (sym)
  ~~~~ expression (send)
~~~

#### Plain

Format:

~~~
(hash (pair (int 1) (int 2)) (pair (int 3) (int 4)))
"{1 => 2, 3 => 4}"
 ^ begin        ^ end
 ~~~~~~~~~~~~~~~~ expression
~~~

### Kwargs

Starting from Ruby 2.7 only implicit hash literals (that are not wrapped into `{ .. }`) are passed as keyword arguments.
Explicit hash literals are passed as positional arguments.
This is reflected in AST as `kwargs` node that is emitted only for implicit
hash literals and only if `emit_kwargs` compatibility flag is enabled.

Note that it can be a part of `send`, `csend`, `index` and `yield` nodes.

Format:

~~~
(kwargs (pair (int 1) (int 2)) (kwsplat (lvar :bar)) (pair (sym :baz) (int 3)))
"foo(1 => 2, **bar, baz: 3)"
     ~~~~~~~~~~~~~~~~~~~~~ expression
~~~

#### Keyword splat (2.0)

Can also be used in argument lists: `foo(bar, **baz)`

Format:

~~~
(kwsplat (lvar :foo))
"**foo"
 ~~ operator
 ~~~~~ expression
~~~

#### With interpolation (2.0)

Format:

~~~
(hash (pair (sym :foo) (int 2)) (kwsplat (lvar :bar)))
"{ foo: 2, **bar }"
 ^ begin         ^ end
 ~~~~~~~~~~~~~~~~~ expression
~~~

### Range

#### Inclusive

Format:

~~~
(irange (int 1) (int 2))
"1..2"
  ~~ operator
 ~~~~ expression
~~~

#### Exclusive

Format:

~~~
(erange (int 1) (int 2))
"1...2"
  ~~~ operator
 ~~~~~ expression
~~~


### Endless (2.6)

Format:

~~~
(irange (int 1) nil)
"1.."
  ~~ operator
 ~~~ expression

(erange (int 1) nil)
"1..."
  ~~~ operator
 ~~~~ expression
~~~

### Beginless (2.7)

Format:

~~~
(irange nil (int 1))
"..1"
 ~~ operator
 ~~~ expression

(erange nil (int 1))
"...1"
 ~~~ operator
 ~~~~ expression
~~~

## Access

### Self

Format:

~~~
(self)
"self"
 ~~~~ expression
~~~

### Local variable

Format:

~~~
(lvar :foo)
"foo"
 ~~~ expression
~~~

### Instance variable

Format:

~~~
(ivar :@foo)
"@foo"
 ~~~~ expression
~~~

### Class variable

Format:

~~~
(cvar :@@foo)
"@@foo"
 ~~~~~ expression
~~~

### Global variable

#### Regular global variable

Format:

~~~
(gvar :$foo)
"$foo"
 ~~~~ expression
~~~

#### Regular expression capture groups

Format:

~~~
(nth-ref 1)
"$1"
 ~~ expression
~~~

#### Regular expression back-references

Format:

~~~
(back-ref :$&)
"$&"
 ~~ expression
(back-ref :$`)
"$`"
(back-ref :$')
"$'"
(back-ref :$+)
"$+"
~~~

### Constant

#### Top-level constant

Format:

~~~
(const (cbase) :Foo)
"::Foo"
   ~~~ name
 ~~ double_colon
 ~~~~~ expression
~~~

#### Scoped constant

Format:

~~~
(const (lvar :a) :Foo)
"a::Foo"
    ~~~ name
  ~~ double_colon
 ~~~~~~ expression
~~~

#### Unscoped constant

Format:

~~~
(const nil :Foo)
"Foo"
 ~~~ name
 ~~~ expression
~~~

### defined?

Format:

~~~
(defined? (lvar :a))
"defined? a"
 ~~~~~~~~ keyword
 ~~~~~~~~~~ expression

"defined?(a)"
 ~~~~~~~~ keyword
         ^ begin
           ^ end
 ~~~~~~~~~~~ expression
~~~

## Assignment

### To local variable

Format:

~~~
(lvasgn :foo (lvar :bar))
"foo = bar"
     ^ operator
 ~~~~~~~~~ expression
~~~

### To instance variable

Format:

~~~
(ivasgn :@foo (lvar :bar))
"@foo = bar"
      ^ operator
 ~~~~~~~~~~ expression
~~~

### To class variable

Format:

~~~
(cvasgn :@@foo (lvar :bar))
"@@foo = bar"
       ^ operator
 ~~~~~~~~~~~ expression
~~~

### To global variable

Format:

~~~
(gvasgn :$foo (lvar :bar))
"$foo = bar"
      ^ operator
 ~~~~~~~~~~ expression
~~~

### To constant

#### Top-level constant

Format:

~~~
(casgn (cbase) :Foo (int 1))
"::Foo = 1"
   ~~~ name
       ~ operator
 ~~~~~~~ expression
~~~

#### Scoped constant

Format:

~~~
(casgn (lvar :a) :Foo (int 1))
"a::Foo = 1"
    ~~~ name
        ~ operator
 ~~~~~~~~ expression
~~~

#### Unscoped constant

Format:

~~~
(casgn nil :Foo (int 1))
"Foo = 1"
 ~~~ name
     ~ operator
 ~~~~~~~ expression
~~~

### To attribute

Format:

~~~
(send (self) :foo= (int 1))
"self.foo = 1"
     ^ dot
      ~~~ selector
          ^ operator
 ~~~~~~~~~~~~ expression
~~~

### To attribute, using "safe navigation operator"

Format:

~~~
(csend (self) :foo= (int 1))
"self&.foo = 1"
     ^^ dot
       ~~~ selector
           ^ operator
 ~~~~~~~~~~~~~ expression
~~~

### Multiple assignment

#### Multiple left hand side

Format:

~~~
(mlhs (lvasgn :a) (lvasgn :b))
"a, b"
 ~~~~ expression
"(a, b)"
 ^ begin
      ^ end
 ~~~~~~ expression
~~~

#### Assignment

Rule of thumb: every node inside `(mlhs)` is "incomplete"; to make
it "complete", one could imagine that a corresponding node from the
mrhs is "appended" to the node in question. This applies both to
side-effect free assignments (`lvasgn`, etc) and side-effectful
assignments (`send`).

Format:

~~~
(masgn (mlhs (lvasgn :foo) (lvasgn :bar)) (array (int 1) (int 2)))
"foo, bar = 1, 2"
          ^ operator
 ~~~~~~~~~~~~~~~ expression

(masgn (mlhs (ivasgn :@a) (cvasgn :@@b)) (array (splat (lvar :c))))
"@a, @@b = *c"

(masgn (mlhs (lvasgn :a) (mlhs (lvasgn :b)) (lvasgn :c)) (lvar :d))
"a, (b, c) = d"

(masgn (mlhs (send (self) :a=) (send (self) :[]= (int 1))) (lvar :a))
"self.a, self[1] = a"
~~~

### Binary operator-assignment

Binary operator-assignment features the same "incomplete assignments" and "incomplete calls" as [multiple assignment](#assignment-1).

#### Variable binary operator-assignment

Format:

~~~
(op-asgn (lvasgn :a) :+ (int 1))
"a += 1"
   ~~ operator
 ~~~~~~ expression

(op-asgn (ivasgn :a) :+ (int 1))
"@a += 1"
~~~

#### Method binary operator-assignment

Format:

~~~
(op-asgn (send (ivar :@a) :b) :+ (int 1))
"@a.b += 1"
    ~ selector (send)
 ~~~~ expression (send)
      ~~ operator (op-asgn)
 ~~~~~~~~~ expression (op-asgn)

(op-asgn (send (ivar :@a) :[] (int 0) (int 1))) :+ (int 1))
"@a[0, 1] += 1"
   ~~~~~~ selector (send)
 ~~~~~~~~ expression (send)
          ~~ operator (op-asgn)
 ~~~~~~~~~~~~~ expression (op-asgn)
~~~

### Logical operator-assignment

Logical operator-assignment features the same "incomplete assignments" and "incomplete calls" as [multiple assignment](#assignment-1).

#### Variable logical operator-assignment

Format:

~~~
(or-asgn (ivasgn :@a) (int 1))
"@a ||= 1"
    ~~~ operator
 ~~~~~~~~ expression

(and-asgn (lvasgn :a) (int 1))
"a &&= 1"
   ~~~ operator
 ~~~~~~~ expression
~~~

#### Method logical operator-assignment

Format:

~~~
(or-asgn (send (ivar :@foo) :bar) (int 1))
"@foo.bar ||= 1"
      ~~~ selector (send)
 ~~~~~~~~ expr (send)
          ~~~ operator (or-asgn)
 ~~~~~~~~~~~~~~ expression (or-asgn)

(and-asgn (send (lvar :@foo) :bar) (int 1))
"foo.bar &&= 1"
     ~~~ selector (send)
 ~~~~~~~ expr (send)
         ~~~ operator (and-asgn)
 ~~~~~~~~~~~~~ expression (and-asgn)

(or-asgn (send (ivar :@foo) :[] (int 1) (int 2)) (int 1))
"@foo[1, 2] ||= 1"
     ~~~~~~ selector (send)
 ~~~~~~~~~~ expr (send)
            ~~~ operator (or-asgn)
 ~~~~~~~~~~~~~~~~ expression (or-asgn)

~~~

### Right-hand assignment

Format:

~~~
(lvasgn :a (int 1))
"1 => a"
 ~~~~~~ expression
      ~ name
   ~~ operator
~~~

#### Multiple right-hand assignment

Format:

~~~
(masgn (mlhs (lvasgn :a) (lvasgn :b)) (send (int 13) :divmod (int 5)))
"13.divmod(5) => a,b"
 ~~~~~~~~~~~~~~~~~~~ expression
              ^^ operator
~~~

## Class and module definition

### Module

Format:

~~~
(module (const nil :Foo) (nil))
"module Foo; end"
 ~~~~~~ keyword
             ~~~ end
~~~

### Class

Format:

~~~
(class (const nil :Foo) (const nil :Bar) (nil))
"class Foo < Bar; end"
 ~~~~~ keyword    ~~~ end
           ~ operator
 ~~~~~~~~~~~~~~~~~~~~ expression

(class (const nil :Foo) nil (nil))
"class Foo; end"
 ~~~~~ keyword
            ~~~ end
 ~~~~~~~~~~~~~~ expression
~~~

### Singleton class

Format:

~~~
(sclass (lvar :a) (nil))
"class << a; end"
 ~~~~~ keyword
       ~~ operator
             ~~~ end
 ~~~~~~~~~~~~~~~ expression
~~~

## Method (un)definition

### Instance methods

Format:

~~~
(def :foo (args) nil)
"def foo; end"
 ~~~ keyword
     ~~~ name
          ~~~ end
 ~~~~~~~~~~~~ expression
~~~

### Singleton methods

Format:

~~~
(defs (self) :foo (args) nil)
"def self.foo; end"
 ~~~ keyword
          ~~~ name
               ~~~ end
 ~~~~~~~~~~~~~~~~~ expression
~~~

### "Endless" method

Format:

~~~
(def :foo (args) (int 42))
"def foo() = 42"
 ~~~ keyword
     ~~~ name
           ^ assignment
 ~~~~~~~~~~~~~~ expression
~~~


### "Endless" singleton method

Format:

~~~
(defs (self) :foo (args) (int 42))
"def self.foo() = 42"
 ~~~ keyword
          ~~~ name
                ^ assignment
 ~~~~~~~~~~~~~~~~~~~ expression
~~~

### Undefinition

Format:

~~~
(undef (sym :foo) (sym :bar) (dsym (str "foo") (int 1)))
"undef foo :bar :"foo#{1}""
 ~~~~~ keyword
 ~~~~~~~~~~~~~~~~~~~~~~~~~ expression
~~~

## Aliasing

### Method aliasing

Format:

~~~
(alias (sym :foo) (dsym (str "foo") (int 1)))
"alias foo :"foo#{1}""
 ~~~~~ keyword
 ~~~~~~~~~~~~~~~~~~~~ expression
~~~

### Global variable aliasing

Format:

~~~
(alias (gvar :$foo) (gvar :$bar))
"alias $foo $bar"
 ~~~~~ keyword
 ~~~~~~~~~~~~~~~ expression

(alias (gvar :$foo) (back-ref :$&))
"alias $foo $&"
 ~~~~~ keyword
 ~~~~~~~~~~~~~~~ expression
~~~

## Formal arguments

Format:

~~~
(args (arg :foo))
"(foo)"
 ~~~~~ expression
~~~

### Required argument

Format:

~~~
(arg :foo)
"foo"
 ~~~ expression
 ~~~ name
~~~

### Optional argument

Format:

~~~
(optarg :foo (int 1))
"foo = 1"
 ~~~~~~~ expression
     ^ operator
 ~~~ name
~~~

### Named splat argument

Format:

~~~
(restarg :foo)
"*foo"
 ~~~~ expression
  ~~~ name
~~~

Begin of the `expression` points to `*`.

### Unnamed splat argument

Format:

~~~
(restarg)
"*"
 ^ expression
~~~

### Block argument

Format:

~~~
(blockarg :foo)
"&foo"
  ~~~ name
 ~~~~ expression
~~~

Begin of the `expression` points to `&`.


### Anonymous block argument

Format:

~~~
(blockarg nil)
"&"
 ~ expression
~~~

### Auto-expanding proc argument (1.9)

In Ruby 1.9 and later, when a proc-like closure (i.e. a closure
created by capturing a block or with the `proc` method, but not
with the `->{}` syntax or the `lambda` method) has exactly one
argument, and it is called with more than one argument, the behavior
is as if the array of all arguments was instead passed as the sole
argument. This behavior can be prevented by adding a comma after
the sole argument (e.g. `|foo,|`).

Format:

~~~
(procarg0 (arg :foo))
"|foo|"
  ~~~ expression

(procarg0 (arg :foo) (arg :bar))
"|(foo, bar)|"
  ~ begin
           ~ end
  ~~~~~~~~~~ expression
~~~

### Expression arguments

Ruby 1.8 allows to use arbitrary expressions as block arguments,
such as `@var` or `foo.bar`. Such expressions should be treated as
if they were on the lhs of a multiple assignment.

Format:

~~~
(args (arg_expr (ivasgn :@bar)))
"|@bar|"

(args (arg_expr (send (send nil :foo) :a=)))
"|foo.a|"

(args (restarg_expr (ivasgn :@bar)))
"|*@bar|"

(args (blockarg_expr (ivasgn :@bar)))
"|&@bar|"
~~~

### Block shadow arguments

Format:

~~~
(args (shadowarg :foo) (shadowarg :bar))
"|; foo, bar|"
~~~

### Decomposition

Format:

~~~
(def :f (args (arg :a) (mlhs (arg :foo) (restarg :bar))))
"def f(a, (foo, *bar)); end"
          ^ begin   ^ end
          ~~~~~~~~~~~ expression
~~~

### Required keyword argument

Format:

~~~
(kwarg :foo)
"foo:"
 ~~~~ expression
 ~~~~ name
~~~

### Optional keyword argument

Format:

~~~
(kwoptarg :foo (int 1))
"foo: 1"
 ~~~~~~ expression
 ~~~~ name
~~~

### Named keyword splat argument

Format:

~~~
(kwrestarg :foo)
"**foo"
 ~~~~~ expression
   ~~~ name
~~~

### Unnamed keyword splat argument

Format:

~~~
(kwrestarg)
"**"
 ~~ expression
~~~

### Keyword nil argument

Format:

~~~
(kwnilarg)
"**nil"
   ~~~ name
 ~~~~~ expression
~~~

### Objective-C arguments

MacRuby includes a few more syntactic "arguments" whose name becomes
the part of the Objective-C method name, despite looking like Ruby 2.0
keyword arguments, and are thus treated differently.

#### Objective-C label-like keyword argument

Format:

~~~
(objc-kwarg :a :b)
"a: b"
 ~ keyword
  ~ operator
    ~ argument
 ~~~~ expression
~~~

#### Objective-C pair-like keyword argument

Format:

~~~
(objc-kwarg :a :b)
"a => b"
 ~ keyword
   ~~ operator
      ~ argument
 ~~~~~~ expression
~~~

#### Objective-C keyword splat argument

Format:

~~~
(objc-restarg (objc-kwarg :foo))
"(*a: b)"
   ~ objc-kwarg.keyword
    ~ objc-kwarg.operator
      ~ objc-kwarg.argument
  ~ operator
  ~~~~~ expression
~~~

Note that these splat arguments will only be parsed inside parentheses,
e.g. in the following code:

~~~
def f((*a: b)); end
~~~

However, the following code results in a parse error:

~~~
def f(*a: b); end
~~~

## Numbered parameters

### Block with numbered parameters

Ruby 2.7 introduced a feature called "numbered parameters".
Numbered, implicit it, and ordinal parameters are mutually exclusive, so if the block
has only numbered parameters it also has a different AST node.

Note that the second child represents a total number of numbered parameters.

Format:

~~~
s(:numblock,
  s(:send, nil, :proc), 3,
  s(:send,
    s(:lvar, :_1), :+,
    s(:lvar, :_3)))
"proc { _1 + _3 }"
      ~ begin   ~ end
 ~~~~~~~~~~~~~~~~ expression
~~~

## Block with `it` block parameter (Prism only)

Ruby 3.4 introduced a feature called the "`it` block parameter".
`it`, numbered, and ordinal parameters are mutually exclusive, so if the block
only has the implicit it parameter it also has a different AST node.

The second child will always be the symbol `it`.

Format:

~~~
s(:itblock,
  s(:send, nil, :proc), :it,
  s(:send, s(:lvar, :it)))
"proc { it }"
 ~~~~~~~~~~~ expression
~~~

## Forward arguments

### Method definition accepting only forwarding arguments

Ruby 2.7 introduced a feature called "arguments forwarding".
When a method takes any arguments for forwarding them in the future
the whole `args` node gets replaced with `forward-args` node.

Format if `emit_forward_arg` compatibility flag is disabled:

~~~
(def :foo
  (forward-args) nil)
"def foo(...); end"
            ~ end
        ~ begin
        ~~~~~ expression
~~~

However, Ruby 3.0 added support for leading arguments before `...`, and so
it can't be used as a replacement of the `(args)` node anymore. To solve it
`emit_forward_arg` should be enabled.

Format if `emit_forward_arg` compatibility flag is enabled:

~~~
(def :foo
  (args
    (forward-arg)) nil)
"def foo(...); end"
        ~ begin (args)
            ~ end (args)
        ~~~~~ expression (args)
         ~~~ expression (forward_arg)
~~~

Note that the node is called `forward_arg` when emitted separately.

### Method call taking arguments of the currently forwarding method

Format:

~~~
(send nil :foo
  (forwarded-args))
"foo(...)"
     ~~~ expression
~~~

### Method call taking positional arguments of the currently called method

Format:

~~~
(send nil :foo
  (forwarded-restarg))
"foo(*)"
     ~ expression
~~~

### Method call taking keyword arguments of the currently called method

Format:

~~~
(send nil :foo
  (forwarded-kwrestarg))
"foo(**)"
     ~~ expression
~~~

## Send

### To self

Format:

~~~
(send nil :foo (lvar :bar))
"foo(bar)"
 ~~~ selector
    ^ begin
        ^ end
 ~~~~~~~~ expression
~~~

### To receiver

Format:

~~~
(send (lvar :foo) :bar (int 1))
"foo.bar(1)"
    ^ dot
     ~~~ selector
        ^ begin
          ^ end
 ~~~~~~~~~~ expression

(send (lvar :foo) :+ (int 1))
"foo + 1"
     ^ selector
 ~~~~~~~ expression

(send (lvar :foo) :-@)
"-foo"
 ^ selector
 ~~~~ expression

(send (lvar :foo) :a= (int 1))
"foo.a = 1"
     ~ selector
       ^ operator
 ~~~~~~~~~ expression
~~~

### To superclass

Format of super with arguments:

~~~
(super (lvar :a))
"super a"
 ~~~~~ keyword
 ~~~~~~~ expression

(super)
"super()"
      ^ begin
       ^ end
 ~~~~~ keyword
 ~~~~~~~ expression
~~~

Format of super without arguments (**z**ero-arity):

~~~
(zsuper)
"super"
 ~~~~~ keyword
 ~~~~~ expression
~~~

### To block argument

Format:

~~~
(yield (lvar :foo))
"yield(foo)"
 ~~~~~ keyword
      ^ begin
          ^ end
 ~~~~~~~~~~ expression
~~~

### Indexing

Format:

~~~
(index (lvar :foo) (int 1))
"foo[1]"
    ^ begin
      ^ end
 ~~~~~~ expression

(indexasgn (lvar :bar) (int 1) (int 2) (lvar :baz))
"bar[1, 2] = baz"
    ^ begin
         ^ end
           ^ operator
 ~~~~~~~~~~~~~~~ expression

~~~

### Passing a literal block

~~~
(block (send nil :foo) (args (arg :bar)) (begin ...))
"foo do |bar|; end"
     ~~ begin
               ~~~ end
     ~~~~~~~~~~~~~ expression
~~~

### Passing expression as block

Used when passing expression as block `foo(&bar)`

~~~
(send nil :foo (int 1) (block-pass (lvar :foo)))
"foo(1, &foo)"
        ^ operator
        ~~~~ expression
~~~

### Passing expression as anonymous block `foo(&)`

~~~
(send nil :foo (int 1) (block-pass nil))
"foo(1, &)"
        ^ operator
        ~ expression
~~~

### "Stabby lambda"

~~~
(block (lambda) (args) nil)
"-> {}"
 ~~ lambda.expression
~~~

### "Safe navigation operator"

~~~
(csend (send nil :foo) :bar)
"foo&.bar"
    ~~ dot
~~~

### Objective-C variadic send

MacRuby allows to pass a variadic amount of arguments via the last
keyword "argument". Semantically, these, together with the pair value
of the last pair in the hash implicitly passed as the last argument,
form an array, which replaces the pair value. Despite that, the node
is called `objc-varargs` to distinguish it from a literal array passed
as a value.

~~~
(send nil :foo (int 1) (hash (pair (sym :bar) (objc-varargs (int 1) (int 2) (nil)))))
"foo(1, bar: 2, 3, nil)"
             ~~~~~~~~~ expression (array)
~~~

## Control flow

### Logical operators

#### Binary (and or && ||)

Format:

~~~
(and (lvar :foo) (lvar :bar))
"foo and bar"
     ~~~ operator
 ~~~~~~~~~~~ expression
~~~

~~~
(or (lvar :foo) (lvar :bar))
"foo or bar"
     ~~ operator
 ~~~~~~~~~~ expression
~~~

#### Unary (! not) (1.8)

Format:

~~~
(not (lvar :foo))
"!foo"
 ^ operator
"not foo"
 ~~~ operator
~~~

### Branching

#### Without else

Format:

~~~
(if (lvar :cond) (lvar :iftrue) nil)
"if cond then iftrue; end"
 ~~ keyword
         ~~~~ begin
                      ~~~ end
 ~~~~~~~~~~~~~~~~~~~~~~~~ expression

"if cond; iftrue; end"
 ~~ keyword
                  ~~~ end
 ~~~~~~~~~~~~~~~~~~~~ expression

"iftrue if cond"
        ~~ keyword
 ~~~~~~~~~~~~~~ expression

(if (lvar :cond) nil (lvar :iftrue))
"unless cond then iftrue; end"
 ~~~~~~ keyword
             ~~~~ begin
                          ~~~ end
 ~~~~~~~~~~~~~~~~~~~~~~~~~~~~ expression

"unless cond; iftrue; end"
 ~~~~~~ keyword
                      ~~~ end
 ~~~~~~~~~~~~~~~~~~~~~~~~ expression

"iftrue unless cond"
        ~~~~~~ keyword
 ~~~~~~~~~~~~~~~~~~ expression
~~~

#### With else

Format:

~~~
(if (lvar :cond) (lvar :iftrue) (lvar :iffalse))
"if cond then iftrue; else; iffalse; end"
 ~~ keyword
         ~~~~ begin
                      ~~~~ else
                                 ~~~ end
 ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ expression

"if cond; iftrue; else; iffalse; end"
 ~~ keyword
                  ~~~~ else
                                 ~~~ end
 ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ expression

(if (lvar :cond) (lvar :iffalse) (lvar :iftrue))
"unless cond then iftrue; else; iffalse; end"
 ~~~~~~ keyword
             ~~~~ begin
                          ~~~~ else
                                         ~~~ end
 ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ expression

"unless cond; iftrue; else; iffalse; end"
 ~~~~~~ keyword
                      ~~~~ else
                                     ~~~ end
 ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ expression
~~~

#### With elsif

Format:

~~~
(if (lvar :cond1) (int 1) (if (lvar :cond2 (int 2) (int 3))))
"if cond1; 1; elsif cond2; 2; else 3; end"
 ~~ keyword (left)
              ~~~~~ else (left)
                                      ~~~ end (left)
              ~~~~~ keyword (right)
                              ~~~~ else (right)
                                      ~~~ end (right)
~~~

#### Ternary

Format:

~~~
(if (lvar :cond) (lvar :iftrue) (lvar :iffalse))
"cond ? iftrue : iffalse"
      ^ question
               ^ colon
 ~~~~~~~~~~~~~~~~~~~~~~~ expression
~~~

### Case matching

#### When clause

Format:

~~~
(when (regexp "foo" (regopt)) (begin (lvar :bar)))
"when /foo/ then bar"
 ~~~~ keyword
            ~~~~ begin
 ~~~~~~~~~~~~~~~~~~~ expression

(when (int 1) (int 2) (send nil :meth))
"when 1, 2; meth"

(when (int 1) (splat (lvar :foo)) (send nil :meth))
"when 1, *foo; meth"

(when (splat (lvar :foo)) (send nil :meth))
"when *foo; meth"
~~~

#### Case-expression clause

##### Without else

Format:

~~~
(case (lvar :foo) (when (str "bar") (lvar :bar)) nil)
"case foo; when "bar"; bar; end"
 ~~~~ keyword               ~~~ end
~~~

##### With else

Format:

~~~
(case (lvar :foo) (when (str "bar") (lvar :bar)) (lvar :baz))
"case foo; when "bar"; bar; else baz; end"
 ~~~~ keyword               ~~~~ else ~~~ end
~~~

#### Case-conditions clause

##### Without else

Format:

~~~
(case nil (when (lvar :bar) (lvar :bar)) nil)
"case; when bar; bar; end"
 ~~~~ keyword         ~~~ end
~~~

##### With else

Format:

~~~
(case nil (when (lvar :bar) (lvar :bar)) (lvar :baz))
"case; when bar; bar; else baz; end"
 ~~~~ keyword         ~~~~ else ~~~ end

(case nil (lvar :baz))
"case; else baz; end"
 ~~~~ keyword
       ~~~~ else
                 ~~~ end
~~~

### Looping

#### With precondition

Format:

~~~
(while (lvar :condition) (send nil :foo))
"while condition do foo; end"
 ~~~~~ keyword
                 ~~ begin
                         ~~~ end
 ~~~~~~~~~~~~~~~~~~~~~~~~~~~ expression

"while condition; foo; end"
 ~~~~~ keyword
                       ~~~ end
 ~~~~~~~~~~~~~~~~~~~~~~~~~ expression

"foo while condition"
     ~~~~~ keyword
 ~~~~~~~~~~~~~~~~~~~ expression

(until (lvar :condition) (send nil :foo))
"until condition do foo; end"
 ~~~~~ keyword
                 ~~ begin
                         ~~~ end
 ~~~~~~~~~~~~~~~~~~~~~~~~~~~ expression

(until (lvar :condition) (send nil :foo))
"until condition; foo; end"
 ~~~~~ keyword
                       ~~~ end
~~~~~~~~~~~~~~~~~~~~~~~~~~ expression

"foo until condition"
     ~~~~~ keyword
 ~~~~~~~~~~~~~~~~~~~ expression
~~~

#### With postcondition

Format:

~~~
(while-post (lvar :condition) (kwbegin (send nil :foo)))
"begin; foo; end while condition"
 ~~~~~ begin (begin)
             ~~~ end (begin)
                 ~~~~~ keyword (while-post)

(until-post (lvar :condition) (kwbegin (send nil :foo)))
"begin; foo; end until condition"
 ~~~~~ begin (begin)
             ~~~ end (begin)
                 ~~~~~ keyword (until-post)
~~~

#### For-in

Format:

~~~
(for (lvasgn :a) (lvar :array) (send nil :p (lvar :a)))
"for a in array do p a; end"
 ~~~ keyword
       ~~ in
                ~~ begin
                        ~~~ end

"for a in array; p a; end"
 ~~~ keyword
       ~~ in
                      ~~~ end

(for
  (mlhs (lvasgn :a) (lvasgn :b)) (lvar :array)
  (send nil :p (lvar :a) (lvar :b)))
"for a, b in array; p a, b; end"
~~~

#### Break

Format:

~~~
(break (int 1))
"break 1"
 ~~~~~ keyword
 ~~~~~~~ expression
~~~

#### Next

Format:

~~~
(next (int 1))
"next 1"
 ~~~~ keyword
 ~~~~~~ expression
~~~

#### Redo

Format:

~~~
(redo)
"redo"
 ~~~~ keyword
 ~~~~ expression
~~~

### Return

Format:

~~~
(return (lvar :foo))
"return(foo)"
 ~~~~~~ keyword
 ~~~~~~~~~~~ expression
~~~

### Exception handling

#### Rescue body

Format:

~~~
(resbody (array (const nil :Exception) (const nil :A)) (lvasgn :bar) (int 1))
"rescue Exception, A => bar; 1"
 ~~~~~~ keyword      ~~ assoc
 ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ expression

"rescue Exception, A => bar then 1"
 ~~~~~~ keyword      ~~ assoc
                            ~~~~ begin

(resbody (array (const nil :Exception)) (ivasgn :bar) (int 1))
"rescue Exception => @bar; 1"
 ~~~~~~ keyword   ~~ assoc

(resbody nil (lvasgn :bar) (int 1))
"rescue => bar; 1"
 ~~~~~~ keyword
        ~~ assoc

(resbody nil nil (int 1))
"rescue; 1"
 ~~~~~~ keyword
~~~

#### Rescue statement

##### Without else

Format:

~~~
(begin
  (rescue (send nil :foo) (resbody ...) (resbody ...) nil))
"begin; foo; rescue Exception; rescue; end"
 ~~~~~ begin                           ~~~ end
             ~~~~~~~~~~~~~~~~~ expression (rescue.resbody/1)
                               ~~~~~~~ expression (rescue.resbody/2)
        ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ expression (rescue)
~~~

##### With else

Format:

~~~
(begin
  (rescue (send nil :foo) (resbody ...) (resbody ...) (true)))
"begin; foo; rescue Exception; rescue; else true end"
 ~~~~~ begin                           ~~~~ else (rescue)
                                                 ~~~ end
~~~

#### Ensure statement

Format:

~~~
(begin
  (ensure (send nil :foo) (send nil :bar))
"begin; foo; ensure; bar; end"
 ~~~~~ begin ~~~~~~ keyword (ensure)
                          ~~~ end
~~~

#### Rescue with ensure

Format:

~~~
(begin
  (ensure
    (rescue (send nil :foo) (resbody ...) (int 1))
    (send nil :bar))
"begin; foo; rescue; nil; else; 1; ensure; bar; end"
 ~~~~~ begin
                          ~~~~ else (ensure.rescue)
             ~~~~~~~~~~~~~~~~~~~~~ expression (rescue)
                                   ~~~~~~ keyword (ensure)
             ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ expression (ensure)
                                                ~~~ end
~~~

#### Retry

Format:

~~~
(retry)
"retry"
 ~~~~~ keyword
 ~~~~~ expression
~~~

### BEGIN and END

Format:

~~~
(preexe (send nil :puts (str "foo")))
"BEGIN { puts "foo" }"
 ~~~~~ keyword
       ^ begin      ^ end
 ~~~~~~~~~~~~~~~~~~~~ expression

(postexe (send nil :puts (str "bar")))
"END { puts "bar" }"
 ~~~ keyword
     ^ begin      ^ end
 ~~~~~~~~~~~~~~~~~~ expression
~~~

## Miscellanea

### Flip-flops

Format:

~~~
(iflipflop (lvar :a) (lvar :b))
"if a..b; end"
     ~~ operator
    ~~~~ expression

(eflipflop (lvar :a) (lvar :b))
"if a...b; end"
     ~~~ operator
    ~~~~~ expression
~~~

### Implicit matches

Format:

~~~
(match-current-line (regexp (str "a") (regopt)))
"if /a/; end"
    ~~~ expression
~~~

### Local variable injecting matches

Format:

~~~
(match-with-lvasgn (regexp (str "(?<match>bar)") (regopt)) (lvar :baz))
"/(?<match>bar)/ =~ baz"
                 ~~ selector
 ~~~~~~~~~~~~~~~~~~~~~~ expression
~~~

## Special constants

### File

Format:

~~~
(__FILE__)
"__FILE__"
 ~~~~~~~~ expression
~~~

### Line

Format:

~~~
(__LINE__)
"__LINE__"
 ~~~~~~~~ expression
~~~

### Encoding

Format:

~~~
(__ENCODING__)
"__ENCODING__"
 ~~~~~~~~~~~~ expression
~~~

## Pattern matching

### Using `in` operator

Ruby 2.7 throws a `NoMatchingPatternError` for `foo in bar` if given value doesn't match pattern.

Format when `emit_match_pattern` compatibility attribute is disabled (the default):

~~~
(in-match
  (int 1)
  (match-var :a))
"1 in a"
   ~~ operator
 ~~~~~~ expression
~~~

Format when `emit_match_pattern` is enabled:

~~~
(match-pattern
  (int 1)
  (match-var :a))
"1 in a"
   ~~ operator
 ~~~~~~ expression
~~~

Starting from 3.0 Ruby returns `true`/`false` for the same code construction.

Ruby 3.0 format (compatibility attribute has no effect):

~~~
(match-pattern-p
  (int 1)
  (match-var :a))
"1 in a"
   ~~ operator
 ~~~~~~ expression
~~~

### Using `=>` operator

This node appears in AST only starting from Ruby 3.0.

Format:

~~~
(match-pattern
  (int 1)
  (match-var :a))
"1 => a"
   ~~ operator
 ~~~~~~ expression
~~~

### Case with pattern matching

#### Without else

Format:

~~~
(case-match
  (str "str")
  (in-pattern
    (match-var :foo)
    (lvar :bar)) nil)
"case "str"; in foo; bar; end"
 ~~~~ keyword             ~~~ end
 ~~~~~~~~~~~~~~~~~~~~~~~~~~~~ expression
~~~

#### With else

Format:

~~~
(case-match,
  (str "str")
  (in-pattern
    (match-var :foo)
    (lvar :bar))
  (lvar :baz))
"case "str"; in foo; bar; else; baz; end"
 ~~~~ keyword             ~~~~ else  ~~~ end
 ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ expression
~~~

#### With empty else

Empty `else` differs from the missing (or _implicit_) `else` for pattern matching, since
the latter one raises a `NoMatchingPattern` exception. Thus, we need a way to distinguish this
two cases in the resulting AST.

Format:

~~~
(case-match,
  (str "str")
  (in-pattern
    (match-var :foo)
    (lvar :bar))
  (empty-else))
"case "str"; in foo; bar; else; end"
 ~~~~ keyword             ~~~~ else
                                ~~~ end
 ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ expression
~~~

### In clause

Format:

~~~
(in-pattern
  (match-var :foo)
  (lvar :bar))
"in foo then bar"
 ~~ keyword
        ~~~~ begin
 ~~~~~~~~~~~~~~~ expression
~~~

### If guard

This guard runs after matching, so it's not an `if` modifier.

Format:

~~~
(in-pattern
  (match-var :foo)
  (if-guard
    (lvar :bar)) nil)
"in foo if bar"
        ~~ keyword
        ~~~~~~ expression
~~~

### Unless guard

This guard runs after matching, so it's not an `unless` modifier.

Format:

~~~
(in-pattern
  (match-var :foo)
  (unless-guard
    (lvar :bar)) nil)
"in foo unless bar"
        ~~~~~~ keyword
        ~~~~~~~~~~ expression
~~~

### Match variable

Format:

~~~
(match-var :foo)
"in foo"
    ~~~ name
    ~~~ expression
~~~

### Match rest

#### With name

Format:

~~~
(match-rest
  (match-var :foo))
"in *foo"
    ~ operator
    ~~~~ expression
~~~

#### Without name

Format:

~~~
(match-rest)
"in *"
    ~ operator
    ~ expression
~~~

### Pin operator

Format:

~~~
(pin
  (lvar :foo))
"in ^foo"
    ~ selector
    ~~~~ expression
~~~

### Pin operator with expression

Format:

~~~
(pin
  (begin
    (send
      (int 2) :+
      (int 2))))
"in ^(2 + 2)"
    ~ selector
    ~~~~~~~~ expression
     ~ begin (begin)
           ~ end (begin)
     ~~~~~~~ expression (begin)
~~~

### Match alternative

Format:

~~~
(match-alt
  (pin
    (lvar :foo))
  (int 1))
"in ^foo | 1"
         ~ operator
    ~~~~~~~~ expression
~~~

### Match with alias

Format:

~~~
(match-as
  (int 1)
  (match-var :foo))
"in 1 => foo"
      ~~ operator
    ~~~~~~~~ expression
~~~

### Match using array pattern

#### Explicit

Format:

~~~
(array-pattern
  (pin
    (lvar :foo))
  (match-var :bar))
"in [^foo, bar]"
    ~ begin   ~ end
    ~~~~~~~~~~~ expression
~~~

#### Explicit with tail

Adding a trailing comma in the end works as `, *`

Format:

~~~
(array-pattern-with-tail
  (pin
    (lvar :foo))
  (match-var :bar))
"in [^foo, bar,]"
    ~ begin    ~ end
    ~~~~~~~~~~~~ expression
~~~

#### Implicit

Format:

~~~
(array-pattern
  (pin
    (lvar :foo))
  (match-var :bar))
"in ^foo, bar"
    ~~~~~~~~~ expression
~~~

#### Implicit with tail

Format:

Adding a trailing comma in the end works as `, *`,
so a single item match with comma gets interpreted as an array.

~~~
(array-pattern-with-tail
  (match-var :foo))
"in foo,"
    ~~~~ expression
~~~

### Matching using hash pattern

#### Explicit

Format:

~~~
(hash-pattern
  (pair
    (sym :a)
    (int 10)))
"in { a: 10 }"
    ~ begin ~ end
    ~~~~~~~~~ expression
~~~

#### Implicit

Format:

~~~
(hash-pattern
  (pair
    (sym :a)
    (int 10)))
"in a: 10"
    ~~~~~ expression
~~~

#### Assignment using hash pattern

Format:

~~~
(hash-pattern
  (match-var :a))
"in a:"
    ~ name (match-var)
    ~~ expression (match-var)
~~~

#### Nil hash pattern

Format:
~~~
(hash-pattern
  (match-nil-pattern))
"in **nil"
    ~~~~~ expression (match-nil-pattern)
      ~~~ name (match-nil-pattern)
~~~

### Matching using find pattern

Format:

~~~
(find-pattern
  (match-rest
    (match-var :a))
  (int 42)
  (match-rest))
"in [*, 42, *]"
    ~ begin
             ~ end
    ~~~~~~~~~~ expression
~~~

Note that it can be used as a top-level pattern only when used in a `case` statement. In that case `begin` and `end` are empty.

### Matching using const pattern

#### With array pattern

Format:

~~~
(const-pattern
  (const nil :X)
  (array-pattern
    (pin
      (lvar :foo))
    (match-var :bar)))
"in X[^foo bar]"
     ~ begin (const-pattern)
               ~ end (const-pattern)
    ~~~~~~~~~~~~ expression (const-pattern)
    ~ name (const-pattern.const)
    ~ expression (const-pattern.const)
~~~

#### With hash pattern

Format:

~~~
(const-pattern
  (const nil :X)
  (hash-pattern
    (match-var :foo)
    (match-var :bar)))
"in X[foo:, bar:]"
     ~ begin (const-pattern)
                ~ end (const-pattern)
    ~~~~~~~~~~~~~ expression (const-pattern)
    ~ name (const-pattern.const)
    ~ expression (const-pattern.const)
~~~

#### With array pattern without elements

Format:

~~~
(const-pattern
  (const nil :X)
  (array-pattern))
"in X[]"
     ~ begin (const-pattern)
      ~ end (const-pattern)
    ~~~ expression (const-pattern)
    ~ name (const-pattern.const)
    ~ expression (const-pattern.const)
     ~~ expression (const-pattern.array_pattern)
~~~

#### With find pattern

Format:

~~~
(const-pattern
  (const nil :X)
  (find-pattern
    (match-rest)
    (int 42)
    (match-rest)))
"in X[*, 42, *]"
     ~ begin
              ~ end
    ~~~~~~~~~~~ expression
~~~
