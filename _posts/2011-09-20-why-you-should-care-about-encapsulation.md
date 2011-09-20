---
layout: post
title: Why you should care about encapsulation
---

#Why you should care about encapsulation
As I've been preparing to give a talk entitled ["Why You Don't Get Mock
Objects"](http://rubyconf.org/presentations/21) at this year's RubyConf,
I've begun to change my mind a little bit about why the Ruby community
scorns mock objects. I used to think that the community just didn't
understand how mock objects are meant to be used, but I've come be
believe there is another contributing factor. I think that the reason
the community doesn't really get mock objects is because they don't
really understand the principles that drove the authors of mock objects
to create them to begin with. Things like encapsulation, information
hiding, and the law of demeter are not really well understood, which
makes the motivation behind mock objects very difficult to grasp. As
such, I want to try and sketch out some of these design principles in
some blog posts as a way to "till the soil", and motivate why someone
would bother to create mock objects in the first place. This is the
first of those blog posts, where I'm going to attempt to lay out the
idea of encapsulation, and why you should care about it.

##What is encapsulation?
Before we talk about why you should care about encapsulation, it would
be good define exactly what I mean by the term. In my experience people often confuse
encapsulation with information hiding, but I think there is a subtle difference
between the two. For the purpose of this blog post, when I say
encapsulation, I mean "the behavior of an object can only be affected
through its API". Or to put it negatively, an object is not well
encapsulated when it's behavior can be affected without calling it's
API. Or another way to think about it is that a well encapsulated object draws
a solid boundry or wall around itself, and ensures that all the code that
changes it's behavior is contained within the object itself.

Consider the following example:

{% highlight ruby %}
class TrainTicketStation
  def initialize(train)
    @train = train
  end

  def buy_ticket
    available_seats = @train.seats.reject { |seat| seat.reserved? }
    raise "TrainIsFull" if available_seats.length < amount
    available_seats.first.reserved = true
  end
end

class TrainTicketWebsite
  def initialize(train)
    @train = train
  end

  def buy_tickets(number_of_seats)
    available_seats = @train.seats.reject { |seat| seat.reserved? }
    raise "InsuffecientSeats" if available_seats.length < number_of_seats
    seats_to_reserve = available_seats.slice(0, number_of_seats)
    seats_to_reserve.each { |seat| seat.reserved = true }
  end
end
{% endhighlight %}

If we were to have a station and a website both referencing the same
train, then the behavior of both can be affected without touching their public
API. For example, by reserving seats through the website, we can
modify the behavior of the buy_ticket method on the ticket station
object.

This is because both objects make decisions about reserving seats based upon the
state of their shared train object. And since the state of that train
object is mutable, when one object changes that state, it necessarily
impacts the behavior of the other object.

##Why should I care?
The issue with poorly encapsulated code is that when we go to make a
change, we have to spend a good amount of time tracing what the potential
effects of this change might be on the system as a whole. And the bigger
the system, the bigger this problem becomes. If you have ever worked in a system
where you were afraid to make a change to the behavior of one section of the
code base because you didn't know if that change would break other places,
you've experienced the pain of poorly encapsulated code. To put it in the language
of the definition of encapsulation that I offered earlier, what you were really
worried about that in making this change, you were then changing the behavior
of the system in another place.

So for example, if we wanted to change the above example from keeping an array
of available seats to just keep a count of the available seats, we would
have to change the code in two places. The buy_ticket method inside of
the train ticket station, and the buy_tickets method inside of the train
ticket website. And if we origionally made the change inside of the
train ticket station, and forgot that the train ticket website was
dependent upon that, then the application would break.

However when code is well encapsulated, it is like having a wall around
its behavior, and the only things that can modify that behavior are contained
within the wall. With the "wall" of encapsulation in place, we don't
have to worry about if we're changing the behavior of another place in
our system, because we know it is "outside the wall".

So for example, imagine that we refactored the origional code to be the following:

{% highlight ruby %}
class TrainTicketStation
  def initialize(train)
    @train = train
  end

  def buy_ticket
    @train.buy_tickets(1)
  end
end

class TrainTicketWebsite
  def initialize(train)
    @train = train
  end

  def buy_tickets(number_of_seats)
    available_seats = @train.seats.reject { |seat| seat.reserved? }
    raise "InsuffecientSeats" if available_seats.length < number_of_seats
    seats_to_reserve = available_seats.slice(0, number_of_seats)
    seats_to_reserve.each { |seat| seat.reserved = true }
  end
end

class Train
  def buy_tickets(amount)
    raise "InsuffecientSeats" if available_seats.length < amount
    seats_to_reserve = available_seats.slice(0, number_of_seats)
    seats_to_reserve.each { |seat| seat.reserved = true }
  end

  private
  def available_seats
    @seats.reject { |seat| seat.reserved? }
  end
end
{% endhighlight %}

Now, if we wanted to make a change to only keep track of the number of
available seats, instead of using an actual array of seats, the changes are all
located inside of the train object. And we can modify the train, without
having to modify the two ways to purchase tickets as follows:

{% highlight ruby %}
class Train
  def initialize(seats)
    @available_seats = seats
  end

  def buy_tickets(amount)
    raise "InsuffecientSeats" if @available_seats < amount
    @available_seats -= amount
  end
end
{% endhighlight %}

##How can we maintain encapsulation?
One of the primary ways that one can move towards well encapsulated code
is to follow the "Tell, Don't Ask Principle", which is what we did in
the code above. Instead of asking an object for it's properties and then
making decisions based on that information, objects communicate with
each other by telling other objects to do something. We make it such
that the only properties that can affect the behavior of the object are
contained within the object itself. Any time something needs to happen
that operates on the data "outside our wall", we tell the object that is
responsible for that data to do it for us. For example in the above example, 
instead of asking the train for it's seats, and then making decisions
based on what we get back, we merely tell the train to reserve the
proper number of seats, and let it operate on it's own data.

There are of course more design techniques that you can use to help
maintain encapsulation. A couple of them are avoiding global variables
or singletons, copying collections or mutable value objects(entities), and
defining imutable value types. But those are topics for another post.

For now at least I hope you've come to understand encapuslation a bit
better, and why it's a helpful thing to have. I welcome feedback in the
comments.
