---
layout: post
title: Sproutcore training with Erich Ocean- A Review
---
#Sproutcore training with Erich Ocean: A Review
So today I took took the day off work, and attended Erich Ocean's Sproutcore training session that he's been posting on the Sproutcore mailing [list](http://groups.google.com/group/sproutcore/browse_thread/thread/18fb7b643a745552#). I figured that since Erich is the only source that I have seen for training, and since he also is the one who has put forward the [book proposal](http://www.google.com/url?sa=D&q=http://erichocean.com/book/index.html) that it would be a good oportunity to improve my skill in the Sproutcore world. This article is an account of my experience. Hopefully it will help you decide whether to attend his next class.

##Format
The class mainly took the form of Erich talking though diagrams or code via screen-sharing, and answering questions inputted into a chat window as we were going along. This was accomplished using GoToMeeting's GoToTraining [software](http://www.gotomeeting.com/fec/training/online_training).

This format generally worked quite well. There were 23 people who showed up for the class, so initially I was a little worried that the chat would be too busy. However only 6 or 7 people really actively participated in the chat, so it was easy to get my questions answered. The only downside to the setup was that the audio (and I think the video) kept going out on Erich's side. This happened about ~10 times over the 4 hour period, where it seemed like his network connection just died. Overall though I didn't find it to be terribly painful, and it was a small price to pay in order to be able to get training in my pajamas, and from my own home.

In our case, because Erich was on the road he wasn't recording our particular session, but he did say that he was going to give us access to a previously recorded session, which had the same content.

##Content
The content was broken down into three sections, which covered the follwing material:

###Sproutcore MVC
* Sproutcore runloop
* How bindings fit into the MVC
* How the MVC is different from something like Rails
* The "V property", and the order things must process through the stack
* Using pre-built controllers
* "event-check state-action" paradime
* The two fundamental tasks to correctly code an app
* How the cloud slows everything down


###Statecharts
* The statecharts notation
* How to desigin statecharts
* How statecharts solve "the two fundamental tasks to correctly coding an app"
* Practicing building a statechart
* How to "unit" test a statechart


###Miscelanious Topics
* How to divide up the work with statecharts and the mvc
* How to access subviews (SC.Outlet)
* When to use custom views (always)
* How to connect with your backend (by avoiding datasources)
* Q/A

##Conclusions
The class was billed as a course that would "permanently change how you think about application construction and coding with a unique, visual explanation of the MVC architecture in SproutCore", and I think for the most part it accomplished it's goals in that regard. Erich clearly was both very knowledgeable about Sproutcore, and very opinionated. He was not merely content to layout the way things are in Sproutcore, but desired to impart to his students how he thought they could improve the structure of their applications. It was this opionated approach that I think was the class's greatest benefit, although it left me with several questions. 

For instance, Erich said that any view that is going to be drawing content should be a custom view, and that one should never extend those views from anything but SC.View. This viewpoint astounded a large number of people in the class, and several people expressed that they felt that their entire application had been built on faulty ideas. When some said that they liked the idea of moving away from html, Erich suggested that they might find Cappuccino to be a better fit than Sproutcore. This was all the more shocking although it certainly necessitated transforming about how one should think about building their view layer.

This was not the only place that what Erich said seemed to stray from the "suggested way" that I have seen in working through the wiki tutorial. The following are a couple more examples:

* You want to move all of your domain logic onto the server if you can, and only store attributes on the client.
* Put no custom code in controllers.
* Have a singleton application controller which controls all the actions of the application.
* .observes is a code smell

I point these things out not because I disagree with them, but because I'm not sure if I'm qualifed at this point to even have an opinion on these issues. The class certainly challeged me in that regard, but it also left me with a distict feeling that I wasn't getting the "Sproutcore way", but the Erich Ocean's way. Perhaps this is exactly what I need - only time will tell- but for the time being, it left me with more questions than answers - particularly about how people on the core team view Erich, and his approach. 

Having said that, I certainly felt that I got something from the course. The content was engaging and well prepared, and it was fun to work through the material. The training materials on Sproutcore are definatly lacking, and so to have the oportunity to watch, and talk with Erich for 4 hours was a steal of a deal at double what I was charged. I would certainly recomend the course to others, just go into the class knowing your getting an opinionated lesson.

##EDIT - 2011-01-29
Couple points of clarification:

* There doesn't seem to be as much tension between what Erich was saying in the class and the canonical way of building views. Yehuda clarified this point in a thread on the developers list, which you can read [here](http://groups.google.com/group/sproutcore/browse_thread/thread/eb75b3fbc0c69da3). 

* Erich further clarified to me that he would not always move all of his domain logic onto the server, but only in a case of working on a LAN, when the price of the network is basically zero. 

* Also, when I said "singleton application controller", you can see my naivete in relation to Sproutcore. Erich actually meant a single responder, usually putting the logic in SC.Application for small to medium size apps. Although everything seems to be moving to statecharts now.

* Most of the things that shocked me was likely relating to the wiki being so outdated (particularly the todos tutorial). The guides seem to fit into Erich's approach without as much of a problem. Erich's opinionated tone made it seem to me like there was more of a difference than there really was.
