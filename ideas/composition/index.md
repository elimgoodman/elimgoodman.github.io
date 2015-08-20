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
        margin-bottom: 1em;
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

Hm. The *prelude* to our function (ie, the bit that describes what the function takes, returns, and is named) no longer comprehensively describes what the function needs to do its job. Specifically, the function is calling some other function, which is getting some data from the outside world (in this case, some database).

Let's set ourselves a new constraint - *we want the prelude of any function to completely and unambiguously specify all of the possible inputs to a computation, and all of the possible results of the computation*. Our current prelude is currently failing the test - it doesn't specify that we need values from database in order to complete the computation described within.

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

Now, in addition to our parameters not adequately describing the inputs to our function, the return type (and throws clause) don't adequately describe the possible results of the computation: a possible result is that we *delete the whole friggin' database!* And that's nowhere in the function prelude.

Based on the above examples, here's a good taxonomy of the kinds of things that need to appear in a well-formed prelude:

- the name of the function
- the return type
- all of the parameters, with their types
- if the function can fail, and what sorts of failures it can produce
- if the function needs some values from the outside world, and the types of those values
- if the function can affect some changes on the outside world, and the nature of those changes

Wow - that's a lot of stuff!

##Why does any of this shit matter?

Function preludes are important because if the computer can unambiguously determine these properties to a complete level of precision, it can be very helpful in our quest to compose computer programs out of smaller pieces.
