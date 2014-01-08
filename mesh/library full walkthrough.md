TODO:

* Language foundation at the beginning
* Some sort of section on the workspace
* Unify the syntax for adding a panel and creating a new extension
* Change ExecutionContext to either Context or VM
* Digging into an object to see how it works
* I think I may have to simplify/change the language to make it easier to write a debugger - either more like Lisp, or more like assembly
	* 	God, maybe there's some assembly-like sub-language that everything gets compiled into, and is also represented as Mesh objects, and the debugger taps into that layer

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
	(cursor.getCurrentPerspective).(getCurrentPackage).regions.(append! {{Fn}})
```

And voila! A blank function appears!

[picture:empty_fn]

What just happened back there? Well, let's think of how we would describe what we just did. In a traditional programming language, the act of "creating a function" starts when the first letter of the function definition block is typed, and ends when the last letter is typed. In Mesh, the function is added in one fell swoop, all at once. We call these changes to our program **motions**. 

Motions are extremely powerful. While the actual syntax of the command isn't important right now, what you should note about it is that is simply a Mesh statement like any other, manipulating Mesh objects like any others. In other words, the *program we're building* is represented as an object in our system, and we can manipulate it with a set vocabulary of discrete transformations.

Let's now add a statement to our function. Remember, we're doing this the hard way first. Let's click on our function to give it focus:

[picture: fn_with_focus]

Next, we'll bring back up the command bar and enter this:

```
	def fn (cursor.getCurrentRegion):Fn
	(fn.setName! "main")
```

Note a few things here. First, we didn't have to create the symbol "fn" again - the command bar persists objects for you, just like a REPL would. Second, you can see that we're operating on that function just like we would operate on an object with a traditional OO language - we're using predefined methods to alter its internal state. This is how *all* manipulations to your program take place in Mesh. It might seem cumbersome at first, but with a combination of hotkeys and practice, we've found that one actually becomes faster by using motions instead of text. Additionally, manipulating the program this way allows us to leverage the full power and expressivity of the Mesh language, while also maintaining the type and state safety guarantees that the languages provides us. What this adds up to is the ability to make changes to your program quickly, safely, and ambitiously. Mesh allows you to boldly refactor where you may have not dared before. 

-----

We're now ready to add our first statement to our program. We'll bring up the command bar again and type this in:

```
	def stmt (Statement.fromString "log (print \"hello world\")")
	(fn.statements.append! stmt)
```

[picture:fn_with_statement]

Ah, that "fromString" method looks convenient - we didn't have to create all of the underlying objects that that statement requires (such as expressions and literals). Something important to note here is that, while the Mesh program is manipulable like an object while you're editing it, it is stored on disk as text. This means that you can continue to use text-based utilities with your program, such as git and grep. This also means that we can do what we did above - serialize and deserialize Mesh objects easily. 

I'd also like to say that we won't be devoting a lot of space here to Mesh's syntax. Mesh is not very novel in that respect, and we'd much rather show you the cool stuff.

We should now be able to see our one statement in our function. That having been done, let's run the sucker. While we could always run our program using the "mesh" command-line utility, that wouldn't be any fun (and plus, Mesh offers some other really cool ways to do it). We're gonna take the really long way around on this one, but bear with us. This is where the magic happens. 

We're gonna start out by just running our program via the command bar:

```
	ref stdout ""
	def vm {{VM :stdout stdout}}
	loop fn.statements statement
		statement.exec! vm
	context.stdout
```

^^The reason it wouldn't work is that (print) has nowhere to go. What happens with blocking IO? What happens when you exec "perform (getLine)!" Maybe the object you pass it needs to conform to a certain specification. ie, getLine needs a context that understands how to read from stdio. However, we could mock up a context that also just mocks up those inlets and outlets. VMs need to know at compile time which inlets and outlets are needed from. We could compute all of the inlets and outlets needed by the whole program, and then require that any statement that's executed fulfill all of those inlets and outlets. That feels sloppy - maybe could do it at the package level?

Don't worry too much about that VM stuff. What's important is that we can see there that context.stdout is equal to "Hello world". We did it!

However, that was pretty involved. What we're going to want to do is create an object that can do the running of the program for us. One thing that we haven't talked about is that not only is the *program* just a collection of Mesh objects, but the *environment* is also just comprised of instances of Mesh objects. We're going to exploit this fact to extend the editor with a new kind of object. Objects that only exist in the environment and have no bearing on the execution of the program are called **extensions**. We'll be creating an extension to run our program for us and display the output. We'll call it, creatively, Runner. Let's type this in the command bar:

```
	def extension_string = """
		Extension Runner:
			fn : Fn
			run():String ->
					ref stdout ""
					def vm {{VM :stdout stdout}}
					loop this.fn.statements statement
						statement.exec! vm
					return context.stdout
						
	"""
	
	def extension = (Extension.fromString extension_string)
	(editor.extensions.add! extension)
```

We're creating an extension from a string, just like we did above with the statement. What that syntax specifies is that we're making a new extension (named Runner), and that instances of the Runner extension will need to take an instance of a Fn. Runner extension instances will also have a method called "run" which takes no parameters, and returns a String. At the end, we're adding the extension object to the editor's set of extensions. 

Now we need to actually create a Runner. We'll type this into the command bar:

```
	def runner {{Runner :fn fn}}
	(cursor.getCurrentPerspective).(getCurrentPackage).regions.(append! runner)
```

And look at that! We see a Runner in our project, with it's function set to our "main" function. Now we can use it to actually run our code:

```
	(runner.run)
```

And the right thing came out! Yay!

However, this isn't too much of an improvement. After all, why have that runner region sitting there on our screen? We can't really interact with it meaningfully, except to change the function that it runs. Let's make the UI of that element a little more helpful. 

All extensions have a user-editable UI that they present in the project view. Let's make our runner display the output of its last run right in the region. To do that, we need to do three things: 

1. alter the Runner extension to store the state of stdout after it's been run 2.
2. Update the "run" method to update the result after running 
3. Update the UI of the extension to display that information. Let's add that field first (and again - don't worry too much about syntax):

```
	def field_type (Type.fromString "&String")
	def empty_string #(return "")
	(extension.addField! :name "result" :type type :transitionWith empty_string)
```

Now our existing Runner now has a "result" field. We can see that it's currently set to the empty string (which is what that 
```:transitionWith empty_string``` business is about):

```
	extension.result
```
Next, let's update the method:

```
	def block_string """
		ref stdout ""
		def vm {{VM :stdout stdout}}
		loop this.fn.statements statement
			statement.exec! vm
		alter this.result context.stdout
		return this.result
	"""
	
	def block (Block.fromString block_string :vm => (extension.getVM))
	(extension.run.setBlock! block)
```

(Note for the curious: The reason we pass in that :context variable is so that references to ```this.result``` evaluate correctly)

Then let's update the UI:

```
	def panel_string """	
		{{Row
			{{StaticLabel "Result:"}}
			{{DynamicLabel this.result}}
		}}		
	"""
	
	def panel (Panel.fromString panel_string)
	(extension.ui.setPanel! panel)
	(extension.ui.setFieldVisibility! :result False)
```

Tada! We now see that our Runner region has a much prettier label, and the result field is hidden:

[picture:extension_ui_updated]

And now, running our runner again should make the result appear:

```
	(runner.run)
```

[picture:extension_with_result]

Nice! But let's say we want to take this one step further - let's have Mesh re-run our Runner every time we alter that function. To do that, we're going to introduce another new idea: **triggers**. Triggers are just snippets of Mesh code that get run whenever certain motions occur. Let's make a new trigger to kick off our Runner when our function is changed:

```
	def trigger_string """
		Trigger:
			region: fn
			motion: Editor.Motions.Any
			action: #(
				(runner.run)
			)
			
	"""
	
	def trigger (Trigger.fromString trigger_string :vm (editor.command_bar.getVM))
	(editor.triggers.add! trigger)
```

(Note for the curious: that business with ```:context (editor.command_bar.getVM)``` is what allows us to refer to the symbols 'fn' and 'runner' in our trigger definition. If explicitly passed a context, the deserializer will use it to perform symbol lookups. This is a property of all deserializers, not just the one for Trigger.)

Now, let's alter our function:

```
	def new_expr (Expression.fromString "\"goodbye world\"")
	def stmt (fn.statements.first):LogStatement
	(stmt.setExpression! new_expr)
```

And there it is! As soon as we performed the ```setExpression!``` motion, our trigger fired, told the Runner to re-run itself, which caused it to set the result value on the Runner. And now we've got our own, custom-built "live coding" environment!

-----

So now we've added our lovely function with its one statement, and we've successfully run it and seen the output. Now, let's investigate how debugging happens in Mesh.

First, we're going to fast-forward a bit and make our project slightly more complex:

```
Fn addTwo(a:Int, b:Int):Int:
	return (+ a b)

Fn main():None:
	log (print "We should see 7:")
	log (print (addTwo 4 4))
		
```

If we enter ```(runner.run)```, we'll see that the incorrect thing is shown. While this is a contrived example, we're going to use this as an opportunity to build ourselves a custom debugger. As usual, we'll be taking the very long way around.

To begin with, we're going to explore another new idea: **VMs**. We were using them in previous examples, but now we'll explore what they are and how they work. Just as the Mesh *program* is represented as a Mesh object, and the Mesh *editor* is a Mesh object, so too is the VM that the program runs on represented as a Mesh object. VMs keep track of the following pieces of information:

* The next expression to be evaluated (you can think of this as the instruction pointer)
* The symbols that are in scope at a given point in time
* The input and output channels that the VM can take advantage of to interact with the outside world

Let's say that we want to see all of the symbols that are in scope at that ```return``` statement in ```addTwo``` - this should help us track down the bug. To do this, we're going to change our runner in the following ways:

1. We want to create a new extension to display the contents of the execution context's scope.
2. We want to display the symbols in scope at that point
3. 


