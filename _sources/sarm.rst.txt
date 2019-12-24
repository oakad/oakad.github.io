#############################################
Access rights management: slower means better
#############################################

Proper management of resource access rights in computer systems is one of the
most complicated and, one might say, convoluted problems of computer system
design. Even more so, access rights are too often managed by means of badly
designed protocols, which are prone to abuse. However, proper design of
rights management protocol is impossible without an appropriate analysis of
protocol entities involved. In fact, root causes of security failures
experienced by rights management systems can easily be attributed to
incorrect identification of foundational entities of those systems.

There exist a considerable body of publications and software dedicated to
the issues of access rights and identity management. Unfortunately, many of
these works suffer from over formalization and other common "design by
committee" drawbacks, making them very difficult to implement and use
properly. Invariably, implementational difficulties translate into future
security failures. Thus, design of a quality access right management system
should not only be formally correct and featureful, but must also pay a close
attention to "mental ergonomics" of its developers and end-users.

*****************************************
Basic traits of rights management systems
*****************************************

Prior to management any sort of "rights", one shall establish the nature of
objects and subjects, concept of "rights" applies to. In regard to computer
system, identifying the object is rather easy.

.. note::

   Just like with any system, access to a computer system always involves
   observation or modification (control) of its state variables. Following
   the accepted terminology (due to R. E. Kalman), we can formulate the
   following rule. For every access channel established with the system, two
   non-exhaustive, overlapping sets of system variables will be available: set
   of controllable system state variables and (usually larger) set of observable
   ones. These sets together form the "access rights set" (ARS) of the channel.

The issue of subjects, accessing the computer system is way more complicated.
First, it is clear, that computer system can only respond to an information,
emerging from a "near end" of an access channel. It has no material way to
"perceive" what's going on on the "far end" of the mentioned access channel and
thus must rely on "identity proofs" of various strength to establish the
appropriate ARS.
