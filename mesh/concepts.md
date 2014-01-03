In the previous video, we talked about the basic ideas that Mesh builds upon - how it represents the program that you're working on, and how we can make changes to that program. In this video, we're going to talk about how we can extend Mesh to make it "fit our hands" better, to better represent the domain we're working in. 

As we talked about in the previous video, Mesh provides a vocabulary of nouns and verbs that facilitate making changes to your program. Mesh allows you to add both nouns and verbs to the system. Adding nouns is how we can add new kinds of declarative data to our system, while adding verbs will allow us to make changes to our program safely and efficiently.

Adding nouns to a Mesh program is done by creating a new **concept**, the object that we avoided talking about in the last video. A concept, at first glance, should look very similar to a struct - it has a series of fields with types, and methods. So why do they exist? Well, when we create a new concept, that concept can now be treated a noun in our vocabulary. We can create motions that operate over instances of that concept, just as we can make motions that operate over all of the functions or structs in our program.

As a rule of thumb, if something is your program can be influenced by user input, it should be a struct. If it would require a code change to update, it should be a concept. For example, let's pretend we're building a chess program. We would probably want to represent the board as a struct, because the data contained within it is influenced by user input. However, we would probably want to represent the types of pieces and rules of the game as concepts. The distinction is subtle, but once you play around with it for awhile, you get used to it.

There are a couple of key benefits to representing parts of your program in concepts. First, instances of concepts appear as regions in the editor, just like functions and structs. What's cool is that we can change how each concept's region looks and behaves - we're able to bring in arbitrary HTML, CSS, and Javascript and use those to represent instances of that concept. For example, let's say we have a concept to represent a color:

```
	concept Color:
		r : Int
		g : Int
		b : Int
```

Here's what it looks like before we style how Color instances look:

And here's what it looks like after:

Much better!

Using concepts also encourages us to make as much of our programs declarative as possible. Imperative code tends to be where complexity lives. By making as much of our program declarative as possible, we can cut down on unnecessary complexity. 

Also, it's important to note that concepts can include imperative code. For example, here's how we might represent routes in a website:

```
	concept Route:
		url : String
		method : HTTP.Method
		action : #(req: HTTP.Request):String
```

One cool thing to note here is that Mesh will try to be smart about how it represents the concept's fields. Because the "method" field in the above example has a value set type, instances will by default have that field represented by a select box. Additionally, because the "action" field has a function type, it will be represented as a list of statements by default.

Adding verbs to our vocabulary is a little more straightforward. Just as we can create new concepts that look like structs, we can create new motions that look like functions. You can actually think of a motion as a function with a very specific signature:

```
	myMotion(&$regions : (List Region), &$cursor : Cursor):None ^{#altersReference => true, #performsIO => false} ...
```

There's some unfamiliar syntax in there, but what's being described is a function that takes references to all of the regions in the program (which are the functions, structs, interfaces, and so on), the cursor (which is an object that knows where the "focus" is), returns nothing, can modify either of the objects that are passed to it by reference, and cannot perform IO. 

For example, let's make a motion that makes 5 new functions in the current package:

```
	motion fiveNew():
		def current_package ($cursor.currentPackage)
		(repeat 5 #(current_package.addRegion! {{Fn}}))
```

Then we can run it in the command bar:

```
	(fiveNew)
```

And we have 5 new functions! We can also make motions that take parameters:

```
	motion makeFunction(name : String):
		def current_package ($cursor.currentPackage)
		(current_package.addRegion! {{Fn :name => name}})
```

And we can run it:

```
	(makeFunction "foo")
```

We can also run it without the argument, and Mesh will ask for it:

```
	(makeFunction)
```

You might be thinking about how weird it is to have concepts, concept instances, and motions live in the same package as functions, structs, interfaces, and value sets - the former are very "meta", being related to the functionality of the editor itself, while the latter are all about functionality that users actually see. However, having the two live together actually have some very cool ramifications which will be addressed in a subsequent video. Additionally, Mesh's type system prevents the two "kinds" of objects from colliding in  destructive ways. For example, what if you could make a motion that would drop your database? That'd be crazy! Mesh tries its best to stop that from happening.