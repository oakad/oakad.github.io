######################################################################
Language for system programing, or C is not going to stay here forever
######################################################################

*This article is still being written.*

**********************************************************
What is really needed from the system programing language?
**********************************************************

There are several important traits that define a certain programing language
as "system" one. First, some important limitations of such language can be
stated:

1. System programing language must have a minimalistic, or even "nil" runtime
   component. It should be practical to write programs in such a language which
   can be run on "bare iron". This also implies seamless integration with
   platform level assembly code (through appropriate assembly inlining
   features).
2. The above implies, that complex run-time or storage domain behavior can not
   be part of language specifications. Layout of compound variables must be
   exactly known to the application implementor (up to the usual alignment and
   padding details); non-local transfer of control (such as exceptions) would
   also be unsuitable. As "system" applications are typically developed for all
   kinds of restricted and customized environments, mechanisms related to memory
   allocation and concurrency control must necessarily be provided by user side
   libraries, rather than by language specifications.

Now, that limitations are identified, desirable features of a new system
language can be examined:

1. Fully featured system for "type metaprogramming", including built-in
   compile-time introspection facilities (analogous to Boost.Fusion
   functionality).
2. Fast compilation, especially considering the need to handle complex generics.
3. Compact target binary format; this applies both to executables and loadable
   libraries. Support for compact loadable libraries poses an interesting
   problem due to the presence of the complex type system.

The demand for fast compilation is not a mere fancy. Rather, it is expected that
future software system will become increasingly complex, while supporting
increasing number of platforms and revisions. Considering the needs of such as
important development technique as "continuous integration", as well as
increased productivity of developers when the usual "compile - test - modify"
cycle can be performed rapidly, it is clear that issues of compilation speed
must be given a foremost importance.

