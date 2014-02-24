---
layout: page
title: Mesh
---
{% include JB/setup %}

#A Structure Editor

Here are some screenshots of a structure editor prototype that I put together a little while ago. The general idea was to represent all elements of both the program itself and the editor configuration using a uniform, declarative form. Given this uniform underlying representation, the user could then be allowed to modify how those structures appeared and were operated on, using some additional information provided in the same format. I also really wanted each block to have a well-defined set of mandatory and optional fields with associated types, as I find that this kind of "fill in the blanks" style reduces my own cognitive load a great deal, and allows the editor to offer more assistance.

As a small example, here's how a very simple function might be represented:

![image](http://elimgoodman.com/assets/mocks/output/first.png)

Because each "slot" in the structure is typed, the editor could be helpful when it comes time to add another statement in that function:

![image](http://elimgoodman.com/assets/mocks/output/first_add.png)

However, it'd be pretty hard to understand the language if all the statements looked like that. Thus, the user has the ability to change how that kind of statement is rendered (within certain constraints - we want to make sure that the resulting form is still editable with the keyboard and reflowable on different screen sizes). The user can change how the statement is displayed by creating some additional declarative structures (that's what those ```ConceptUI``` blocks are):

![image](http://elimgoodman.com/assets/mocks/output/second.png)

With enough effort, that giant clunky structure we had at the beginning can be massaged to look like something that might appear in a regular programming language (note that the necessary ```ConceptUI``` blocks have been hidden):

![image](http://elimgoodman.com/assets/mocks/output/third.png) 
