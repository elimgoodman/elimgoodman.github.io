---
layout: page
title: Idea Inspirations
---

##Some Ideas About the Filesystem

- Basically, there's an abstraction 
between files on disk what applications read and write to
- Apps like iTunes and iPhoto already do this - create a metadata DB that adds all kinds of stuff on top of files and speeds up access
- This is largely focussed on "regular" computer users, not programmers
- Largely geared towards situations where one piece of "content" corresponds to one file
- All files on disk represented by a document DB
- When an application wants to persist some data, it writes it to the DB
- There are some canonical types (document, picture, video, contact, etc) - each piece of content must have a type associated with it
- Each piece of content can also have arbitrary metadata associated with it (who's the picture of, this picture is from Instagram, etc)
- No binary/"actual" content data stored in the DB - DB has references to another binary data store.
	- At save time, metadata written to DB, binary data written to data queue
- User can write "persistence strategies" that take data out of the queue and persist it for reals
	- ie "write all files to disk in this way", which is what we currently have
	- "write the 1000 newest of my photos to disk using their date_taken to make the directory structure, then sync the remaining photos to S3"
- Metadata DB maintains a pointer to wherever the data ends up, so you know what's local and what isn't

- How does this impact sharing? "I want to give you access to this slice of my data/I want to group this slice of my data together and give it a URL"
- Is it interesting if (some subset of )this is mirrored to the cloud. FB is this, centralized. I would want to own my content DB.
- It might be interesting to think about how something like this could replicate something like a blog
	- If sharing is a thing, how do I get control over the presentation layer?
- What does it look like if it replaces email?