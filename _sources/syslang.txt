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
stack based activation record mechanism, as apparent in C/C++ implementation
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
   functions taking it as argument.
#. ``Variant`` (or discriminating union) represents a value which can be of
   one of the data types defined in the variant's declaration. Unlike C unions,
   it can not be assigned as one data type, then taken as another. Rather, it
   "remembers" the type it was assigned to last time.

It is expected, that these structures, no matter how intricately interposed,
should have a zero size if no actual byte values are present in their
definitions, serving only for compile-time metaprograming purposes.

Apart from basic data structures some additional type declaration constructs
will be necessary, to encompass the source level abstractions. Thus, just like
in C++, pointers and safe pointers (references) constitute value types with
very specific traits, dissimilar to their underlying numeric value types.

Same applies to functional closures, which normally require relatively complex
data structures at their implementation level, as well as special compiler
treatment, yet appear as simple values to the application programmer.

Type aliases are of "strong" nature - data types having identical structure, but
different names are considered completely different. This also implies that
array and function value types are not identical to pointers, rather they are
close in nature to C++ array<> and function<> templates.

All values have, by default, copy and literal constructors whose syntax is
different from normal function definition. The idea is to only allow
constructors, which can not fail by their definition, while still allowing
the programmer to automate, somewhat, creation and copying of values.
Enumerations, as well as any more general restricted value assignment, should
be possible to implement through this mechanism.

Destructors, which are also customarily assumed to never fail are implemented as
aptly named function variable within a tuple (with a special built in behaviour
for arrays of tuples). It should be possible to infer through static analysis if
a particular variable has destructor functions and emit a code to call
destructors in required order where needed (while this may border on "magic"
as defined above, it is of relatively innocent kind).

**********************
Namespaces and scoping
**********************

To control the visibility of symbol names in large programs, Chi borrows the
notion of namespaces from C++. Any language statement can access symbols
defined in any of the hierarchically enclosing namespaces. Symbols defined in
namespaces not enclosed in the immediate namespace hierarchy must be referenced
by full namespace path (fully qualified name), with shortcut notation for
local aliasing provided. Namespaces can also possess qualifiers.

**Default**
  By default, any namespace is accessible by its full path on the source level.
  However, none of the symbols defined in such namespaces is exported on the
  binary level.

**Global**
  Dynamic libraries will put at least some of their symbols into the ``global``
  namespaces. Sufficient information will be put into generated binaries to
  make those symbols accessible through dynamic loading.

**Hidden**
  Hidden namespaces are only accessible from immediately enclosing ones. They
  are useful to clearly isolate user invisible implementation from interface
  definitions.

Each stand-alone source file (compilation unit) has a local unnamed, hidden
namespace defined by default. While symbols in it are not accessible to user
code, it still can be used for ``const`` statements evaluated at compile time,
which can place their results into the visible, named namespaces. Some
implementations may choose to make this namespace visible and even put it into
some hierarchy by default - such approach is handy in interactive JIT
compilation oriented environments.

Besides this property, compilation units do not aim to impose any additional
logical structure on the program. Namespaces can span multiple source files
in arbitrary fashion, while still remaining a singular entities. Order of
symbol declarations within the namespace is irrelevant - all symbols within
the namespace are assumed to exist simultaneously.

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

******************
Syntactic features
******************

To be written together with the prototype code.

