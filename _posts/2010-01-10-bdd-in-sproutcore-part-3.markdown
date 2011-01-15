---
layout: post
title: BDD in Sproutcore - Part 3 ~ Adding a task
---
This is a part of an ongoing series on BDD in Sproutcore. You can find the index for the series [here](/2010/01/10/bdd-in-sproutcore-intro.html)
##Step 3 - Adding a task
The next thing that I would like to be able to do with the todos application is to be able to add a task. In order to accomplish this, I write the following integration test:

<script src="https://gist.github.com/773146.js"> </script>

The only thing new here is the simulating of a mouse click on the button within the run loop inside the test. Within sproutcore, there is no such thing as a native "click" event. Instead, Sproutcore registers a click on a mousedown event, followed by a mouseup. Mike Cohen(FrozenCanuck) has a great blog post explaining the idea, which you can view [here](http://frozencanuck.wordpress.com/2010/04/06/simulating-events-in-sproutcore/). For now, know this is how I simulate a clicking on the button.

This generates the following failure:

<script src="https://gist.github.com/773157.js"> </script>

This means that the page does not have the button currently on it. I add it to the main_page.js:

<script src="https://gist.github.com/773164.js"> </script>

This then changes the error to read:

<script src="https://gist.github.com/773171.js"> </script>

This tells me that the button is not doing what it is supposed to be doing (adding a task), so I need to implement that functionality. In order to do that, I need to setup the button to target the tasksController, with an action called addTask, which does not yet exist. I modify main_page.js to be the following:

<script src="https://gist.github.com/773177.js"> </script>

Again, at this point Sproutcore does not give an error saying that the specified bind function does not exist, but I know that it doesn't, so I add a unit test to implement this functionality:

<script src="https://gist.github.com/773180.js"> </script>

This then fails with the following:

<script src="https://gist.github.com/773185.js"> </script>

I add the function to the controller:

<script src="https://gist.github.com/773191.js"> </script>

Which changes the failure to the following:

<script src="https://gist.github.com/773196.js"> </script>

To get that test to pass, I add implement the function:

<script src="https://gist.github.com/773204.js"> </script>

This causes both my unit tests, and my integration tests to pass, leaving me in the green. As such, it's time for me to clean up the view a bit.

<script src="https://gist.github.com/773207.js"> </script>

[Corrections for steps 1-3](/2010/01/15/bdd-in-sproutcore-corrections-for-1-3.html)
