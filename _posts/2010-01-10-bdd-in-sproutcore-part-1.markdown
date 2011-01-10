---
layout: post
title: BDD in Sproutcore - Part 1 ~ Viewing an existing task
---
This is a part of an ongoing series on BDD in Sproutcore. You can find the index for the series [here](/2010/01/10/bdd-in-sproutcore-intro.html)
#Step 1: Viewing an existing task
The first thing that I would like to do is have the page display an existing task. So I start with the following end-to-end test within a blank project (apps/todos/tests/integration/viewing_task.js):

<script src="https://gist.github.com/772850.js"> </script>

Within the setup, I'm starting the actual application within the page, and creating the existing record which I want to display. Within the teardown, there are two steps that I've found are necessary to "stop" the application. The first is to remove the pane, and the second is to clear the database.

To begin with here, in order to check that the item is on the page, I'm merely testing that it is within the content of the listView which is going to display the item. I could check here the display is showing the description of the item, and that the checkbox is not checked, but that's a bit more details than I care about at this point. 

In order to run the test, I start up my sc-server, and goto http://localhost:4020/todos/en/current/tests.html. Two tests should initially fail:

<script src="https://gist.github.com/772897.js"> </script>

The first failure is telling me that Todos.Task doesn't exist, so I have to create it (apps/todos/models/task.js):

<script src="https://gist.github.com/772892.js"> </script>

I rerun my tests, and see that the first failure is now passing, leaving me only with the second:

<script src="https://gist.github.com/772899.js"> </script>

This failing test tells me the path for the requested view does not exist, so I have to modify apps/todos/resources/main_page.js to include it:

<script src="https://gist.github.com/772909.js"> </script>

This changes the failure to the following:

<script src="https://gist.github.com/772921.js"> </script>

This is telling me that the content of the ListView is null. So what I need to do is bind the contentView to a controller for the tasks, so I do that by modifying the main_page again:

<script src="https://gist.github.com/772927.js"> </script>

Here however when I run my tests again, Sproutcore does not give me any kind of error, it just fails silently. I'm not sure why this is (if you have more knowledge of the framework, please enlighten me) because Todos.tasksController doesn't actually exist. But in any case I need to create it (apps/todos/controllers/tasks.js):

<script src="https://gist.github.com/772931.js"> </script>

When I rerun my tests, I get a different failure:

<script src="https://gist.github.com/772937.js"> </script>

So now my test is telling me that the item is not found in the view. This is because the controller has not had it's content to be set to use Todos.task. In order to ensure that this is happening, I write the following unit test (apps/todos/tests/main_test.js):

<script src="https://gist.github.com/772970.js"> </script>

In order to make this pass, within the setup I delegate to the store to find the proper object:

<script src="https://gist.github.com/772975.js"> </script>

That makes the unit test that we wrote pass, but the integration test is still failing, so we need to delegate to the controller to set it's content. Here's the unit test to specify that:

<script src="https://gist.github.com/772982.js"> </script>

And to make this pass, I need to implement that delegation:

<script src="https://gist.github.com/772988.js"> </script>

And that makes all of our existing tests (both unit and integration) pass. 

##Reflection on mocking problems.

To be honest, I'm not entirely happy with the mocking that I just did, because I wasn't able to validate what was contained within the messages that were going between the objects. The mock really should be able to do something like wasCalledWith, and give it a parameter. That's what I would generally do in something like Jasmine, but I haven't worked out how to integrate a solid mocking or spying framework with Sproutcore yet. 

Also, I really should have written the delegation to the controller first, but since I knew I couldn't validate what it was called with it would have missed testing that the main function delegated to the store to find the tasks, so I broke the true BDD cycle there. But, I guess it works for now.

##One last thing before moving on
At this point, the view - although it is fully functional - is still quite ugly. Since this is purly aesthetic, I like to wait to style until all my tests are passing. I change main_page.js to be the following:

<script src="https://gist.github.com/773047.js"> </script>

Generally, since I tend to think of the binding to the description and the value as functionality, I think I would want an integration test which checks both of those values. However for the sake of brevity (this post is already quite long), I'll leave out those tests.

[Step 2- Viewing the count of tasks](/2010/01/10/bdd-in-sproutcore-part-2.html)

