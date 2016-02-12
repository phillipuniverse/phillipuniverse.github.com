---
layout: post
title: When in Doubt, Write it Out
tags: programming, writing 
---

Over the course of my career I have run into *countless* (and I mean countless) situations where I have no idea what's going on with a particular piece of functionality. I will stare and stare and stare, go on countless debugging sessions over and over and frequently ask myself "how did this *ever* work?". Then I'll throw up a `git blame` to see who it was that committed the mortal sin of writing this abhorrent pile of excrement sitting in my editor (of course, sometimes it's me). If I'm still stumped and the path to correctness (or is the definition of what exactly "correctness" is) is unclear, I start writing. I write about utility classes, I write about code paths, I write about parameters, I write about assumptions I make about the code and assumptions that are baked into the code itself; I write about *everything*. There are some very clear advantages to writing out problems in this way.

### It forces you to structure your thoughts
Writing forces me to flex a different muscle. When I think about a problem and just mull it around in my own head, my brain tends to go off in a thousand different directions. Same thing when I have a conversation about the problem to someone else; I have a tendency to go off on rabbit trails and it's difficult to stay on track. Writing it out makes me get it all out there in excrutiating detail in a way that someone else can understand and serves as a form of [rubber duck debugging](https://en.wikipedia.org/wiki/Rubber_duck_debugging).

### It's asynchronous
Developers are all familiar with the concept of *The Zone™*. You know, that magical place you go where the world disappears around you and code flows from your fingertips like water. Usually when you're stumped, the tendency is to go ask someone else that knows more for help. That runs the risk of throwing that person out of the zone which can take an eternity to get back into. Writing the problem out and throwing a link (or creating/commenting on an issue in your tracking system) to someone allows that person to circle back to it when they have time.

### You're documenting as you go
Frequently, codebases are poorly documented (especially non-public ones). Often there are hidden business logic assumptions baked deep into utility classes or other gotchas that you find along the way that somebody that hasn't worked here for the past 3 years wrote with a big `//TODO: FIXME LATER` comment. This is the chance to root all of those out and put the entire code path or feature in question. Then if somebody else runs into a similar problem, you have a good reference guide for why something is the way it is.

### It's a bookmark for your work
Most of the time, you are the best person to solve the problem you are looking at. But things come up, priorities shift and you won't always have a block of 4-5 hours to devote to finishing a problem. If you've written it all out, when priorities shift back you can pick up right where you left off without trying to keep it all in your head and remember where you were. A lot of the times I'll come up with the solution by the time I'm done writing!

So next time you're stuck with a seemingly unsolvable problem, start writing. You'll be surprised with what you come up with.
