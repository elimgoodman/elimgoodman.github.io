In this video, we'll explore two ideas that Mesh employs to make the development process faster and safer: **triggers** and **overlays**. 

Overlays represent structured metadata about the entities in your program. Any entity in the program can have metadata assigned to it. All overlays share the same signature:

```
	(Map Entity (Set Metadata))
```

You can make your own metadata structs - they just have to implement the Metadata interface. For example, all projects have a built-in overlay to store and display errors. Overlays are also a good way to store comments, TODOs, and human-readable annotations. Additionally, because overlays are just Mesh objects, we can create motions to create or modify them. For example, here's a motion to add a TODO to the current region (note: we should show the creation of the overlay itself, along with the Todo metadata object): 

```
	struct Todo : Metadata
		content : String
		getLabel : #(this.content)
	
	overlay Todos(Entity Todo)

	motion addTodo(content : String)
		def region ($cursor.currentRegion)
		(Todos.addForEntity! region {{Todo :content => content}})
```
Now, we see that annotation by showing the Todo overlay.

The Mesh editor allows you to show or hide overlays as desired. It also allows us to navigate to the various entities that have been annotated in a given overlay, or view all of annotations in a list.

Triggers are a compliment to overlays - they're just some arbitrary code that runs after a motion is performed. Each trigger can specify the motions that it wants to be triggered by, or it can be triggered by all of them. Their general signature is:

```
	#($motion : Motion, $affected : (Set Entity), $regions : (List Region), &$overlays : (Set Overlay)):None ^{performsIO => false, #altersReferences => true}
```

Triggers can inspect the current state of the program and add or remove overlay entries as needed. For example, we could make a trigger that would check to make sure that we don't add two functions with the same name:

(Note: the below would only be bound to the NewRegion motion, and this wouldn't have to do casts..?)

```
	trigger checkDuplicateFnName (:motions):
		match $motion.region_type:
			case Fn:
				def new_fn ($affected.pop):(Maybe Fn)
				def other_fn ($regions.find (= (:name %) new_fn.name))
				match other_fn:
					Just _: 
						def errors ($overlays.findByName "Errors")
						def error {{Error :content => "Duplicate name", :data => {:other => other_fn} }}
						(errors.addForEntity! new_fn error)
						
```