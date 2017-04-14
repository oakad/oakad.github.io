#################################################
Brief discussion of input/output interface traits
#################################################

Despite the considerable number of different programing interfaces, which are
present in the popular and not so much operating systems, there are,
essentially, less than two real ways to perform input/output operations. It
seems worthwhile to analyse these methods briefly, as a useful exercise and a
reminder.

First, we need to establish, what does I/O operation means. In its simplest
form, I/O corresponds to retrieval of the information which can not be obtained
through local memory access, or, in other words, fetched by short enough CPU
command sequence on behalf of currently executing program. Of course, really
useful information almost never can be found locally (this is, undoubtedly, one
of the greatest creation mysteries), so any non-trivial program devotes major
parts of its code and run time to the matters of information I/O.

As a side note, I happen to think, that such magnificent programing languages
like `Haskell <http://en.wikipedia.org/wiki/Haskell_(programming_language)>`_
and similar non-procedural languages have not much chance of being widely
adopted. And this is not because of some perceived learning difficulties, but
simply owning to the unfortunate fact, that I/O functionality necessarily looks
very cumbersome when implemented in them. It's not like procedural languages do
it so much better, simply correct I/O as it is already presents so many
challenges, that fitting it into additional language mandated abstract form does
it no good service as all.

.. _mp-dev:

*************************
Message passing interface
*************************

The most basic and ubiquitous I/O communication pattern is a message passing
interface. Messages are nothing more than finite sequences of bits exchanged
between local and remote parties. Even disregarding computers outright, we
may notice that oral or written conversations between humans still can be
described in terms of message passing. Nature knows some other ways to
communicate (chemical trails come to mind) but those have not made it into
computer engineering just yet.

Given a need to engage in information exchange with a remote party we can do
three basic things:

#. Establish the identity of the remote party using some form of address
   literal.
#. Prepare a message by putting some bits together and send it over.
#. Designate an area of local memory to serve as mailbox and check it every now
   and then to see if something (remotely originated message) had dropped in.

Surprising, as it may be, there's not much else to do, communication wise.

Atop of the basic pattern some convenient extensions are always possible.

**Blocking wait**
  When waiting to receive a message, application can ask an operating system
  to poll the mailbox on its behalf and conveniently do nothing in the meantime.
  This is, of course, mostly useful on time-sharing operating systems.

**Request/reply**
  Sometimes, the remote party will do something only if asked to. Single
  interface entry point can be used by application to format a request message,
  send it, wait for remote party to reply and return the reply as a result of
  seemingly single operation. Ordinary file access on most operating systems
  works roughly like this (it's worth remembering how complex modern storage
  devices actually are).

**Streaming**
  Sometimes, I/O interaction is described in terms of continuous data stream.
  However, streaming normally means rearranging (cutting and pasting) messages
  to form other messages out of them. As some editing of messages in always
  necessary, there appears to be no need to give streaming pattern any special
  distinction.


.. _rm-dev:

***********************
Remote memory interface
***********************

Some remote parties happen to possess considerable amount of memory, taken in
most broad meaning of the word. It can either be semiconductor memory, similar
to local one, or magnetic medium, or even an optical imaging device. At any rate
it is very convenient to make this remote memory available to an application as
if it were a local one. While impossible in general (moving information around
requires time and effort) such memory mapping can be achieved in standardised,
application independent way, whereupon actual message passing can be delegated
to operating system or even hardware (as it is done on
`NUMA platforms <http://en.wikipedia.org/wiki/Non-Uniform_Memory_Access>`_).

Normally, remote memory access is an additional capability of otherwise usual
message passing interface. Unless a lot of hardware magic is employed, operation
of the remotely shared memory requires explicit synchronisation of interested
parties by means of predefined message exchange protocols. Therefore, when
mentioning remote memory interfaces, it is implied that those interfaces can be
operated like a pure message passing ones.

`General comments, corrections, objections? <http://github.com/oakad/oakad.githu
b.com/issues#issue/2>`_
