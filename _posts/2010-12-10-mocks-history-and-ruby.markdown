---
layout: post
title: Mocking, History & Ruby Community
---
#Mocking, History & the Ruby Community
Recently Nick Gauthier wrote a [post](http://www.ngauthier.com/2010/12/everything-that-is-wrong-with-mocking.html) in which he critiqued how a lot of people use mocks in RSpec. Then in the comments on the post there was a good discussion with Nick, David Chelimsky and others discussing the topic. But in the course of reading through, a bigger problem jumped out at me. As a whole, we in the Ruby community are too ignorant of history and theory. So this post is not a critique per se on the mocking discussion (I myself tend to be more of a classicist TDDer rather than a mockist TDDer) but rather an attempt to point out a larger problem.

##Why does mocking exist?
Mocking was first presented (publicly) at the XP 2000 conference by Tim Mackinnon, Steve Freeman and Philip Craig. You can read the original paper [here](http://connextra.com/aboutUs/mockobjects.pdf). It was developed initially as a way to do two things: (1) Test an object in isolation from it's dependencies. (2) Avoid polluting domain objects with getters so that the testing framework could check their internals. 

However, as the description of the concept evolved, the idea of mocking objects became much more about OO-design than it was about dependencies. The "founders" clarified their usage in 2004 at OOPSLA in [this](http://www.jmock.org/oopsla2004.pdf) paper. They desired to design their systems "as a web of collaborating objects" each of which followed the [SRP](http://www.objectmentor.com/resources/articles/srp.pdf), and [the Law of Demeter](http://en.wikipedia.org/wiki/Law_of_Demeter). As such, they desired to change the behavior of the system by changing the composition of its objects - that is adding and removing instances or pluggin different combinations together - rather than writing procedural code. Therefore, they insisted on "focused objects that send commands to each other and don’t expose any way to query their state". 

This of course then seemed to back them into a corner, since if their objects did not expose their state, how was one to write unit tests for said objects? They proposed to "change the way we approach TDD" by focusing more on the interaction (or ["messages"](http://lists.squeakfoundation.org/pipermail/squeak-dev/1998-October/017019.html) as Alan Kay would call them) between the objects. Instead of testing the state of an object after a given operation, they proposed checking how the object communicates with it's neighbors. The unit test's goal then is to make explicit the relationship between the target object and it's environment. 

Perhaps the worst decision that they made in the description of their procedure is that they originally decided to call the testing structures mock "objects". Because of the name of their proposed test structure, people thought that they were advocating designing the **objects** that the mock is representing. In fact however they were advocating that one ask what *role* that object is going to have to play, *without* specifying how one is going to implement that role. They were attempting to advocate that one mock an **idea** or a **role**, not an implementation, so that the object under testing would be forced to delegate any responsibility that was not their's.

##So then how should mocks be used?
In the section of the RSpec book which Nick quoted, David created a mock object representing an output object which is a dependency for the codebreaker game that they were writing. The ensuing discussion then focused upon the need to specify the implementation of the standard output object, and as such making the code brittle. Nick's objection (and quite rightly) was that David was seeming to specify the implementation of the standard out object in his test. 

In fact though, the real problem is that David has broken the rule of "only mock types you own". What he should have done (and could easily refactor it to) is created a thin wrapper around the standard output object that speaks in the domain of the system in which he was working (notice by the way that I'm speaking in David's language from his comments, he knows this). The mock output object then would have represented *the idea* of an output object within the codebreaker domain, and it would have been acknowledging the fact that the game object should **not** be responsible for it's own output (SRP). The mock object then would be specifying not **how** the other object functioned, but **what** it must do. He then would be able to temporarily mock the output object, and once the object specifying the state of the game was green, he could write tests to specify *how* that object functions. Also, then the implementation of *how* that output object works could change without breaking the game object, and vice versa.

##So then what can we learn?
I think the real thing to take away from this discussion is that we as a community need to do a better job of educating ourselves about why the tools that we use exist, and what they were intended to accomplish. The Ruby community is a great place to work in, and we really are pushing the boundaries on many fronts. However we tend to not care as much about ideas that were thought of and developed in other languages like Java. We tend to only focus on their ruby implementation, instead of understanding the ideas behind the technology. If we are going to be true craftsman, I think we also need to learn our history and theory. 

Also, I think that the true solution to when we see newbies making the sort of errors that newbies make is to point them to the right sources. They need to know the theory behind the tool that they're using. They don't only need to learn how to do something, but why. And history often is an engaging source to get them interested.

##Further Mocking References
Martin Fowler, ["Mocks Aren't Stubs"](http://martinfowler.com/articles/mocksArentStubs.html)

Dan North (Founder of BDD), ["The End of Endotesting?"](http://blog.dannorth.net/2008/09/14/the-end-of-endotesting/)

XUnit Test Patterns by Gerard Meszaros [Amazon Link](http://www.amazon.com/dp/0131495054?tag=xuntespat0f-2&camp=14573&creative=327641&linkCode=as1&creativeASIN=0131495054&adid=1WNXJS2GC92A3C5RKSY4&)

Growing Object-Oriented Software, Guided by Tests by Steve Freeman (One of the founders of mocking) [Amazon Link](Growing Object-Oriented Software, Guided by Tests)
