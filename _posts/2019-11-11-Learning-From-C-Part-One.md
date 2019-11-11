--------------------------------------------------------------------------
layout: post
title: "Learning From C Part One: What I learned From making a simple UDP Calculator Client and Server"
---------------------------------------------------------------------------
Welcome to my series chronicling the hard won lessons I have received while
attempting to do things (read: school projects and side projects) in the
programming language of the gods, C. Why am I doing things in C, instead of some
newfangled language like python or in java like the University spent two years
teaching me? Simple. I like to make things difficult for myself, and I like to
learn. I do not want to stay a simple programmer who strings things together
with little underlying knowledge, and C is a great way to get an appreciation
for what the machine is actually doing, before delving deep into the abyss that
is assembly. Also, C is a great teacher. C will teaches you more than any other 
programming language ever will. By teach, I most certainly mean break everything 
you try to do until you do it the way that C wants, and even then you can still
learn much about how to self check your code when the nice C program you just
cleanly compiled, does exactly what you told it to do, and proceeds to lock up
you laptop for thirty minutes.

In this first installment we are going to be covering the lessons I gleaned from
my last school project, which was a UDP socket server and client that
implemented a simple calculator. This simple program sends a structured request
for a client (In the form of a byte array, that we had to encode and decode
ourselves), the server performs the calculation detailed, and sends a
structured response back. All in all, a simple project that I had already done
in other languages, so how hard could it be? Very hard, as it turns out, but
very instructive as it turns out. What follows is three of the lessons learned,
not in any particular order. 


## Lesson One: You should have payed attention when they taught you about binary

This was arguably the most important lesson that I learned. Notice that I did
not say "Lesson: how binary actually works" because that is not what I learned,
not from C at least. C just taught me that having a solid grasp of what how
binary actually relates to the numbers you are using, and the operations you are
performing on them helps you to debug lots of logic errors that seem mystical
when looking at them. For example, part of this project involves implementing a
packet defined by the professor. The specifics of this were fairly simple, the
request packet was an array of 8 bytes, with the last 4 indices containing the
operands for the calculation (two short integers in big endian order). Simple
right? In theory yes, and in practice yes, if you know what you're doing. I did
not at first. So, I merrily implemented this structure using `char outgoing_packet[SIZE]` as my variable because char ==== byte in size. The problem may
already by apparent to veteran C programmers, but it did not become apparent to
me until late into project, nearer to the due date that I would have liked. The
issue is that char is signed, so when I load bits into indices, C (and CPU)
do exactly what I told it to do, and represents the numbers as signed. Which
means, 127 as a short (0000 0000 1000 0000) no where near the top end of short,
but when you attempt to do outgoing_packet[LSB_index] = 127 what happens? Well
a short is much larger than a char, so C puts the eight least significant bits
into it and does not even complain, since char and short are both integers. So
outgoing_packet[LSB_index] is not 1000 0000 ?= 127 right? Nope, because, as I
was taught at some point between freshman year and now, char is signed, in two's
complement no less. So this is actually the lowest end of the char range, i.e.
-128. And this looks weird when debugging, because it looks like an overflow,
and it is one, the issue is when you pull the number back out you get all sorts
of weird behavior, as C desperately tries to maintain that shiny new negative
sign on the other side of the socket. The problem was eventually solved when a
friend pointed out (very patiently) what the behavior actually was, and I
replaced `char` with `unsigned char` and let the mathematics handle signage when
necessary.


## Lesson Two: Documentation is only helpful when you read it

What does function x do? A question we all ask from time to time, and this
project was no different. Turns out, if all you do is look at arguments and
return values it really is basically the same as randomly typing until you get a
clean compile. For example sendto for example actually does quit a bit, and
those arguments mean things beyond "hur dur make thing work." I know this seems
obvious to everyone other than past me, but you have to actually read the man
page about a given command, or the source documentation if you want to actually
use it in any meaningful way.


## Lesson Three: Do not fight your build system.

Your build system is your friend. It also hates you and will break everything if
you do not treat it well. For this project, I was using cmake to generate
makefiles, and building that way. This has several advantages over just pure
make or relying on a built-in build system from an IDE. For one thing, pulling
in external libraries is nice, and so writing modular code is fairly simple, and
baking in unit tests with ctest. However, cmake wants things set up the way that
cmake wants them, and if you try and do things your way you might just end up
breaking everything forever. Mostly, just putting the configuration files where
cmake wants them, as well as understanding your own directory structure and
headers. Once you stop trying to force your build system to do what you want,
and instead work with it your life is so much better.


All in all, this project was a wild ride and I glad it is over. But I am about
to start the same thing with TCP. I'll be back afterwards with some more hard
won lessons about segmentation faults and what memory you actually own and the
like them. There will be other lessons too I am sure.
