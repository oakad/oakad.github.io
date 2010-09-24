######################################################################
Language for system programing, or C is not going to stay here forever
######################################################################

*This article is still being written.*

It so happens, that both in development of new programing languages, as well
as in development of new operating systems, the issue of IO and interaction
with machine hardware in general is often neglected. Fancy abstractions,
breath-taking in their elegance or, to the contrary, excessive simplification
of language constructs and idioms draw almost all of the attention of language
designers. Yet, apart from focusing on the logical and psychological aspects
of software engineering, it seems worthwhile to pay some attention to the
real world machines and create a language which will target the hard task of
direct hardware management.

No wonder that C programing language remains a mainstay of system programing for
nearly 40 years already. There simply was no worthy successor to take on the
task. One of the honourable reasons for this, is of course a considerable
wisdom (if not genius) of its original designers, who were able to clearly
identify the goals and needs of a basic system language. C++, which aimed to
become such a successor, had essentially failed in this task.

Here I would like to point out, that, to my opinion, modern C++ is the most
powerful and versatile language in existence, and well written programs in it
are often exceptionally elegant. Yet, to be used in a system role it must be
necessarily reduced to a subset of its features and, therefore, loses a lot of
its innate flexibility.

**********************************************************
What is really needed from the system programing language?
**********************************************************

Every good design must build on a solid conceptual foundation. For a system
language, the main law by which it should behave can be stated as *"no run-time
magic"*. The exact behaviour of program in run-time should be more or less
obvious from the inspection of source code.

We can identify two major sorts of "magic": storage domain and time domain one.
Examples of storage domain magic include things like C++ vtables or,
generally, object/value storage layout as (not)defined by most modern high-level
languages. Time domain magic is represented by rather non-trivial run-time
functionality needed to make constructors, destructors (if present in the
language) and exceptions work. Garbage collectors also belong to the category
of time domain magical things, and in some more advanced cases, this epithet is
not even slightly exaggerating.

On the other hand, there are plenty of compile time constructs that new
language can and should support. Functional classes (interfaces), type
inference, patterns (or templates), name spaces - all these allow development
of most complicated programs without the need to rely on any complicated and
non-obvious run-time requirements. Partial application and closures are also
a necessity, to the extent possible (the limitations are those imposed by
stack based activation record mechanism, as apparent in C/C++ implementation
of this feature).

*************************************
Type system and basic data structures
*************************************

On the most basic level the language should possess exactly one data type,
which I shall call ``byte type``. This data type represents smallest
individually accessible physical storage location and has a size of 1.

Byte type values can be composed into combinations of three basic constructs:

# ``Array`` densely packs set number of values in consecutive locations of the
  physical storage. Short array of bytes can be used to represent all the basic
  numeric types, apart from other useful things.
# ``Tuple`` (or structure) defines a value composed of several differently typed
  values, possibly named. It also affects name resolution rules in functions
  taking it as argument.
# ``Variant`` (or discriminating union) represents a value which can be of
  one of the data types defined in the variant's declaration. Unlike C unions,
  it can not be assigned as one data type, then taken as another. Rather, it
  "remembers" the type it was assigned to last time.

It is expected, that these constructs, no matter how intricately interposed,
should have a zero size if now actual byte values are present in their
definitions, serving only for compile-time metaprograming purposes.

Functional closures shall have their own data type as well.

Just like in C++, values can be passed around as copies, as well as by pointers
or references (safe pointers, which can only be assigned during construction).

Type aliases are of "strong" nature - data types having identical structure, but
different names are considered completely different. This also implies that
array and function value types are not identical to pointers, rather they are
close in nature to C++ array<> and function<> templates.

All values have, by default, copy and literal constructors whose syntax is
different from normal function definition. The idea is to only allow
constructors, which can not fail by their definition, while still allowing
the programmer to automate, somewhat, creation and copying of values.

Destructors, which are also customarily assumed to never fail are implemented as
aptly named function variable within a tuple (with a special built in behaviour
for arrays of tuples). It should be possible to infer through static analysis if
a particular variable has destructor functions and emit a code to call
destructors in required order where needed (while this may border on "magic"
as defined above, it is of relatively innocent kind).

******************
Syntactic features
******************

To be written together with the prototype code.
