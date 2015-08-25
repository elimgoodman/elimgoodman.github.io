---
layout: page
title: Representing Computation
---

<link href='http://fonts.googleapis.com/css?family=Karla:400,400italic,700' rel='stylesheet' type='text/css'>

<style>
    body {
        margin-top: 60px;
        background: #F5F4F0;
        color: rgb(12, 12, 12);
    }

    body, h1, h2 {
        font-family: 'Karla', sans-serif;
    }

    h1, h2 {
        font-weight: bold;
        margin: 1em 0;
    }

    ul {
        list-style-type: circle;
    }

    p {
        font-size: 18px;
        font-weight: normal;
        margin: 1.5em 0;
    }

    ol, ul {
        padding-left: 2em;
    }

    li {
        padding-left: 1em;
        margin-bottom: .5em;
    }

    .container {
        max-width: 980px;
    }

    hr {
        border-color: rgb(12, 12, 12);
    }
</style>

##Motivation

Each day, I come to work with one overriding motivation: [I want to make my own job easier](http://c2.com/cgi/wiki?LazinessImpatienceHubris). As a web programmer, I'm constantly on the lookout for ideas, techniques, and tools that enable me to build software in a way that uses less of my time and brainpower.

There are a handful of connected questions that got me thinking:

1. Why aren't computer programs more composable? What stops me from taking part A from some program I've written, and sticking in into part B? What approaches exist to solve this problem, and what are their shortcomings?
2. Can the unidirectional dataflow model popularized by [React](http://facebook.github.io/react/) be applied more generally?
3. As a mechanism for constructing software, is there a compromise between heavy-weight, full-fledged IDEs and lighter-weight text editors? Do we have to choose?

So that's where my head is at. I'm going to design some imaginary pieces of programming technology (language, editor, and runtime) to see if we can't make something new and interesting. This is food for thought - not meant to be rigorous.

##Composability

So let's think about some issues around composability. Here are some scenarios that highlight how tough composability is today:

- Let's say I've made a really nice datatype for an address. It's got all of the fields (even weird international fields), validation rules, formatters, all that stuff. What happens when I start a new project? How can I suck all of that stuff out, cleanly, and put it into my new project?
- I've got a pretty nice authentication system in my current project. Do I have to write the whole over again?


***

We should have a way of describing *everything* that a given function of executable code can do, and everything it needs to do what it needs to do.

We're going to be describing a function comprehensively in a made-up language. The syntax will be pretty verbose, but that's so you can follow along and figure out what's going on. We'll be starting realllllly basic for the sake of creating a shared foundation.

First, our function is gonna have a name:

<pre>
function myfunction {
    //statements
}
</pre>

Our function will also take *parameters*. We want to know what bits of information need to be provided to function in order for it to do its job:

<pre>
function myfunction(param1, param2) {
    //statements
}
</pre>

Because we're trying to specify the inputs and outputs to our function in as much detail as possible, we want to describe the *types* of the parameters:

<pre>
function myfunction(param1 : String, param2 : Integer) {
    //statements
}
</pre>

Our function will probably also *return* a value to its calling context. In that case, we want to specify the type of that value:

<pre>
function myfunction(param1 : String, param2 : Integer) : Integer {
    //statements
    return 1;
}
</pre>

This should all look pretty standard.

So, now, what if our function can fail in some way, and not return a value of the expected type? Let's take a page from Java's playbook:

<pre>
function myfunction(param1 : String, param2 : Integer) : Integer throws UnexpectedArguementException {
    if(param2 == 1) {
        throw new UnexpectedArguementException("I wasn't expecting 1!!");
    }

    return 1;
}
</pre>

Ok, still pretty familiar. But getting kind of verbose.

So now let's wade into unfamiliar waters. What if our function does something like this?

<pre>
function myfunction(param1 : String, param2 : Integer) : Integer throws UnexpectedArguementException {
    if(param2 == 1) {
        throw new UnexpectedArguementException("I wasn't expecting 1!!");
    }

    return countRowsInDB();
}
</pre>

Hm. The *signature* to our function (ie, the bit that describes what the function takes, returns, and is named) no longer comprehensively describes what the function needs to do its job. Specifically, the function is calling some other function, which is getting some data from the outside world (in this case, some database).

Let's set ourselves a new constraint - *we want the signature of any function to completely and unambiguously specify all of the possible inputs to a computation, and all of the possible results of the computation*. Our current signature is currently failing the test - it doesn't specify that we need values from database in order to complete the computation described within.

Just for fun, let's make the problem worse before we make it better. What happens if we do this?

<pre>
function myfunction(param1 : String, param2 : Integer) : Integer throws UnexpectedArguementException {
    if(param2 == 1) {
        throw new UnexpectedArguementException("I wasn't expecting 1!!");
    }

    numRows = countRowsInDB();

    if(numRows > 0) {
        deleteTheWholeFrigginDB();
        return 1;
    } else {
        return 2;
    }
}
</pre>

Now, in addition to our parameters not adequately describing the inputs to our function, the return type (and throws clause) don't adequately describe the possible results of the computation: a possible result is that we *delete the whole friggin' database!* And that's nowhere in the function signature.

Based on the above examples, here's a good taxonomy of the kinds of things that need to appear in a well-formed signature:

- the name of the function
- the return type
- all of the parameters, with their types
- if the function can fail, and what sorts of failures it can produce
- if the function needs some values from the outside world, and the types of those values
- if the function can affect some changes on the outside world, and the nature of those changes

Wow - that's a lot of stuff!

##Why does any of this shit matter?

Function signatures are important because if the computer can unambiguously determine know these properties unambiguously, it can actively help us in our quest to compose computer programs out of smaller pieces: it can help us know if all of the pieces are going to fit together correctly. They're also extremely useful to humans who might be reading the code - they're essentially functional documentation.

##What about Haskell?

Haskell is one language that attempts to tackle this issue head-on. From what I understand, Haskell attempts to cram all of the stuff outlined above into one completely comprehensive function signature. Our signature would look like this:
{% highlight haskell %}
myFunction :: String -> Integer -> IO Integer
{% endhighlight %}

Specifically, Haskell tries to force all of the messy information about side-effects and external resources into the type system. Haskell's lineage of [referential transparency](https://en.wikipedia.org/wiki/Referential_transparency_(computer_science)) and [the lambda calculus](https://en.wikipedia.org/wiki/Lambda_calculus) means that functions, once called, can be replaced by their resulting values. This approach dictates two things: 1. a function must take in everything it needs through parameters, and 2. the function must produce one (and only one) value, which is substituted for the function call.

There are drawbacks to this approach, but how about you just take my word for the fact that I think that this isn't the most intuitive way to address the issue. Cool? Cool.

##Fuck the lambda calculus.

In every language I'm aware of, functions are seen as the fundamental unit of computational composition. This, I think, is an artifact of the aforementioned lambda calculus. Also, they're a pretty easy abstraction to understand and reason about. But, as we've been exploring, the idea of a function that reduces values to other values is pretty limiting, and when I use a purely functional language, I have to jump through some large conceptual hoops to build applications that actually map to my mental models of how computer systems operate.

So what's the alternative?

Well - bearing in mind that the purpose of functions is to serve as discrete units of computation that can be composed into larger, more complex computations, what if we imagined each "function" as a miniature computer, which we composed into **networks** that yielded more complex computational behavior?

Let me lay out a model for how we can imagine computers, in a purely abstract sense. Then, let's see if we can create programs not out of functions, but out of networks of tiny imaginary computers (or as networks of networks of tiny imaginary computers, and so on).

##Imaginary machines.

Here's my definition of a computer: **a computer is a device that reacts to stimulus from the outside world. Based on that stimulus and the computer's internal state, the computer either changes its own internal state, affects the state of the world in some way, or both.** Based on that definition, let's define the parts of our abstract computer:

- **Emitters**: these are the bits of the computer that have the power to kick the computer into action - the *stimulus* described above. For a physical computer, these are the input devices of the machine (keyboard, mouse, etc.), as well as the internal clock.

TKTKTKTKTKTK

##Isn't this just Smalltalk?

Yes, to an extent. Smalltalk pioneered the idea of the computer program as a recursive network of smaller, simpler "imaginary computers." However, Smalltalk objects had no idea about types or correctness, and important bits of state ended up distributed throughout the application. If we want to stick to our original goal of making program composition as easy as possible, Smalltalk's dynamic model makes it very hard for the computer to tell us if our pieces "fit together" correctly.

So then, the question becomes: **how can we marry the helpful mental model of Smalltalk with the rigor and safety of Haskell?** My answer: look to React.

##Event sourcing, unidirectional dataflow, and React

React has become super fucking popular, and with good reason: it's fast, it's easy to learn, and doesn't utilize *too* much magic. React's popularity has also made the [Flux architecture](https://facebook.github.io/flux/) pretty popular. For myself, learning Flux was an eye-opening moment - because of the way React was structured, we could only have data flow in one direction in our applications, and because data only flowed in one direction, our applications became much easier to reason about. //In other words - **a breakthrough made at a low level enabled us to leverage new high-level metaphors, which made everything easier to grok.**

[This article](http://www.confluent.io/blog/turning-the-database-inside-out-with-apache-samza/) also made a huge impact on my thinking. Was there a way to leverage the concept of event sourcing to "port" the simplicity and grokability of Flux to different kinds of problems, beyond just UIs?

##Deep breath; recap.

Reiterating the initial goal: we want to figure out a way to make it easier to compose computer programs out of smaller, simpler parts. The "network of imaginary computers" metaphor pioneered by Smalltalk is promising, but is a little too permissive. We want to computer to tell us when things don't fit together; we want everything to be "a little more Haskell". The concept of event sourcing is also promising. Can everything be smushed together into something cogent?

##Something cogent.

Let's do two things to move this mass of ideas


TKTKTKTKTKTKT: **There are different "kinds" of functions/computational units**
