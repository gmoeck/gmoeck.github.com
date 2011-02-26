---
layout: post
title: Testing Your Sproutcore Views
---
#(Unit) Testing Your Sproutcore Views
About a week ago there was an [issue](https://github.com/sproutcore/sproutcore/issues#issue/191) created on the sproutcore repository asking about how to programmatically trigger events for unit testing purposes. This created an interesting discussion between Michael Cohen ([frozencanuck](http://frozencanuck.wordpress.com/)) and myself about what is the best way to unit test a Sproutcore view. I figured that this would be a good topic for a blog post, both from the testing perspective and from the Sproutcore perspective. So lets first take a closer look at what our discussion was about.

##The common way of testing views
The approach with Mike articulated seems to be the standard way by which views are tested using sproutcore. Basically what he suggested is that you create and render the view which you want to put under test, and then interact with and assert on html for the view. To understand this better, let's look at an example from Mike's excellent [blog post](http://frozencanuck.wordpress.com/2010/04/06/simulating-events-in-sproutcore/) on simulating events in sproutcore where he is testing a segmented view like the one bellow.

![Segmented View](/images/segmented_view.png)

{% highlight javascript %}
var pane, view;
module("SC.SegmentedView Logic", {
  setup: function() {
    SC.RunLoop.begin();
    pane = SC.MainPane.create({
      childViews: [
        SC.SegmentedView.extend({
          layout: { height: 25 },
          items: 'foo bar'.w(),
        })]
    });
    pane.append();
    SC.RunLoop.end();

    view = pane.childViews[0];
  },

  teardown: function() {
    pane.remove();
    pane = view = null;
  }
});

test("Check that second item is selected correctly", function() {
  var target = view.$('.sc-segment').get(1);
  SC.Event.trigger(target, "mousedown");
  target = view.$('.sc-segment').get(1);
  SC.Event.trigger(target, "mouseup");
  equals(view.get('value'), 'bar', 'the second item should be selected');
});
{% endhighlight %}

Ok, so what is going on here? We're trying to verify that when the second item within the view (the bar section in the picture above) is clicked that it is now the selected item. In order to do this, Mike sets up his view to be nested in a main pane, and renders that pain to the page using the append method of his setup. Then, within his unit test he finds the "bar" section of the page (by knowing that it is the second view with the se-segment class), and triggers a mouse down, followed by a mouse up (since that is the equivalent of a click event in sproutcore). He then asserts the the value for the segmented view is equal to bar, which lets him know that the second section is selected.

##So what is wrong with this?

The first thing that I think needs to be pointed out is that this is not a unit test for the SegmentedView class. The system under test here spans a much larger range than just the SegmentedView code. The fact that we are needing to add other classes in our test setup is the first smell that should indicate this. But let's ignore that for just a moment, and look at what happens in the action which we're testing.

The first thing that we do is we do is ask the DOM for the element which we want to click. Then we send a message to the singleton SC.Event to trigger a "mousedown" event on that dom object - this much should be clear to the code. What happens next though isn't clear from the code. Roughly what happens is the following. SC.Event triggers the event on the DOM node, which is then picked up by the RootResponder. The RootResponder figures out how to pass the message into the system, does so, and eventually the segmented view receives a mouseUp message with the event passed in. The segmented view then handles this message (doing something). This process is illustrated bellow:

![View Events](/images/view_events.png)

The same thing then happens with triggering the mouseup event. And then finally, we assert that the value of the view is 'bar'.

The key thing to notice is that even in this abbreviated sequence how many pieces of sproutcore are we touching? A lot! Instead of isolating our view class to do assertions on it alone, we're using the bigger portion of the stack to "unit" test our view. As such, this isn't really a unit test but an integration/functional test. 

So then how would i test this?

##My way of testing views (Using mocks and stubs for isolation)
What I want to test within the unit test for my view is that it's api functions properly. As such, when I'm testing my view, I don't want to think about DOM events at all, but calls into my api. Some might be confused by this idea, so perhaps it might first be best to ask ourselves what is a view class really?

I think most may think of a Sproutcore view as how their users are going to view the state of their application, but this isn't quite correct. According to the documentation, views provides two functions:

1. They translate state and events into drawing instructions for the web browser.
2. They act as first responders for incoming keyboard, mouse, and touch events.

Really though views doesn't even really talk to the web browser or the DOM directly. They collaborate with a render context, which at some later point in time will be used by the run loop to talk to the DOM directly. This means that I can be assured that the view itself is doing the right thing without ever having to interact with the DOM at all, because the view itself doesn't interact with the DOM. All I need to verify is that the view properly changes it's own state and that it sends the correct message to it's collaborators. Then is it's collaborator's job to do the right thing with that message, not the view itself.

Since the segmented view is a complicated example and I think most people aren't necessarily familiar with the ideas I'm describing, I'm going to give an example with a little simpler situation. Let's take just take a look at a standard button view. The first thing that I would like to happen is when the user mouses over the button, it changes color and when it mouses away, it changes back. 

How would I test this? First I would write a failing integration test (more on that at the end of this post), but then what? Well I want to define the interface to my view such when the mouse enters it, it knows it is being hovered over. So I could write something like the following:

{% highlight javascript %}
var view;
module("MyApp.MyButtonView#mouseEntered", {
  setup: function() {
      view = MyApp.MyButtonView.create();
  }
});

test("When the mouse is hovering over the button", function() {
  var event = {};
  view.mouseEntered(event);

  equals(view.get('isHovering'), true, 'Then the button knows the mouse is hovering over it');
});
{% endhighlight %}

But that doesn't deal with any of the HTML your thinking. That's right, because here we're only dealing with the view as a responder, not as a translator- and yes, we could (should?) break this into two classes each having a single responsibility. The mouseEntered function shouldn't handle the translation into HTML, the render method does. Lets say that html translation of the button being hovered over is to say that the button adds the class 'mouseHovering'. These could be my unit tests:

{% highlight javascript %}
var view, context, addedClass;
module("MyApp.MyButtonView#render", {
  setup: function() {
      view = MyApp.MyButtonView.create();
      context = {addClass: function(str) { addedClass = str; }};
  }
});

test("When the mouse is hovering over the view", function() {
  view.set('isHovering', true);
  addedClass = 'none';
  view.render(context);

  equals(addedClass, 'mouseHovering', 'Then the button has the class mouseHovering');
});

test("When the mouse is not hovering over the view", function() {
  view.set('isHovering', false);
  addedClass = 'none';
  view.render(context);

  equals(addedClass, 'none', 'Then the button does not have the class mouseHovering');
});
{% endhighlight %}

So what I've done here, is I have created a mock object to stand in the place of the render context, but note it is *not* a render context. This is because I'm mocking the role that the render context plays here, not a render context itself. All I'm doing is defining an API for the role that the render collaborator of the view will use. Then I'm essentially just checking that the proper message is sent to the render collaborator, and trusting that it does it's job right when it receives that message(if I were BDDing this, my integration test would still be failing, so those would be the next tests I wrote).By doing this, I've isolated the view from all of the other components of the system, and I'm checking only that the view is functioning correctly. 

If instead of using QUnit, I were to use Jasmine, it's spy framework makes doing this even easier, and in my opinion cleaner:

{% highlight javascript %}
describe('MyApp.MyButtonView', function() {
  var view, context;
  describe('#render', function() {
    var addClassSpy;
    beforeEach(function() {
      view = MyApp.MyButtonView.create();
      context = {addClass: function() {}};
      addClassSpy = spyOn(context, 'addClass');
    });

    describe('when the mouse is hovering', function() {
      beforeEach(function() {
        view.set('isHovering', true);
        view.render(context);
      });

      it('the button has the class mouseHovering', function() {
        expect(addClassSpy).toHaveBeenCalledWith('mouseHovering');
      });
    });

    describe('when the mouse is not hovering', function() {
      beforeEach(function() {
        view.set('isHovering', false);
        view.render(context);
      });

      it('the button does not have the class mouseHovering', function() {
        expect(addClassSpy).not.toHaveBeenCalled();
      });
    });
  });
});
{% endhighlight %}

Perhaps it would be helpful if I were to give one more example to show the power of this approach. Let's say that we're ready to test that on a mouse up, the action for our button is called. For simplicity, let's not make that action dynamic. How would I test that? Like this:

{% highlight javascript %}
describe('MyApp.MyButtonView', function() {
  ...
  describe('#mouseUp', function() {
    var delegate, buttonClickedSpy;
    beforeEach(function() {
      view = MyApp.MyButtonView.create();
      delegate = {buttonClicked: function() };
      view.set('buttonDelegate', delegate);
      buttonClickedSpy = spyOn(delegate, 'buttonClicked');
    });

    describe('with valid conditions for click to happen', function() {
      it('announces that it was clicked', function() {
        expect(buttonClickedSpy).toHaveBeenCalled();
      });
    });
    ...
  });
  ...
});
{% endhighlight %}
Here you can see that it doesn't really even matter what type of class the button is delegating to, all that matters is that it's interface contains the method buttonClicked. This delegate could be a statechart, a responder, a controller, or whatever and the test doesn't care. More importantly the view doesn't care. And it shouldn't, because all we need to see to know that the view is functioning properly is that it is sending the right object the proper message, not what that object is doing with the message.

##So then how do you know if your whole system is working?

One of the things that Mike brought up in our discussion is that he "wants his tests to fully exercise the event handling logic and do it in a way that closely simulates a real world environment within the confines of unit testing". The approach that I'm advocating here does push it a bit further away from a "real world environment", as we're isolating the components for testing purposes. But that is why acceptance/integration tests are so important. 

I do want to verify that the components of my system are working together in a way that provides value to *the consumer* of my code. When I don't isolate my components, it is hard for my tests to truly think about the way that a consumer is going to interact with my system, because my test is focusing on some component's function. However when I have successfully isolated my components with unit tests, my acceptance/integration tests can use the system the same way the user would. So here's an example of an acceptance test for the segmented view that we were talking about earlier, using my jasmine-sproutcore library:

{% highlight javascript %}
describe('Selecting an item from a segmented view', function() {
  var pane;
  describe('Given a page that contains a segmented view', function() {
    beforeAll(function() {
      pane = SC.MainPane.create({
              childViews: [
                SC.SegmentedView.extend({
                  layout: { height: 25 },
                  items: 'foo bar'.w(),
                })]
      });
    });

    afterAll(function() {
      pane.destroy();
    });

    describe('When I am looking at that page', function() {
      beforeAll(function() {
        SC.RunLoop.begin();
        pane.append()
        SC.RunLoop.end();
      });

      afterAll(function() {
        SC.RunLoop.begin();
        pane.remove();
        SC.RunLoop.end();
      });

      describe('And I click on the second item in the segmented view', function() {
        beforeAll(function() {
          clickOn('#barButton'); //You can use whatever CSS selector would be clearest.
        });

        it('Then I should see that the second item is selected', function() {
          //sel is how the segmented view marks what is selected
          expect(SC.CoreQuery('#barButton')).toHaveClass('sel');
        });

        it('And I should see that the first item is not selected', function() {
          expect(SC.CoreQuery('#fooButton')).not.toHaveClass('sel');
        });
      });
    });
  });
});
{% endhighlight %}

The main benefits that I see in this is that it is looking at the problem from a more abstract viewpoint. It both manipulates the page as a consumer would, and verifies that the page is working in a way that the consumer would. It doesn't need to care about the implementation details - since those are covered with unit tests - and so it is free to merely verify that the total functionality is working. And it can speak in the language of the consumer, instead of the developer.

##So why does this discussion matter?
When I was first trying to figure out how to test my views, the first thing I did was go look at how the pre-built views are tested. To my dismay a good number of the pre-built views functionality was not even tested, and even that which was was didn't impress me very much. In fact, the testing as on a whole for the Sproutcore community hasn't really impressed me very much (but I'm pretty spoiled from the Ruby community). This is really rather sad, because Sproutcore makes it **so** much easier to test your applications than a vanilla javascript application. But hopefully this post has helped you to think through how to better test your views, so that our applications will be more easily changeable and cleanly designed in the future. 

If you have any feedback, I would relish challenges in my thinking :) Please feel free to leave some comments.
