---
layout: post
title: Lambda Days 2017 - The First Day
categories: [tech]
---

The day kicked off with a Keynote from John Hughes, who provided an
energetic introduction to functional programming, recapped some of its
history, and explained [why it matters][WhyFP], along with Mary Sheeran.

Perhaps the most surprising talk was by Sydney Padua, a cartoonist and
animator - an unlikely presenter for a Computer Science related
conference. She talked about how a one-off comic, portraying Charles
Babbage and Ada Lovelace, turned into a series, and eventually evolved
into a very informative graphic novel.

![Analytical Engine]({{ site.url }}/assets/sydney_padua_econ2_008.jpg)
*[Lovelace and Babbage Vs. The Economy, Pt 2][BabbagePt2]*

As Padua made more comics, she learned more about the Babbage's
Difference and Analytical engines. While a working Difference Engine
exists, in the Computer History Museum in Mountain View, CA, there is no
Analytical Engine to use as a reference. Padua ended up recreating the
Analytical Engine in 3D Modelling software using technical diagrams as
reference, and even animated parts of the engine. Seeing the intricate
detail and huge scale of the engine was extremely impressive, especially
considering how little memory and computation power it would have had,
compared to modern technology.

With a majority of the day left, it was time to delve into some
technical talks. Unfortunately these were found wanting. Perhaps my
choice of tracks was poor, as I avoided the Research track for fear of
everything going over my head. The talks I attended were mostly a mix of
basic FP concepts or marketing/recruitment material under the veil of
a "How we solved X" type presentation.

There were a few talks which stood out, such as "Free the Conqueror!" by
Tamás Kozsik, "Implementing Event-Driven Microservices Architecture
using Functional programming" by Nikhil Barthwal, and "Keeping the flow
going: Data-flow oriented workflow systems" by Annette Bieniusa.

Tamás Kozsik discussed a tool he is working on which refactors Divide
and Conquer algorithms to be better suited for parallelisation, using
static analysis. This is done by normalising such functions into five
distinct parts;

 1. Check if the current element is the base case,
 2. If so, solve the case,
 2. Otherwise divide the search space
 3. Map the solve function over each half,
 5. Combine the results

This normalised form lends itself to easy parallelisation as each leaf
can be solved in parallel.

Microservices are all the rage at the moment, and it seems FP is gaining
traction. Nikhil Barthwal discussed how [Jet.com](http://www.jet.com)
combined the two concepts, building their infrastructure completely on
microservices and F#. He drew parallels between FP and microservices,
explaining that there can be pure and impure microservices (loosely
speaking, e.g., logging isn't considered impure), microservices
typically don't have state, but rather react to events - similar to how
immutable data is a collection of states. These features makes
parallelising microservices fairly straight forward. Unfortunately he
didn't elaborate much more than that.

In her talk, Annette Bieniusa described a method of modelling tasks
(forms and actions) using Kleisli Arrows, which [John Hughes discussed in
a paper][Arrows]. Not having been around academia much, it was
interesting to hear John Hughes ask questions about the concept.

Attending [Lambda Days 2017][LambdaDays] has reminded me how amazing
Functional Programming is. Listening to some iconic people in the
field, such as John Hughes, was really inspiring. The amount of
attendees was comforting, showing how popular the paradigm is becoming.
If you're not writing applications in a functional language, you're more
than likely using a significant amount of FP features regardless.

[2DGoggles]: http://sydneypadua.com/2dgoggles/
[Arrows]: http://www.cse.chalmers.se/~rjmh/afp-arrows.pdf
[BabbagePt2]: http://sydneypadua.com/2dgoggles/lovelace-and-babbage-vs-the-economy-pt-2/
[LambdaDays]: http://www.lambdadays.org/lambdadays2017
[WhyFP]: http://www.cse.chalmers.se/~rjmh/Papers/whyfp.html
