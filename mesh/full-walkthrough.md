---
layout: page
title: Mesh - Crash Course
---
{% include JB/setup %}

#Startin' Real Basic: Hello World

As our very first task, let's start with the tried and true: printing "Hello world" to the screen. Note that we're gonna be doing things the hard way at first, to give you a better idea of how Mesh works.

###Let's Make, like, just one function

When we open Mesh, we're shown a new, blank project:

![image](http://elimgoodman.com/assets/mocks/output/blank_project.gif)

The first thing we're gonna want to do is create a function to house our instruction to print "Hello World." To do that, we're going to want to bring up the **command bar** by hitting ":". You can think of the command bar as a really powerful REPL:

![image](http://elimgoodman.com/assets/mocks/output/command_bar_open.gif)

Next, we're going to enter two commands into the command bar, hitting enter after each one (don't worry about the syntax for now):

![image](http://elimgoodman.com/assets/mocks/output/new_fn_command.gif)

And voila! A blank function appears!

What just happened back there? Well, let's think of how we would describe what we just did. In a traditional programming language, the act of "creating a function" starts when the first letter of the function definition block is typed, and ends when the last letter is typed. In Mesh, the function is created in one fell swoop, all at once (that's what the first command is). Then, it's attached to our program (the second command). We call changes to the entities in our program **motions**.

Motions are extremely powerful. While the actual syntax of the commands isn't important right now, what you should note about them is that they are just regular Mesh statements, manipulating regular Mesh objects. In other words, the *program we're building* is represented as an object in our system, and we can manipulate it using the full power of the Mesh language.

###Changing Our Function's Name

Let's now make a change to our function. Remember, we're doing this the hard way first. Let's bring back up the command bar and enter this command:

![image](http://elimgoodman.com/assets/mocks/output/name_changed_fn.gif)

Note a few things here. First, we didn't have to create the symbol "fn" again - the command bar persists objects for you, just like a REPL would. Second, you can see that we're operating on our function region just like we would operate on an object with a traditional OO language - we're using predefined methods to alter its internal state. This is how *all* manipulations to your program take place in Mesh. It might seem cumbersome at first, but with a combination of hotkeys and practice, we've found that one actually becomes faster by using motions instead of text.

Additionally, manipulating the program this way allows us to leverage the full power and expressivity of the Mesh language, while also maintaining the type and state safety guarantees that the languages provides us. What this adds up to is the ability to make changes to your program quickly, safely, and ambitiously. Mesh allows you to boldly refactor where you may have not dared before.

###Adding a Statement

We're now ready to add our first statement to our program. The statements in our program are just regular Mesh objects. To add one, we'll bring up the command bar again and type this in:

{% highlight clojure %}
{% raw %}
def stmt (Statement.fromString "log (print \"hello world\")")
(fn.statements.append! stmt)
{% endraw %}
{% endhighlight %}

After we hit enter, we see that our function now contains a new statement:

![image](http://elimgoodman.com/assets/mocks/output/fn_with_statement.gif)

Ah, that "fromString" method was convenient; using it meant that we didn't have to create all of the underlying objects that that statement requires (such as expressions and literals). All Mesh objects serialize and deserialize to text; Mesh programs are stored as text on disk, so you can still used familiar tools such as git and grep with them.

###Running Our Program

Let's start out by just running our program via the command bar:

![image](http://elimgoodman.com/assets/mocks/output/running_program.gif)

We can see there that ```vm.stdout``` is equal to "hello world". We did it! But what exactly did we do?

In short, what we did above was create an execution environment (that's what {% raw %}```def vm {{VM :stdout}}```{% endraw %} is doing). We then manually executed each statement of our function in that environment.

While we don't have to get too deep into the exact mechanics of how Mesh code is run, it's important to note from the above example that we created a regular old Mesh object to help us run our function . So - not only is our program comprised of Mesh objects, but the actual execution environment is also represented as a Mesh object. This is very powerful, as it allows us to have a lot of control over and insight into how our program is run.


#Let's Build Ourselves a Live Coding Environment

So, we got our program to run, but it was pretty involved. It would be super annoying if we had to type all of that code into the command bar every time we wanted to run our program. Instead, let's create an object to help us run our program easily.

One thing that we haven't talked about is that not only are the *program* and *execution environment* just regular Mesh objects, but the *editing environment* is also just comprised of instances of Mesh objects. We're going to take advantage of this fact to extend the editor with a new kind of interface element. Objects that only exist in the editing environment and have no bearing on the execution of the program are called **extensions**. We'll be creating an extension to run our program for us and display the output. We'll call it, creatively, Runner. Let's type this in the command bar:

{% highlight clojure %}
{% raw %}
def extension_string """
	Extension Runner:
		fn : Fn
		run():String ->
				ref stdout ""
				def vm {{VM :stdout stdout}}
				loop this.fn.statements statement
					(vm.executeStatement! statement)
				return context.stdout

"""

def extension (Extension.fromString extension_string)
(editor.extensions.add! extension)
{% endraw %}
{% endhighlight %}

We're creating an extension from a string, just like we did above with the statement. What that syntax specifies is that we're making a new extension (named Runner), and instances of the Runner extension will need to be provided with a function as a parameter. Runner extension instances will also have a method called "run" which takes no parameters, and returns a String.

Now that the editor knows how to make Runner objects, so let's actually make one. We'll type this into the command bar:

{% highlight clojure %}
{% raw %}
def runner {{Runner :fn fn}}
(cursor.getCurrentPerspective).(getCurrentPackage).regions.(append! runner)
{% endraw %}
{% endhighlight %}

And we hit enter:

![image](http://elimgoodman.com/assets/mocks/output/runner_created.gif)

And look at that! We see a Runner in our project, with its function set to our "main" function. Now we can use it to actually run our code:

![image](http://elimgoodman.com/assets/mocks/output/runner_from_command_bar.gif)

And the right thing came out! Yay!

However, this isn't too much of an improvement. After all, why have that Runner region sitting there on our screen? We can't really interact with it meaningfully, except to change the function that it runs. Let's make the UI of that element a little more helpful.

All extensions have a user-editable UI that they present in the project view. Let's make our Runner display the output of its last run right in the region. To do that, we need to do three things:

1. Alter the Runner extension to store the state of stdout after it's been run.
2. Update the "run" method to update the result after running
3. Update the UI of the extension to display that information.

Let's add a field to the Runner extension to store the state of stdout:

{% highlight clojure %}
{% raw %}
def field_type (Type.fromString "&String")
(extension.addField! :name "result" :type type)
{% endraw %}
{% endhighlight %}

Now our existing Runner instance now has a "result" field. We can see that it's currently set to the empty string:

![image](http://elimgoodman.com/assets/mocks/output/runner_with_result_command.gif)

Next, let's update the method:

{% highlight clojure %}
{% raw %}
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
{% endraw %}
{% endhighlight %}

Then let's update the UI:

{% highlight clojure %}
{% raw %}
def panel_string """
	{{Row
		{{StaticLabel "Result:"}}
		{{DynamicLabel this.result}}
	}}
"""

def panel (Panel.fromString panel_string)
(extension.ui.setPanel! panel)
{% endraw %}
{% endhighlight %}

Tada! We now see that our Runner region is looking much prettier:

![image](http://elimgoodman.com/assets/mocks/output/runner_new_ui.gif)

And now, running our Runner again should make the result appear:

![image](http://elimgoodman.com/assets/mocks/output/runner_new_ui_result.gif)


Nice! But let's say we want to take this one step further - let's have Mesh re-run our Runner every time we alter that function. To do that, we're going to introduce another new idea: **triggers**. Triggers are just snippets of Mesh code that get run whenever certain motions occur. Let's make a new trigger to kick off our Runner when our function is changed:

{% highlight clojure %}
{% raw %}
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
{% endraw %}
{% endhighlight %}

Now, let's alter our function:

![image](http://elimgoodman.com/assets/mocks/output/trigger_run.gif)

And there it is! As soon as we performed the ```setExpression!``` motion, our trigger fired, told the Runner to re-run itself, which caused it to set the result value on the Runner. And now we've got our own, custom-built "live coding" environment!