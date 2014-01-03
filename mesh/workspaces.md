Thus far, we've covered the basics of the ideas that underpin Mesh, and how those basic ideas can be extended to match your program's domain (and your own personal preferences). In this video, we're going to discuss how to discuss how Mesh allows you to organize and traverse the actual workspace of your program. 

To help keep things organized, Mesh has an object called a **perspectives**. A perspective is simply a grouping of some subset of the regions in your program. One region can appear in any number of perspectives. You can think of perspectives like browser tabs - you'll probably have a  bunch of them open at any given time.

Each perspective takes a set of regions, a filtering function, and an optional name. A filtering function is just any function that returns a set of regions. A general description for filtering functions is:

```
	#($regions : (Set Region)):(Set Region) ^{#performsIO => false}
```

The perspective will display a union of all of the regions that were explicitly passed to it and all of the regions returned by the filtering function.

So, let's say we wanted to show the contents of a particular package. Its filtering function would be:

```
	def isPackage #([r : Region] (= (:type r) Region)))
	def package ($regions.filter #(and (isPackage %) (= (:name %) "myPackage"))
	package.regions
```

Alternatively, if we wanted to show all of the functions in our program that returned an Int:

```
	def isFn #([r : Region] (= (:type r) Fn)))
	($regions.find #(and (isFn %) (= (:returns %) Int))
```

Because we can have many different perspectives open at any given time, Mesh also gives us the ability to zoom out and organize both our perspectives and regions spatially.

Additionally, because perspectives are just instances of Mesh structs, we can create and manipulate them programmatically using motions. For example, we can write a motion that, if the cursor is focussed on a function call, will open the definition of that function in a new perspective:

```
	def focussedEntity ($cursor.getFocussedEntity)
	match focussedEntity.type:
		case FnCall: 
			cast focussedEntity FnCall
			def fn focussedEntity.fn
			def perspective {{Perspective :regions => [fn] }}
			($perspectives.append! perspective)
```

When combined with the ideas presented in the previous video about [concepts](http://), perspectives offer a good way of hiding the complex bits of a program while exposing the declarative bits. When combined with the ideas around motions, they offer a powerful way to traverse and organize the contents of a program, irrespective of how the program is represented in the filesystem.
 