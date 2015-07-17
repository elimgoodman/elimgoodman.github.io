---
layout: page
title: Representing Computation
---

<style>
	ul {
		list-style: disc;
		margin: 24px;
	}
</style>

<div style="margin: 36px 0;">
<h1 style="margin-bottom: 0;">Representing Computation:</h1>
<h2>A fun topic that everyone loves</h2>
</div>

I've been thinking a lot about something recently: the *fundamental metaphors of programming.* Or put another way - the answers to the question: *what is a computer program*? Different programming philosophies and the languages that embody them answer this question in a variety of ways, but I don't think any of them are totally right. 

A good model of computation should be:
- universal - should allow for all kinds of computations
- comprehensible, and conversely, analogous. It should sort of map to things that we understand intuitively from the real world, even if the conception is highly abstract.
- composable - the metaphor should allow


A computer program is a virtual artifact that responds to stimulus.

What do we mean by "stimulus"? 

Keyboard presses, mouse movements, web requests, changes in the weather. Stuff happening in the real world.

Aha - all of that stimuli can be represented as *streams of discrete state change events*. ""The mouse moved 2 pixels to the left." "The escape key was pressed." The *state* of the real world changed in some discrete way.

So, let's refine our definition:

A computer program is a virtual artifact that responds to *streams of discrete state change events*.

(Let's use the term **transition** to stand in for "discrete state change event", and the term **signal** to stand in for "streams of transitions.").

So:

A computer program is a virtual artifact that responds to *signals*.

OK - so what do we mean by responds?

Computer programs only exist to *effect* change in the real world. They respond to changes in the real world, and they create changes in the real world. *"Side effects" are central to the purpose of all computer programs.*

We can think of all of the changes that a program might make to the outside world as more streams of transitions. We can imagine the database writes as one stream, the log file writes as another, and so on. So given that, let's again refine our definition:

A computer program is a virtual artifact that transforms *streams of transitions into streams of transitions.*

Or:

A computer program is a virtual artifact that transforms *signals into signals*.



Or:

```[Signal, ...] => {program} => [Signal, ...]```

<!---
OK - this seems better, but it's not the whole story. Now - you could definitely make a program that conforms to this definition. For example, if you had a program that just echoed every key that the user pressed to the screen, that would fit this definition. So - this definition holds for programs that are *pure* - which is to say, programs that are guaranteed to produce the same output, given the same input.
--->

That seems pretty good. Let's now unpack what's inside of ```{program}```.

<!---
One element that will almost certainly be a part of our program are bits of code that transform values of one type into values of another type. Let's refer to these as **transformations**. These are bits of code that are guaranteed to produce the same output when given the same input, don't affect the outside world, and don't have side effects on the running program.
--->

Our program might need to read information from the outside world in course of running. While we already have information from the outside world coming into our program (in the form of streams of transitions), there's also information that we'll want to acquire on demand. For example, we might need to read the contents of a file, or perform some database queries. Let's refer to aspects of the world that are readable on demand as **resources**. 

