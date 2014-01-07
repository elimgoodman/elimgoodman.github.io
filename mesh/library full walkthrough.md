OK - I should present both the concepts involved by using a story. Go from small to big - everyday to more complex. That's how usage of the program will progress (from small company to large company).

The application we're going to create is run from the command line. It's going to help us keep track of our personal library. The program will present a prompt, and will respond to the following commands:

- add "$title" "$author"
- show all
- show by "$author"
- read "$title"
- quit

Let's open a new Mesh project. 

[picture:blank_project]

As our very first task, let's write a function that we can run that just takes input from the user and prints it right back out. Note that we're gonna be doing things the hard way at first, to give you a better idea of how Mesh works. Most of things that seem extremely tedious can be assigned to hotkeys. 

The first thing we're gonna want to do is create a function that gets run when our program gets run - the equivalent of ```main()``` in C. To do that, we're going to want to bring up the **command bar** by hitting ":". (You can think of the command bar as a REPL with special powers. Not only can you evaluate arbitrary Mesh statements, the command bar allows you to make programmatic changes to the program you're building).

[picture:command_bar_open_and_blank]

Next, we're going to type this command into the command bar:

```
	(cursor.currentPerspective).(currentPackage).(addRegion! {{Fn}})
```

And voila! A blank function appears!

[picture:empty_fn]

What just happened back there? Well, let's think of how we would describe what we just did. In a traditional programming language, the act of "creating a function" starts when the first letter of the function definition block is typed, and ends when the last letter is typed. In Mesh, the function is added in one fell swoop, all at once. We call these changes to our program **motions**. 

Motions are extremely powerful. While the actual syntax of the command isn't important right now, what you should note about it is that is simply a Mesh statement like any other, manipulating Mesh objects like any others. In other words, the *program we're building* is represented as an object in our system, and we can manipulate it with a set vocabulary of discrete transformations.

Let's now add a statement to our function. Remember, we're doing this the hard way first. Let's click on our function to give it focus:

[picture: fn_with_focus]

Next, we'll bring back up the command bar and enter this:

```
	def fn (cursor.currentRegion):Fn
	(fn.setName! "main")
```

We see that we've changed the name of our function. Next, let's designate our function as main:

```
	(fn.setAsMain!)
```

Note a few things here. First, we didn't have to create the symbol "fn" again - the command bar persists objects for you, just like a REPL would. Second, you can see that we're operating on that function just like we would operate on an object with a traditional OO language - we're using predefined methods to alter its internal state. This is how *all* manipulations to your program take place in Mesh. It might seem cumbersome at first, but with a combination of hotkeys and practice, we've found that one actually becomes faster by using motions instead of text. Additionally, manipulating the program this way allows us to leverage the full power and expressivity of the Mesh language, while also maintaining the type and state safety guarantees that the languages provides us. What this adds up to is the ability to make changes to your program quickly, safely, and ambitiously. Mesh allows you to boldly refactor where you may have not dared before. 

-----

We're now ready to add our first statement to our program. We'll bring up the command bar again and type this in:

```
	def stmt (Statement.fromString "perform (print \"hello world\")")
	(fn.statements.append! stmt)
```

[picture:fn_with_statement]

Ah, that "fromString" method looks convenient - we didn't have to create all of the underlying objects that that statement requires (such as expressions and literals). Something important to note here is that, while the Mesh program is manipulable like an object while you're editing it, it is stored on disk as text. This means that you can continue to use text-based utilities with your program, such as git and grep. This also means that we can do what we did above - serialize and deserialize Mesh objects easily. 

I'd also like to say that we won't be devoting a lot of space here to Mesh's syntax. Mesh is not very novel in that respect, and we'd much rather show you the cool stuff.

We should now be able to see our one statement in

