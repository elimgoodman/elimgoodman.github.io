So now we've added our lovely function with its one statement, and we've successfully run it and seen the output. Now, let's investigate how debugging happens in Mesh.

First, we're going to fast-forward a bit and make our project slightly more complex:

```
Fn myPrint(s:Serializable):None
	tell StdOut write (s.serialize)
	
Fn addTwo(a:Int, b:Int):Int:
	def result (+ a b)
	return result

Fn main():None:
	log (myPrint "We should see 7:")
	log (myPrint (addTwo 4 4))
		
```

If we enter ```(runner.run)```, we'll see that the incorrect thing is shown. While this is a contrived example, we're going to use this as an opportunity to build ourselves a custom debugger. As usual, we'll be taking the very long way around.

To do this, we need to explore what's happening behind the scenes when Mesh code is executed. As you may have seen above, we rely on a special kind of Mesh object to execute code: a **VM**. Just as the Mesh *program* is represented as a Mesh object, and the Mesh *editor* is a Mesh object, so too is the VM that the program runs on represented as a Mesh object. 

You can think of a VM as a little abstract computer - it has some internal state and some connections to the outside world. Regular computers are connected to all sorts of IO devices - screens, keyboards, network connections, and so on. In Mesh, we represent these though another idea, called **channels**.  

In Mesh, channels are how VMs affect change in or get information about the outside world. Fetching records from a database, getting input from the keyboard, or writing a file to disk all involve channels. 

//Why is this super relevant right now? How deep do we have to go on this? Don't want to get too deep on syntax...

As mentioned, Mesh VMs record some internal state, just as a normal computer might have RAM and hard drive space. Mesh VM objects keep track of the following pieces of information as internal state:

* The next instruction to be evaluated
* The symbols that are in scope at a given point in time
* A stack of locations that represent the function calls that got us to the current point of execution
* Some space that's used to pass parameters to functions when the instruction pointer jumps
* Some space that's used to pass back return values from functions
//* The input and output channels that the VM can take advantage of to interact with the outside world

OK, So we're keeping track of the above state, and we have a set of input and output channels we can take advantage of to perform IO. The Mesh VM needs the program to be described as a set of discrete operations that take place over the above elements - either on one of the pieces of internal state, or on a channel. Many valid Mesh statements are actually a composite of a few of these discrete operations. For example, the "return" statement both sets the piece of memory that's used to ferry return values around, and also jumps the instruction pointer back to the call site. So, we need a more basic, VM-friendly way of representing our program. 

Luckily, Mesh makes transforming a program into a VM-friendly format super easy. As an example, let's click on our return statement in ```addTwo```. Let's bring that statement into scope:

```
	def stmt (cursor.getSelectedStatement)
``` 

Now watch:

```
	(stmt.toInstructions) -> [Instructions.SetReturnValue, Instructions.PopLocationAndJump]
```

Lookit that! It turns out that every statement has a method called ```toInstructions```, which returns a list of **instructions**. Instructions are commands that can be run on a VM to affect some change to its internal state or IO channels. The Mesh VM needs this idea of instructions to more accurately represent where the program is "at" at any particular moment in time. For example, consider this Mesh statement:

```
	match (someFn a b)
		case 1 (print "it was one")
		case 2 (print "it was two")
```

When we execute the whole statement, a number of things occur - we evaluate the call to ```someFn```, we pass the result into the cases, then the body of the correct case executes. To say that we're "on" that particular statement isn't accurate enough. Luckily, instructions are accurate enough for us - and they're built right in.

As you might expect, instructions themselves are *also* represented as regular Mesh objects. We can create them independently, manipulate them programmatically, and so on. If we wanted to, we could create arbitrary instructions in our REPL, and then run those on arbitrary VMs in order to get them into some desired state. We could also just alter the internal state of any VM to be what we needed it to be. Even better, if we get a VM into a state that's useful, we can spawn copies from it to use during future runs of our program. Having VMs and instructions represented as regular Mesh objects is powerful and useful (and again, we get that delicious, delicious type and state safety for free).

OK, so after that epic detour, let's come back to our task: to build some kind of debugging mechanism for our program. Let's say that we want to see all of the symbols that are in scope at that ```return``` statement in ```addTwo``` - this should help us track down the bug. To do this, we're going to change our runner in the following ways:

1. We want to create a way to display the contents of a VM's scope.
2. We want to have an instance of that extension display the VM's scope at the correct point in our program

Instead of rolling our own mechanism to view the scope object, we're going to use Mesh's built-in ScopeViewer extension. Let's create one:

```
	def viewer \{\{ScopeViewer}}
	(cursor.getCurrentPerspective).(getCurrentPackage).regions.(append! viewer)
```

We should see an empty scope viewer appear on the screen:

[picture:empty_scope_viewer]

Next, we need to modify our runner to set that scope viewer's content when it encounters a specific instruction. 
]