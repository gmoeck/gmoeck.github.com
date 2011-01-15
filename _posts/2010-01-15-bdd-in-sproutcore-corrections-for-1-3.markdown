---
layout: post
title: BDD in Sproutcore - Corrections for parts 1-3
---
#Corrections for parts 1-3
I haven't gotten any feedback from people from my posts thus far, but I've noticed several places where my code should change, and get cleaned up a bit. This is what this post will be about.

##Using outlets
The first really brittle piece of the code that I noticed is where I was using getPath to reach into the view and find elements. By using a long string like 'mainPage.mainPane.middleView.contentView' I'm coupling a good number of my tests to the current structure of my views. In order to fix this, and to give the element a description that is closer to our domain, we can refactor this dependency to use SC.outlet:

<script src="https://gist.github.com/781228.js"> </script>

What SC.outlet does is generate a computed property that will look up the passed property path the first time you try to get the value. I've done this for the add task button, the count of task, and the list of tasks. 

[changes](https://github.com/gmoeck/sc-bdd-todos/commit/8ece0aa3114fbfc9810572e2cf79e6aed597582d).

##Testing the wrong part of the application in the integration tests
When checking what the list of tasks were in the application I generally would find the view, and then get the content. The problem with this is that I'm then checking the controller, instead of checking the view. However, when I make this change in the viewing_task spec, the view is no longer able to directly find the indexOf the task. Instead, I use the itemViewForContentObject method on the collection view to ensure that the object belongs to it.

This still doesn't feel quite right to me in an end-to-end sense, since I'm still poking at the view object which is something that the user isn't going to do directly. However going through the HTML is difficult when your using the desktop views provided with Sproutcore. I think the ultimate solution to this for me is going to be moving away from using those views, and into custom views which I get to set the HTML for so I can directly validate based on css or xpath selectors. I haven't really gotten to try this yet though.

[changes](https://github.com/gmoeck/sc-bdd-todos/commit/3f5848c87950b91fbd7e5e108c6c85cec2343c97)

##Using get for length
Another place that the code that I wrote needs to change is the way that I'm getting the length of todos from the store, and from the list view. One of the standard rules of SC is to use the getters provided by the framework to access the attributes of an object, which I wasn't doing in my tests. In this case, since I was only accessing it in the tests while outside the runloop, I don't think it made any difference in terms of correctness (the tests still passed), but it's still a good practice to do things right, so I've changed it:

<script src="https://gist.github.com/781245.js"> </script>

[changes](https://github.com/gmoeck/sc-bdd-todos/commit/13bedc8cd87b16c2ae487a8519aa56533c61d73a)

##Summary
If nothing else, these changes should demonstrate that I'm no expert in Sproutcore, and I'm certainly open and looking for feedback on where my mistakes are. I hope this series will be helpul as well for those who are attempting to learn like I am. If you spot something that you think can be done better, or that is unclear please feel free to leave a comment or shoot me an e-mail at gmoeck [at] gmail.com.
