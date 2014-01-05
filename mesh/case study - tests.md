As an example of some cool stuff that you can do with Mesh, let's talk through a case study. Let's say that we want to add the ability to add and run tests to our project. Let's start off by creating a concept that represents a test:

```
	concept Test:
		being_tested : String
		fn : Fn
		body : #():Boolean
```

Now we can make a new Test like so:

```
	(cursor.currentPackage).(addRegion! {{Test}})
```

See that the "fn" field has a select element, while the "body" field has a statement list. Let's add a function, and write some tests for it:

```
	fn add(a : Int, b : Int):Int:
		return (+ a b)
		
	Test
		being_tested: "Adding zero to a number"
		fn: add
		body: #(||
			return (add 0 1).(equals 1)
		)
	
	Test
		being_tested: "Adding two non-zero numbers"
		fn: add
		body: #(||
			return (add 1 2).(equals 3)
		)
```

Now, let's add a trigger to run all of the tests associated with a given method whenever a method gets edited - if a test fails, we'll tag the method:

NOTES: 
* this is not the right syntax for motions but whatever
* Make a point to say that all concept instance collections are read-only global singletons


```
	trigger
		on: Events.RegionEdited
		action: #(
			//ugh some stuff around getting the actual Fn that was edited
			def tests (filter Test #(eq %.fn motion.affected))
			loop tests test
				def result (eval test.body)
				branch result:
					case False: ^{:failedTest true :failure test} fn
		)
	
```

So now, when we edit the function, the tests automatically run. However, we can't see their output. We have to make that new tag visible in the editor:

```
	(editor.setTagVisibility :failedTest true)
```

Now we can see the failing tests!

It would probably be useful to be able to open all of the tests that invoke the focussed method, so let's create a new command to do that:

```
	command showTestsForFn:
		current_fn = getCurrentFn()
		tests = getAllTestsForFn(current_fn)
		perspective = newPerspective()
		perspective.setRegions(tests)
```

So now, we can focus on a function, then run the given command. Voila! The tests appear to the right of the function. Very handy.

Now let's say we want to make it so that we can specify any number of functions for a given test to be associated with. We're going to want to perform the motion of changing that field's type. When we do that, we need to provide a function that specifies how existing instances' "fn" fields should be updated:

```
	(Test.changeFieldType! )
		
```

TODO:
* Later, let's make it so that we can associate a test with multiple methods. We'll have to do a type-change migration, so we'll need to pass a "how to migrate" function
* And suites of tests as well
* Make it infer all of the call fns from the body of a fn, auto-bind to all of those