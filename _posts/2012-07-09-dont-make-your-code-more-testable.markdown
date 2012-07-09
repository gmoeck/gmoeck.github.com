---
layout: post
title: Don't Make Your Code "More Testable"
---
#Don't make your code "More Testable"
In some ways I'm very excited that the Ruby community seems to have
taken up the "Mocking/OO flag" so to speak lately. Most of the Ruby
conferences seem to have talks around "Object Oriented Rails",
"Isolated Unit Tests", and things like the "SOLID Design Principles". And even
more exciting for me is that I see more and more people referencing the 
*Growing Object Oriented Software Guided By Tests* book. I think these
are all great steps for us towards producing more maintainable software
over time.

That being said, I don't think we've quite understood all of the value
of the techniques that we're all excited about. We've become focused
on our making things "testable" rather than using our tests to learn
about our design. So I want to try and take the opportunity to propose a
slight course correction within the way that the community has been
talking about these principles, and particularly the way that I perceive the
rest of the community receiving them. 

##Example Conversation
I've heard the following conversation happen over and over in various forms
lately:

"Excited OO Enginner" has been trying out some of these new "OO" ideas that have
been such a buzz in the community on his latest Rails project. He has
been creating POROs for all the services that his controllers might want
to access, and he's been writing isolated unit tests for each of these
components. He's worked hard on the project, and he's particularly proud
of how **fast** all of his tests run. All of his previous Rails projects
test suites took minutes, or sometimes even hours to run, but all of the
tests for this project run in under a second.

Then "Experienced Rails Engineer" joins the team, excited to see how the entire
company could learn from this project, particularly its fast test suite. However
as he begins to look at the code, his gut tells him something is
wrong. What would previously have been 10-15 lines of code seems to now
take 20-30. Everywhere he looks, it seems like objects are creating
objects, and he's not even sure why. To make matters worse, in order to
understand the flow though the system he needs to read 5 different files rather
than just one controller file. Unsure of why all this was necessary, he goes to
talk to "Excited OO Engineer".

Experienced Rails Engineer: "Hey, I have a couple of questions about some of 
the code on the new project, you got a sec?"

Excited OO Engineer: "Sure, whats up?"

Experienced Rails Engineer: "Well, as I was reading through the code, it seemed
like there was a lot of additional abstractions that didn't seem necessary to
me, and I'd like to understand why we need it."

Excited OO Engineer: "I'm not sure what you mean. Where do you feel like we've
over abstracted?"

Experienced Rails Engineer: "Like for example, lets take a look at this
PostCommentClass:"

```ruby
class PostComment
  def initialize(user, entry, attributes)
    @user = user
    @entry = entry
    @attributes = attributes
  end

  def post
    @comment = @user.comments.new
    @comment.assign_attributes(@attributes)
    @comment.entry = @entry
    @comment.save!

    LanguageDetector.new(@comment).set_language
    SpamChecker.new(@comment).check_spam
    CommentMailer.new(@comment).send_mail

    post_to_twitter  if @comment.share_on_twitter?
    post_to_facebook if @comment.share_on_facebook?

    @comment
  end

  private

  def post_to_twitter
    PostToTwitter.new(@user, @comment).post
  end

  def post_to_facebook
    PostToFacebook.new(@user, @comment).action(:comment)
  end
end
```

Experienced Rails Engineer: "I could see the necessity of a PostComment object 
if we were going to need to share or reuse the code, but for now it
seems like this could just be put into a controller directly. Why do we need
the notion of a PostComment?"

Excited OO Engineer: "Oh, well you see we need that to be able to test the
code in isolation from other objects. Doing it this way makes it much easier,
and faster to test."

Experienced Rails Engineer: "Ok, well I can kind of see the benefit of that,
but some of this seems really confusing to me. Like why does the PostComment
take a user, an entry and attributes into its constructor?"

Excited OO Engineer: "That's because we want to be able to replace the user,
entry and attributes in our tests with fake objects, so that we can
control the flow through the system here."

Experienced Rails Engineer: "And the same would be true of the language
detector taking a comment? It doesn't make sense to me that something
connected to detecting language would need to know the structure of a
comment."

Excited OO Engineer: "Well you see we want to assert that the comment is
told to set itself to have the proper language, and so that actually is
the responsibility of the language detector. And since we don't want to
have to use a real comment within our tests, we need to pass it into the
constructor."
...

The conversation goes on for 15 or 20 minutes. Every time the
"Experienced Rails Engineer" asks a question about what he feels is a
flaw in the design of the code "Excited OO Engineer" explains that it is
necessary to do things that way in order to write fast, isolated tests.

After the discussion, "Experienced Rails Engineer" concludes that in
order to write an application with fast tests he has to sacrifice his
design, and write 2x as many lines of code. Being the principled
architect that he is, he's not willing to make that compromise. So
he resigns himself to having a slow test suite, so long as he can
continue to write simple, easy to understand code.

##So what's wrong here?
Although I'm quite happy that some of these discussions are actually
happening inside the community, I think that the flavor is somewhat
wrong. The goal of "fast tests" has become an end upon itself, and
people are at risk of destroying their abstractions for the sake of
"testability".

When we encounter a bit of code that is hard to test-assuming we're
letting our tests guide the design of the production code we're going to
write-we're going to have one of two responses. We're either going to
say, "how can I write this in a testable way", or "what about the design
of the system is this test complaining about".

The difference between the two is neuanced, and difficult to grasp when
you first start to think this way. With the first approach my primary
concern is solving the problem of something being difficult to test.
With the second approach my goal is to reevaluate the design of my
system in light of the flaw that my test is presenting to me. The end
goal of the second approach is *the improving of the design of the
system*, whereas the goal of the first approach is *the improving of the
design of the test suite*.

Another way to think about this is to ask what is the connection between
testability and good design. A good design is *necessarily* easily
testable, but does that mean that any design that is "testable" is
*necessarily* good? I don't think that it is.

A good design is measured in things like coupling, cohesion, and how well the
system hides its complexity. Because of their nature, things with high
cohesion and low coupling are almost always very easy to test. As such, when
something is not easy to test, most of the time one can conclude that
the code does not have high cohesion and low coupling. However unit tests
have no hope of telling us if we're doing a good job hiding complexity
behind our APIs, or modeling our domain in an understandable way. Since
by their very nature they should be isolated, they're unable to serve
that function.

As such, when our tests start talking to us, I think we want to listen not
so that we can learn how to test them better, but so that we can learn
about the flaws in our design, and perhaps even our domain knowledge.
They're likely telling us that we're missing something in the way that
we're modeling the problem, and begging us to rethink our approach.

##A Concrete Example
So let's get concrete, and look at the particular problem of slow tests. When 
our test suites get slow, why are they slow, and what can we learn from
that? In my experience the problem is likely one of two things. First, we might 
be needing to instantiate too much of our application in order to get it under
test (and hence the whole thing is coupled together). Or second we're likely 
explicitly dependent on some external bit of code that is notoriously slow,
for instance the database in a Rails application.

In the case of the former, we want to try and separate the concerns of
the application around the lines of the domain. There are likely
concepts that are hidden that are needing to be expressed, named, and
modeled. Unless we do this separation we're not going to be able to learn
about *what* is hidden within the *how*. Once we discover what these
concepts in isolation from the other parts of the system, it should be
easy to model them in a decoupled, and hence testable way.

In the case of the latter, we're not separating out the concerns of
the domain from whatever the concerns of the third party software is.
This means that we're needing to test that domain logic through the
third party API. What's wrong with this? Well, again a part of our
domain is hiding within this subsystem, coupled to this third party API,
hiding the *what* amidst the *how*. Just as in the case above, we want
to extract that domain logic, properly name it, and hide *how* it does
what it does behind the *what* of a clean API.

#And so what now?
And so I want to encourage us within the Ruby community to make this
slight course correction in the way we think about our isolated unit
tests. Let's never have someone walk away from our code bases thinking
that writing fast tests requires introducing abstractions specificallly
for testing purposes. Instead, let's listen to our tests as a means of
improving our domain models, and hiding the complexity of our systems
behind them. Don't be satisfied with your designs just because they're
"more testable".
