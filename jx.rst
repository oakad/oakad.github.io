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
existence: those which were designed more than 20 years ago and those built
around so called "web technologies". I shall argue, in this article, that it may
be about time to design a new windowing system, which departs from the
traditional approach by borrowing obviously successful concepts from web
interfaces. It so appears, that one step in right direction was already made
many years ago (`Sun NeWS <http://en.wikipedia.org/wiki/NeWS>`_ windowing
system), but it was not the right time, and probably, not the right
implementation approach for that system to be successful.

********************************************************************
Basic architecture and abilities of the traditional windowing system
********************************************************************

It worth remembering that graphical windowing systems were first developed when
workstations had essentially minuscule performance compared to what we used to
these days. It won't be an exaggeration to say that for a personal workstation
of the early 1980's drawing an user interface on the display was the most
demanding task, both in terms of latency and throughput. This performance issue
was there to last. Only lately, somewhere around 2005, graphical capabilities
of an average workstation started to conveniently exceed the actual demand for
graphical quality. For 25 years beforehand, windowing systems continuously
struggled for every additional mbps of throughput and every reduced millisecond
of latency. Slow and expensive inter-computer networking posed an additional
limiting factor in traditional windowing system design.

Because of the reason stated, traditional windowing systems feature a rather
messy and unusual programing interfaces. Most of the time, application
programs has no need to use them, as during the prolonged period of continuous
evolution, layers of helper toolkits were layered atop the basic interface,
hiding it almost completely. What the higher level toolkits can not do, however,
is to alter the basic traits of windowing system operation, which are fully
mandated by its foundation.

If we choose to to abstract ourselves from the particularities of the
specific programing interfaces, it will become clear that graphical applications
don't do anything too unusual. Typical such application shall interact with
two important devices: output, behaving like a :keyword:`remote memory <rm-dev>`
interface and input, of simple :keyword:`message passing <mp-dev>` type.

Drawing the output
==================

To conserve resources, traditional windowing system tried to maintain as little
output state, as possible. It worst case, only information about pixels actually
present in user visible display area was retained. Therefore, every user action
leading to change of the exposed window area requires the controlled application
to redraw the window, either partially or completely.

Getting the input
=================

While output device operation is fairly straightforward, default input device
historically evolved into a rather curious beast. First, it way too often relies
on a set of custom, used only in context of graphic subsystem, programing
interfaces. Second, it delivers an assortment of rather unrelated information
packets, which cover some aspects of user/application interaction, but not
other.

Information packets delivered through the typical windowing system application
input fall into several broad categories:

**Output state notifications**
  Packets of this type inform application of the changes in its output state.
  Output window can be resized, hidden, obstructed or suffer some other mishap,
  of which application must be made aware.

**Input state notifications**
  Also known as focus tracking. Customarily, it is up to a windowing system to
  decide which specific part of application window is expected to act as
  receiver of user input.

**Keyboard events**
  Keys, pressed by user on a keyboard have no spatial information associated
  with them. That's where input state comes handy - key presses are assumed to
  be delivered to the part of application which owns the focus.

  If there are multiple keyboard or keyboard-like devices available on the
  given systems their input messages are merged into a single stream is such a
  way as to make them indistinguishable from each other. Application, of course,
  always has an option to bypass the windowing system altogether and obtain
  its input directly from operating system interfaces, but this is complicated
  and may lead to undesirable effects, especially in unforeseen scenarios.

**Pointer events**
  Everything said about keyboard events applies to pointer events as well, with
  the obvious distinction of pointer events do have absolute spatial coordinates
  attached.

**Lots of other stuff**
  Depending on a specific windowing system implementation there can be many more
  message formats governing other interesting aspects of application/windowing
  system interoperability.

Information, pouring into the application from its input interface is not only
heterogeneous in format, but also in behaviour. Windowing system may spend quite
an effort to manage packets in the application input queue, adding, removing
and merging them as necessary to help application behave consistently and
compatibly to user expectations.

Newly developed varieties of input devices or application interoperability
patterns inevitably present a complicated problem to windowing system
developers. Additional packet formats are being added to already messy input
interface specification, and preexisting packets must be synthesised
artificially for each new packet arriving to enable older applications to work
somehow, further cluttering the input interface. Yet, many common user actions
are never implemented with the help of common windowing system input queue,
forcing applications supporting them to revert to underlying OS' generic device
access methods.

Networking
==========

Because hardware of old used for initial design of traditional windowing systems
was barely fast enough to support even a single user session properly, the whole
windowing system state, including its input and output devices existed as a
unique global object. Application loader would utilise this object to create
a context for an application being invoked, so that it will be able to
immediately start operating its graphical interface, without too much additional
hassle. In our modern age, multiple windowing system context objects can exist
on the same system simultaneously (Microsoft calls such objects
"windows stations"), yet for the above historical reasons they must be local
to the application.

Surprisingly, the necessity to maintain graphical subsystem state on the
application host turned out to be an advantage, rather then detriment. Remote
access client (in Microsoft ecosystem known as "remote desktop") needs only
to work with low level user input and graphic output information. All the
complicated activities of contents rendering and window management happen on
the application host, which retains the application state over intended and
accidental user disconnects. The only major drawback of this approach, not
counting inevitable visual quality degradation of the graphical data, is
inability to access individual remote applications on the client side like if
they were local. This is not a theoretical problem, but rather an artifact of
the particular functionality loaded upon the windowing system context object.


************************
The failure which is X11
************************

Even in the days of first graphical user interfaces, some people tried to
perceive the future. After couple of not so successful attempts, X11 was
created, to become first "naively" networked graphical windowing system.

