---
layout: post
title: Why you should care about information hiding
---
#Why you should care about information hiding
As I mentioned in my [last](http://gmoeck.github.com/2011/09/20/why-you-should-care-about-encapsulation.html) 
post, in preparation for my upcoming RubyConf talk  I'm writing a series of
blog posts about the design priciples that drive the use of mock objects. In
that post, I covered what encapsulation is, and why you should care about it. 
Today I want to take up a closely related, yet slightly different topic: Information Hiding.

##What is information hiding?
Before talking about why you should care about information hiding, lets
talk a bit about what information hiding is, and what it is not. When I 
talk about information hiding, I mean that an object hides how it implements its
behavior from the outside world. That is, its API only exposes *what* a given
object does, not *how* it does it. I've found that people are generally
confused as to the benefits of information hiding because the general
example that is given to illustrate information hiding is something like the following:

{% highlight ruby %}
class Encapsulation
  def initialize(value)
    @value = value
  end

  def set_value(value)
    @value = value
  end

  def get_value
    @value
  end
end
{% endhighlight %}
There are some benefits to doing something like this (you could have
value be computed somehow for instance), but this isn't really
accomplishing information hiding as I described above. It's really
providing an API to modify the data which the object is "encapsulating".
The object isn't really exposing behavior as much as it is holding
data. And as such, in order to use the object, you still have to keep in
mind the low level details of how the object works. Any code that
does not hide it's information behind a solid API is inherantly a [leaky
abstraction](http://www.joelonsoftware.com/articles/LeakyAbstractions.html).


##Why should I care?
Maybe it's just me, but when my applications get large I find it
difficult to keep all of the details about how everything works in my
head. Even when I'm the one who has written all of the code, it
eventually becomes too much, and it only gets
worse when there are multiple developers working in the same code base.
But when the implementation of *how* an object accomplishes what its API
promises to do is hidden, it enables us to treat the object essentially as a black
box. We can trust that the object will do what it is supposed to do,
even if we don't know how it does it.

The biggest benefit to having good information hiding in my mind is that
it allows you to build abstractions that allow you to work on higher
and higher layers because you can ignore details that are not related to
what your *currently* working on. Almost every programmer expects this out of their
libraries, but rarely do they expect it from their own code.

Consider the following ticket machine interface:

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

The key thing to notice is that the API exposes behavior instead of
data, or implementation. In order to use the object, you need only to
know it's API, and give it a ticket reserver on creation. Notice also,
that the "ticket reserver" is merely a role that another object is going
to play. From the perspective of the ticket machine, all it cares about
is that it implements an api to reserve tickets. It could do that by
communicating over HTTP, writing to a database, or whatever. Since the
ticket reserver seems to have hidden it's information as well, the
object is just a role, and any object which implements the role's API
can just step in.

##Isn't this just encapsulation?
You can see that one of the nice side effects of good information hiding
is that it tends to produce well encapsulated code. If your API does not
expose how it does what it does, it is very difficult to change the
behavior of an object without calling its API. However as [Nat Price](http://nat.truemesh.com/archives/000498.html)
has pointed out, the two still have different goals, and it can be
helpful to focus on only one of thos goals at a time.

##What does this have to do with Mock Objects?
I think that the above ticket machine interface code is a good example
of encapsulated code that hides its information well. Now if you were
to follow the state based sort of unit testing approach, how would you
write a test for that object? I think the only way you could is to add
getters or setters for the information that your attempting to "hide".
Whereas with a mock object, you would write a test something like the
following:

{% highlight ruby %}
describe TicketMachineInterface do
  it "reserves the number of tickets inputted when the user submits a
request" do
    ticket_reserver = double('ticket_reserver')
    ticket_reserver.should_receive(:reserve).with(55)

    machine = TicketMachineInterface.new(ticket_reserver)
    machine.number_pressed(5)
    machine.number_pressed(5)
    machine.submit_request
  end
end
{% endhighlight %}

##Conclusion
If your interested further, I'm going to be giving my RubyConf talk on ["Why You Don't Get Mock
Objects"](http://rubyconf.org/presentations/21), which will focus on
these issues some more. I'll also likely follow up with some more blog
posts after the fact.

I welcome any feedback you have in the comments.
