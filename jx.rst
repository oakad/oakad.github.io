###########################################################
Windowing system of the future in the mirror of the present
###########################################################

*This article is still being written.*

Graphical windowing systems (also called GUI, though, technically, GUI can
be organized around different principles) are considered to be one of the
primary components in design of modern computer systems. Windows are used to
manage output of multiple applications sharing the same output device and allow
for considerable increase in interaction bandwidth between user and computer.

Roughly speaking, there are two kinds of windowing systems currently in
existence: those which were designed 20 years ago and those built aroud so
called "web technologies". I shall argue, in this article, that it may be
about time to design a new windowing system, which departs from the traditional
approach by borrowing obviously successful concepts from web interfaces. I do
not consider, however,  the extensive use of client side automation (usually
done with ECMAScript in web applications) to be such a neat idea, so
`Sun NeWS <http://en.wikipedia.org/wiki/NeWS>`_ windowing system may rest in
peace undisturbed.

********************************************************************
Basic architecture and abilities of the traditional windowing system
********************************************************************

It worth remembering that graphical windowing systems were first developed when
workstations had essentially miniscule performance compared to what we used to
these days. It won't be an exagerration to say that for a personal workstation
of the early 1980's drawing an user interface on the display was the most
demanding task, both in terms of latency and throughput. This performance issue
was there to last. Only lately, somewhere around 2005, graphical capabilities
of an average workstation started to conveniently exceed the actual demand for
graphical quality. For 25 years beforehand, windowing systems continuously
struggled for every additional mbps of troughput and every reduced millisecond
of latency. Slow and expensive intercomputer networking posed an additional
limiting factor in traditional windowing system design.

Typical application in such system has a rather inconsistent interface to the
rest of the system. Output must be directed to the memory mapped display device,
which was quite unusual in the era before ubiquitious virtual memory and memory
mapped device support (these are not very common even nowadays). Input side of
the typical application is even more curious.

Input is delivered by asynchronous interface (so called event queue), which
still has no other direct analogs in most operating systems. In reality, this
event queue does not differs much in functionality from network packet
interface, yet it relies on a totally unrelated family of system API calls.
Additional feature of the typical input event queue is a total mess of packet
formats delivered by it. Window resize or focus notifications are interleaved
in random order with key press and mouse move information and require additional
filtering and dispatch in user code (assisted by standard libraries, but still
done by each application separately). Moreover, addition of early unsupported
types of input devices invariably leads to problems of fitting the new packet
types into existing event queue infrastructure. At last, some obviously user
interface related functionality, such as audio input/output is not handled by
event queue at all, forcing applications to operate it through "normal"
operating system interfaces, just like the rest of machine functionality.

Because hardware of old, used for initial design of traditional windowing system
was too slow to support even a single user session properly, a logical concept
of local desktop was introduced. Upon start-up, user application obtains
a fixed reference to the unique desktop object, which has input (event queue)
and output (display) devices associated with it. Application has to
perform no further work, as devices were prepared for it in advance, so it
can start operating them immediately. This particular feature was seen as
deficiency by X11 designers, but now it actually appears to be a good idea
(see below).

When evolution of the networks brought in the need to remotely access window
interfaces, it proved to be relatively easy and straighforward task to
do so with traditional systems (as there was only one way to do so). All that is
needed is to create some additional desktop objects on the host system with
their input/output devices implemented by means of special networking drivers.
Application loader will then decide what desktop object to pass to each newly
started application.

This rather naive approach works surprisingly well and has only one serious
drawback: desktop object, apart from managing input and output devices, is
loaded with additional functionality, which must be there for user application
to use. Modes of interaction with the application from the remote system are,
therefore, somewhat limited and it's very hard to make a remotely running
application behave like if it was local.

************************
The failure which is X11
************************

Even in the days of first graphical user interfaces, some people tried to
perceive the future. After couple of not so successful attempts, X11 was
created, to become first "natively" networked graphical windowing system.

