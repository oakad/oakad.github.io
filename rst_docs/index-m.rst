==========================================================
The M Machine: computer architecture for the long now [*]_
==========================================================
---------------------
An exercise in design
---------------------

Modern computer architectures had all originated in the 1980s (and some even
earlier). The understanding of software system design was rather different in
those days, computer systems being still rather expensive and transistor
budgets tight.

It makes perfect sense, therefore, to try and design an architecture based on
modern advancements in system and application software design and functioning.
Such an architecture should not be incumbered by the long existing legacy
compatibility issues. On the other hand,it must be open for the challenge of
supporting rapidly evolving software system into a substantially long future.

Conventions and Definitions
===========================

MMArch
  The M Machine architecture - an entirety of logical and functional features
  offered by the M Machine.
OpSet
  "Level Zero" data cache of the MMArch, an equivalent of register file in a
  traditional processor.
L0, L1 ... Ln
  Lines, comprising the OpSet.
AT0, AT1 ... ATn
  Corresponding address tags of lines comprising the OpSet.

The OpSet: register file reimagined
===================================

What was traditionally known as register file is replaced in the MMArch by a
special level-zero data cache (OpSet hereby). In a sense, all operations
supported by MMArch are memory to memory.

- The OpSet is comprised of number of individually accessible "lines", each
  line associated with its own address tag and set of control flags. The tag
  and flags are not directly accessible by application software.
- The bit width and number of lines can vary depending on an implementation
  intent. An "embedded" processor may feature 8 lines of 64b widths with 32b
  address tags, whereupon a high performance core may come with 32 or 64 lines
  of 512b width and 64b or large address tags.
- The line is expected to always reflect the state of memory pointed to by
  address tag. As such, implementation is expected to maintain a "coherent"
  view of the data in line at all times. To perform an operation on a Line,
  processor must secure an exclusive ownership of that line and maintain it
  until the end of instruction affecting it. As with normal caches, the address
  tag is expected to be naturally aligned.
- Multiple lines may feature the same address tag (as seen by application
  software). In this case, instruction operating on perceptively distinct
  lines, will actually be operating on the same line via distinctive aliases.
- The Line is merely a collection of bits. The actual interpretation of the
  meaning of those bits is up to the processing unit engaged by the instruction
  being processed.

Instruction set
===============
The instruction set of MMArch follows the CISC philosophy:

- Variable size of encoded instructions
- Consistent encoding format at the slight expense of density in favor of
  better long term extensibility

To this end, an encoding based on Even-Rodeh [*]_ self delimiting code is
employed. A slight penalty in decoding speed and code density is offset by the
virtually unlimited extensibility of the proposed scheme.

The general format of instruction in memory is as following:

- ER coded instruction length in bytes minus one
- ER coded opcode
- Fixed bit width or ER coded arguments to instruction, first to last

  - Line ids and immediates are always ER coded
  - Instruction modes and modifiers can be fixed width

- Zero bit padding to full byte

The proper number of supplied arguments as well as their interpretation (be it
an immediate value or OpSet line number) is established based on opcode. If
any such values exceed limits supported by the implementation or otherwise can
not be processed, the illegal instruction handler will be invoked, possibly
providing a software based decoding.


.. [*] Not related to the Long Now Foundation
.. [*] Preliminary. Some other self delimiting coding may be found more suitable.
