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
become such a successor, had essentially failed in this task. D programing
language (posed as a new alternative to C++) had managed to do the same, even
before gaining any sort of widespread adoption

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
stack based activation record mechanism, as apparent in C++ implementation
of this feature).

Additional commonly neglected aspect is the ability of the language
infrastructure to control information transfer between development and run-time
environments. Size of binary programs and their respective environments matters
even in our days of abundant memory capacities (because smaller size invariably
means higher density, which is a very valuable and difficult to conserve
physical quality). There also exists a necessity to strictly isolate the
interface definitions from their actual implementations, commonly neglected in
modern programing language design, where implementation code often acts as a
sole interface defining entity.

In this article I will endeavour to collect useful thoughts about a somewhat
different programing language, I shall call *Chi* (like the X-shaped Greek
letter). I do not aim for similarity or code level compatibility with C, but
want to try and refine the ideas, which make C and C++ languages to be of
such incredible use in low level application development.

As opposed to C++, I do not want to retain any features of so called classical
object-orientedness, opting instead for functional classes approach. After all,
real-world objects do not have intrinsic notion of built-in operations, unlike
the software ones. Common chair can be moved around the room not because of
some "inherited method", but only because somebody wishes to move it, it isn't
screwed down to the floor and weights only couple of kilogrammes. Those listed
properties can be set together as a functional class applicable to the
chair-like values (and many other things), thus providing a useful alternative
to C++ style objects.

On limited usefulness of exceptions
===================================

Venerable and useful ``goto`` statement (unconditional transfer of control) had
long been seen as an embodiment of evil of some sort, which must be eradicated
in any cost. Curiously enough, in modern programing languages alternatives
offered often end up being similar in their features to the
`comefrom <http://en.wikipedia.org/wiki/Comefrom>`_ construct, which is even
worse. One notable example of such misuse are labelled statements in Java,
whereupon the label is specified at the beginning of the block, while control
is most often transferred past its end. Exceptions happen to suffer a similar
ill fate.

It must be noted, that exceptions come in two rather distinct varieties.
Borrowing the Java terminology, they can be either checked or unchecked, the
difference being that any function desiring to throw checked exceptions must
explicitly list them in its declaration. No such restriction applies to
unchecked exceptions, which can simply pop-up from elsewhere at any time.

It is immediately clear, that checked exceptions are not true exceptions at
all, so to say. Rather they can be treated as alternative return types for
a given function. Discriminated unions were long neglected by mainstream
programing languages, yet checked exceptions essentially result in function
returning such an union of its normal return type with some additional exception
types listed in the declaration. There appears to be no saving in amount of
typing or clarity of the resulting code when using checked exceptions versus
Haskell-style Maybe return types or plain C pointers employing a similar
adaptation (pointers can seldom access their full numeric range worth of
storage, so an arithmetic sub-range can be set within possible pointer values
to indicate various cases of failure).

Unchecked exceptions present a more interesting problem, however. They can be
emitted by functions which don't have a user accessible return value at all,
object constructors and destructors of C++ being a prominent example. They also
serve to provide a variety of non-local return from a throwing function,
delivering their value somewhere deep in the call stack, instead of a simple
return to the caller. To better assess a usefulness of these features to a
system level language, a short enumeration may come handy:

#. Throwing exceptions from constructors requires considerable caution in
   non-garbage collected languages, often resulting in memory and resource
   leaks. Considerable local error checking is almost inevitable in non-trivial
   cases to the extent, that its easier to just normally finish construction of
   semantically invalid object with an additional post-construction check for
   integrity (very common pattern in C++ programs).
#. Throwing exceptions from destructors appears to be an exercise in futility.
   Fortunately, its almost never done or needed.
#. Non-local error handling, or throwing exceptions from arbitrary places in the
   program is advocated on the basis that exceptional conditions are not
   supposed to be recovered from. Yet, high reliability software systems do
   commonly have an ability to recover from almost all varieties of errors
   they may encounter, short of physical hardware damage. Under the assumption,
   that errors are not too rare and must be recovered from as quickly as
   possible, local error handling (C style) appears to be more useful approach.

Nevertheless, in past years, exceptions became a standard programing construct.
It seems beneficial to implement them in Chi as well, albeit in such a fashion
as to afford easy switching to exception-free code. The most obvious way to
do so is to abolish the need for customary ``try`` - ``catch`` construct.
Instead, compiler will implicitely promote any scope immediately followed by
one or more ``catch`` statements to become exception catching. In case,
compiler is instructed to disable exceptions, such ``catch`` statements can
turn into noops, optionally rising compilation warnings or errors.

Chi programing style guideline is not expected to emphasize the of exceptions
for error recovery (unlike many other languages).

*************************************
Type system and basic data structures
*************************************

On the most basic level Chi should possess exactly one data type, which I shall
call ``byte type``. This data type represents smallest individually accessible
physical storage location and has a size of 1. Byte type is also a precursor
to all numeric types and has all the traits of range limited integer.

Byte type values can be composed into combinations of three basic data
structures, all customisable with type patterns:

#. ``Array`` densely packs set number of values in consecutive locations of the
   physical storage. Short array of bytes can be used to represent all the basic
   numeric types, apart from other useful things.
#. ``Tuple`` (or structure) defines a value composed of several differently
   typed values, possibly named. It also affects name resolution rules in
   functions taking it as argument. Special forms of tuples are nameless
   tuples, which bind their member names directly into an enclosing scope and
   extensible tuples, which can act as extentions to other values.
#. ``Variant`` (or discriminating union) represents a value which can be of
   one of the data types defined in the variant's declaration. Unlike C unions,
   it can not be assigned as one data type, then taken as another. Rather, it
   "remembers" the type it was assigned to last time.

Additional important data type present at the Chi's basic abstraction layer
is functional closure, which doubles as a general function type.

Functional classes
==================

Any type can be made conformant with one or more functional classes, which
define a set of operations, applicable to the given class. This is similar
to the Haskell classes or Java interfaces.

In particular, pointers and references are really types which can be passed
as arguments to appropriate dereferencing operator. For couple of integer types
this dereferencing operator is defined by the compiler built-in instances (to
the tune of generic ``addressof`` operator).

Type attributes
===============

Any type can have a set of associated attributes, controlling code generation
and run-time behavior of the corresponding value. Same type with distinct
attributes can be seen as belonging to the same family and Chi should provide
means to automate promotion and conversion of such related types.

Of attributes, those appear to be definitely necessary:
#. ``any`` (stripping of all attributes for use in type patern).
#. ``const`` (read-only values).
#. ``volatile`` (restricted optimization).
#. ``packed`` (dense storage utilization).
#. ``atomic`` (similarly to Java ``synchronized``, extends type with
synchronisztion constructs or generates appropriate code for it).


Type calculations
=================

Chi compiler will provide both the ability to synthesize arbitrary types out
of type patterns by evaluating special programs, taking type names as arguments.
Results of the above evaluations will represent composite types with specific
storage requirements, which can be instantiated and used by user code.



*******************
Compilation process
*******************

Considerable deployment flexibility attainable by using C/C++ in software
development stems from the rather unusual three stage compilation process
employed by the languages. Only the middle stage of the process is controlled by
the core language syntactic constructs. The other two are managed by the special
mini-languages introduced by the supporting tools. To summarise, the steps taken
to build a C application normally look like this:

#. Preprocessor inspects the source files looking for certain recognised
   statements. Among those, include statements are used to pull in additional
   source files, the result being a transformed C language program having no
   external source level (interface) dependencies. A set of all preprocessor
   statements forms a rather complete programing language on its own.
   Unfortunately, in some cases, this language is not up to the task, so
   additional program transformation may need to be performed by some custom
   tool, before the standard preprocessor can kick in. Whether this can be
   avoided in every case is debatable, but observable fact is that many large
   and complex programs do rely on such multi-level preprocessing.
#. Compilation proper is performed on a transformed stand-alone source unit,
   which, at this stage, can be unnecessarily large, having all its source
   dependencies literally included. Object file is produced as a result,
   containing all the CPU instructions for the just compiled program unit, but
   still lacking means to access external run-time dependencies.
#. The set of generated object files is put together by a separate tool, called
   linker, to create the desired end result (executable program or library).
   Linker relies on a script in its own language and some additional user input
   to examine the necessary run-time dependencies and to further transform
   object files producing the final binary.

Chi strives to retain the same flexibility with the important distinction of
incorporating the preprocessor into the core language, in the form of ``const``
statements and ``macro`` construct. Any language statement resulting in an
assignment to a constant value or anonymous ``const`` qualified code blocks
must be fully evaluated by compiler, if possible, otherwise compilation error
will arise. ``macro`` construct, in turn, is a block of statements, which are
literally rewritten by compiler during the argument expansion and then copied
into the original location of macro invocation, just like the C preprocessor
function like macros. Important difference from C preprocessor, in this case,
is handling of macros by the core compiler, which retains information on their
structure and location.

Source level dependencies are also pulled directly by the core compiler.
Additional sources are pulled in by filesystem path, again in similarity with
C preprocessor. Each distinctive (as determined by its contents, not name)
source file pulled in is read exactly once, as the ability for multiple self
inclusion exhibited by C header file appears to be too cumbersome for practical
use. Extensive compile time evaluation abilities should compensate for this
shortcoming.

Besides normal source file, binary object files can also be specified as
dependencies on the source level. It is conceived, that compiler should be
able to extract necessary information directly from the extended symbol table
information optionally present in the binaries, to reduce source level clutter
in programs with extensive third-party dependencies. For example, interface
description in the source file may contain only generic type patterns, while
information on specific pattern instantiations will be read directly from the
link library (or its detached symbol file).

In some cases it is handy not to rely on explicit source import statements at
all. For such environments, Chi compiler will have an ability to use a namespace
to file name mapping database, such that any referenced namespace will pull in
the necessary source dependencies and push the necessary updates back to the
database.

Considering the above considerations, Chi compilation will occur as a single
stage process, collapsing the traditional C compilation stages into a one
heavy-weight stage and abolishing the difference between include and library
file search paths. Each compilation unit will be processed into an
incrementally linked binary, possibly capable of being immediately loaded into
a JIT based application or further linked with other such binaries by means
of standard C linker.

