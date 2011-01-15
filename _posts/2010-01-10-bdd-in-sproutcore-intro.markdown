---
layout: post
title: BDD in Sproutcore - Introduction
---
#BDD in Sproutcore
Over the last month or so, I've been spending a good amount of my free time using Sproutcore to build some javascript applications. Although I was initially put off by a couple of things, the more I play with the framework, the more I appreciate it. The areas that I originally felt that it was weak have turned out on several occasions to be merely places where my thinking about thick client side applications had to change. 

However one thing that I have found mostly lacking in the community is the ability to use BDD (or really any form of TDD)to drive out the implementation of my applications. For me, this is an absolute must as I'm a big proponent of the craftsmanship movement, and XP. So I've been looking into working out a work-flow for doing BDD within Sproutcore.

I've put together an example of my work-flow in building the sample todos application. I've chosen to re-implement the functionality of this system because the explanations of the Sproutcore code themselves are available elsewhere, and as such I won't have to to spend much (any?) time explaining how the Sproutcore code is working. If your not sure what the Sproutcore code is doing, I would highly recommend looking at any accompanying information from the wiki which you can view by clicking [here](http://wiki.sproutcore.com/w/page/12413071/Todos%C2%A0Intro). Anything that isn't contained there though I am planning on explaining. I'm also attempting to keep the code as clean as possible, so hopefully it's intent will be self explanatory when combined with the tests.

I offer this series as an example for beginners who are interested in using BDD principles to drive Sproutcore applications, as well as an opportunity for anyone who has already attempted the same to critique and improve my approach. I genuinely would like criticism as to where both my Sproucore skills, and BDD implementation are lacking.

I've chosen to use the QUnit based Sproutcore testing framework for both my end-to-end tests as well as my unit tests. I'm aware of Lebowski, and I've tried using it, but it seems (at least at this point) to be a tool that is better fit for verification style testing instead of using the integration tests to drive the unit tests, and features for the application. 

#Breakdown
Initially I had this all in a single post, but it seemed way too long, so I've broken it into pages:

* [Step 1: Viewing an existing task](/2010/01/10/bdd-in-sproutcore-part-1.html)
* [Step 2: Viewing the count of tasks](/2010/01/10/bdd-in-sproutcore-part-2.html)
* [Step 3: Adding a task](/2010/01/10/bdd-in-sproutcore-part-3.html)
* [Corrections for steps 1-3](/2010/01/15/bdd-in-sproutcore-corrections-for-1-3.html)

