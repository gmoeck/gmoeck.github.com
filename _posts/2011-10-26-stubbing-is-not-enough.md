---
layout: post
title: Stubbing Is Not Enough
---
#Stubbing Is Not Enough
There has been a good amount of buzz within the Rails community lately
about "Isolation Testing", "Mock Objects", and "Fast Tests". I think
that this is a wonderful trend. Particularly becuase of Corey Haines'
emphasis that practicing Test-**Driven** development means that you need
to listen to your tests in such a way that pain causes you to change
your *design* rather than your *tests*. It has been a long time coming
that Rails developers would begin to listen to the pain coming from
their tests, start to use test doubles, and start breaking dependencies
within their design.This is wonderful, and I applaud the effort.

Yet at the same time, there are a number of ways in which the community
can still press listening to their tests further. For example, although
"mock objects" have become almost a buzz word recently within the community, the
actual objects that are being created by and large are stubs, not mocks. This means
that although we're isolating our tests so that they run faster, we're
not completely dealing with the underlying coupling that is pulling in
all these dependencies, and making our tests slow.

This post is my attempt to lay out how I think we can both improve the
speed of our tests, while at the same time improving the quality of our
architectures.

##Mocks Are Not Stubs, And Stubs Are Not Mocks
The first clarification that I think is necessary within the community
is that there is a difference between "mocks", and "stubs". Although
they both replace a real object within a test, they do different things.
Gerard Meszaros' definition within his book *xUnit Test Patterns* is
helpful here:

"A **Test Stub** is an object that replaces a real component on which the
SUT [system under test] depends so that the test can control the
indirect inputs of the SUT. It allows the test to force the SUT down
paths it might not otherwise exercise...

A **Mock Object** is an object that replaces a real component on which the
SUT depends so that the test can verify its indirect outputs."

The key difference is in the way you use the fake object you create.

You would use a stub when a method depends on another object for data in the
form of a query. Instead of having to setup a real collaborating object
(which might include hitting the database, etc), you replace that object
with another object that only returns the data that you need, so that
you can avoid having to do the setup. By doing this, you isolate
the object under test, because the only dependency that the test needs
to run is the code for the object itself.

This is what Corey Haines essentially does in his ["Fast Rails Tests"](http://confreaks.net/videos/641-gogaruco2011-fast-rails-tests)
talk. First instead of having a method directly inside of his
ActiveRecord object, he moves it into a module, that looks like the
following:

{% highlight ruby %}
class ShoppingCart < ActiveRecord::Base
  include ShoppingCartExtensions::CalculatesTotalPrice

  has_many :shopping_cart_products, dependent: :destroy
  has_many :products, :through => :shopping_cart_products
end

module ShoppingCartExtensions
  module CalculatesTotalPrice
    def total_price
      products.map(&:price).inject(0,&:+)
    end
  end
end
{% endhighlight %}

Then in order to test the object in isolation he includes that mixin in
a dummy object inside of his tests like this:

{%highlight ruby %}
class DummyCartWithProducts
  include ShoppingCartExtensions::CalculatesTotalPrice
end

describe "Calculating Total Price" do
  it "returns the total price of the products" do
    cart = DummyCartWithProducts.new
    products = [stub(price: 5), stub(price:10)]
    cart.stub(:products) { products }
    cart.total_price.should == 15
  end
end
{% endhighlight %}

Essentially what he's done is treated the ActiveRecord part of the
shopping cart as a collaborator, and created a stub method for its
products. This way he doesn't have to add products into the database
and then read them out just in order to be able to test that the cart
can properly calculate its total price. And as you might have already
guessed, the benefit that comes from the stubs is speed.

##Why Stubs Are Not Enough
The primary problem that people generally point out in relation to this
form of stubbing however is that it leaves you with "brittle tests"
which will give you either false positives, or false negatives. So for
example (and I know this is a bit contrived, but the total price method
is fairly simple) imagine that I changed price to be a money object that
returned the currency type, and the value. What would happen to the
calculating total price method? Well, since the test for the total price
method doesn't use a real product object the test is going to stay
green even though when it is used in production it is going to explode.
These sorts of examples can also go the other way as well- although I
can't think of a way to do this for the shopping cart example - where
the tests break in a refactoring, even though the actual behavior of the
system hasn't changed.

One might initially think that this whole "fast tests" thing isn't worth
it if it means that our tests are going to become brittle. However the
right response is to go back to how Corey starts his presentation, by
noticing the difference between Test-**Driven** Development, and
Test-**First** Development. Practicing Test-**Driven** development means that you need
to listen to your tests in such a way that pain causes you to change
your *_design_* rather than your *tests*.

So what is wrong with the design of a system that requires stubs in
order to test its objects in isolation? That problem is almost always
that the system has poor encapsulation. Why? When you need to stub an
object, it means that your going to be querying some dependency for
information in order to test the behavior of the object. This means that
the behavior of whatever object your testing is *necessarily* dependent upon the
behavior of whatever object your stubbing.Now in some cases, maybe this
isn't that bad, but in others this coupling is going to cause a ripple 
of changes throughout your system whenever you have to change the object
that needs to be stubbed. This is a flaw in the design, not the tests.

##How Mocks Solve This Problem

The solution to the design problem is to encapsulate our objects so
that their behavior can only be affected through their public API. One
of the primary ways that we can accomplish this is to follow the ["Tell,
Don't Ask"](http://pragprog.com/articles/tell-dont-ask) principle. That is, instead of asking an object for its
properties and then making decisions based on that information, we want
our objects to communicate with each other by telling other objects to
do something. Any time something needs to happen that has to operate on
the data outside of the wall of our object, we delegate that to the
object that is responsible for that data to do it for us.

Another way to think about this is that any objects collaborators should
be playing a different role than the object itself, and each object
should have only a single responsibility. If in the course of developing
our objects we run into another responsibility/role we want to delegate
that task to the object that is playing that role. If you want more
information about how this solves the problem, you should read my
articles, ["Why you should care about encapsulation"](http://gmoeck.github.com/2011/09/20/why-you-should-care-about-encapsulation.html),
and ["Why you should care about information hiding"](http://gmoeck.github.com/2011/09/28/why-you-should-care-about-information-hiding.html).

This then brings us back to using mock objects instead of stubs inside
of our tests. Remember that Meszaros defined a mock object as an object
that "replaces a real component on which the SUT depends so that the
test can verify its indirect outputs". What he is saying is that a mock
object stands in for one or more of the roles of an object's
collaborators in order for us to verify the *commands* sent to that
object. That is, since some actions do not necessarily modify the state
of the isolated object under test, the way that we want to do our
assertion is if it sends the right command to its collaborator.

So for example, the following ticket machine interface is an example of
a well encapsulated code:

{% highlight ruby %}
class TicketMachineInterface
  def initialize(ticket_reserver)
    @ticket_reserver = ticket_reserver
    @current_display = ""
  end

  def number_pressed(number)
    @current_display += number.to_s
  end

  def delete_pressed
    @current_display.chop!
  end

  def submit_request
    @ticket_reserver.reserve(@current_display.to_i)
  end
end
{% endhighlight %}
If I wanted to test this object in isolation, how would I do that? Well,
instead of passing in a real object that would play the role of the
ticket reserver, I could pass in a mock object that would allow me to
assert on the messages that was sent to it. So for example, my test
would look something like the following:

{% highlight ruby %}
describe TicketMachineInterface do
  it "reserves the number of tickets inputted when the user submits a request" do
    ticket_reserver = double('ticket_reserver')
    ticket_reserver.should_receive(:reserve).with(55)

    machine = TicketMachineInterface.new(ticket_reserver)
    machine.number_pressed(5)
    machine.number_pressed(5)
    machine.submit_request
  end
end
{% endhighlight %}

The assertion here is found in the code "ticket_reserver.should_receive(:reserve).with(55)".
Instead of asserting on the state of the object, I assert on its
behavior. This then allows my test to respect the encapsulation of my
object, and isolate it from its dependencies (which means it runs fast),
while not making my tests brittle.

The only key becomes mocking roles, instead of objects so that we can
let the object decide *how* to accomplish what it is told to do. And
when we follow this "Tell, Don't Ask" style of design it produces
flexable code because it then becomes easy to swap or compose objects
that play the same role in order to change the behavior of the system.

##Conclusion
As I said at the outset, I'm quite happy that there seems to be an
increasing interest in Test-**Driven** Development, and feeling the pain
within our tests. I hope this post has given you something to think
about, and some ways to think about improving the speed of your test
suite without creating brittle tests. I'd love any feedback in the
comments.
