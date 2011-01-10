---
layout: post
title: BDD in Sproutcore - Part 2 ~ Viewing the count of tasks
---
This is a part of an ongoing series on BDD in Sproutcore. You can find the index for the series [here](/2010/01/10/bdd-in-sproutcore-intro.html)
#Step 2 - Viewing the count of tasks
The next feature of the sample todos application that I'd like to implement is the count of todos on the bottom of the page. In order to do this, I'm going to need some more integration tests (apps/todos/tests/integration/viewing_count_of_task.js):

<script src="https://gist.github.com/773028.js"> </script>

This fails with the following errors:

<script src="https://gist.github.com/773034.js"> </script>

This is telling me that the summmary view does not exist yet, so I create it:

<script src="https://gist.github.com/773050.js"> </script>

Which then changes the error message to be the following:

<script src="https://gist.github.com/773226.js"> </script>

This tells us that the summary view doesn't have the binding that we want it to have, so I bind it's value to the controller with a function that does not exist yet, but that I wish it had:

<script src="https://gist.github.com/773059.js"> </script>

Once I do that, Sproutcore again yields no error, but I know that there is no function count_summary on the tasksController, so I write a failing unit test for that function (app/todos/tests/controllers/tasks.js):

<script src="https://gist.github.com/773072.js"> </script>

Which gives the following failure:

<script src="https://gist.github.com/773074.js"> </script>

Which is telling me that I haven't implemented count_summary on the controller, so I implement it:

<script src="https://gist.github.com/773076.js"> </script>

Which changes the failure to:

<script src="https://gist.github.com/773079.js"> </script>

Which I then make pass by modifying tasksController:

<script src="https://gist.github.com/773083.js"> </script>

This makes all the tests pass, which means it's time to clean up the display:

<script src="https://gist.github.com/773087.js"> </script>

###Quick Note
What I'm about to do could(or maybe even should) really be put off until we were adding tasks, but since adding tasks is a bit more complicated, I'm going to show it here for the benefit of the beginner.

###End Quick Note

Now, we want that summary to update when there are tasks, so I add an integration test:

<script src="https://gist.github.com/773106.js"> </script>

In order to accomplish that, I add a unit test for the controller:

<script src="https://gist.github.com/773107.js"> </script>

In order to make that pass, I change the tasksController to be:

<script src="https://gist.github.com/773110.js"> </script>

Now, there is one more case that I want to cover, and that is when is more than one task, so I add an integration test:

<script src="https://gist.github.com/773116.js"> </script>

This fails with the following message:

<script src="https://gist.github.com/773123.js"> </script>

In order to make that pass, I add the following unit test:

<script src="https://gist.github.com/773126.js"> </script>

I use the simplest way I can think of to implement that:

<script src="https://gist.github.com/773130.js"> </script>

And all my tests are passing. However, that count_summary function on the controller isn't really as clean as I would like, so I refactor it to be:

<script src="https://gist.github.com/773134.js"> </script>

And all tests are passing, and when I browse to look at the page, it seems to work as intended when i manually add items into the store.

[Step 3- Adding a task](/2010/01/10/bdd-in-sproutcore-part-3.html)
