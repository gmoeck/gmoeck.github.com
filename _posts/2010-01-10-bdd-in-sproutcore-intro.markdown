---
layout: post
title: BDD in Sproutcore - Introduction
---
#BDD in Sproutcore
Over the last month or so, I've been spending a good amount of my free time using Sproutcore to build some javascript applications. Although I was initially put off by a couple of things, the more I play with the framework, the more I appreciate it. The areas that I originally felt that it was weak have turned out on several occations to be merely places where my thinking about thick client side applications had to change. 

However one thing that I have found mostly lacking in the community is the ability to use BDD (or really any form of TDD)to drive out the implementaion of my applications. For me, this is an absolute must as I'm a big proponent of the craftsmanship movement, and XP. So I've been looking into working out a workflow for doing BDD within Sproutcore.

I post bellow an example of my workflow in building the sample todos application, which most in the community will be familiar with. I offer it as an example for anyone new attempting to do the same, as well as an opportunity for anyone who has already attempted the same to critique and improve my approach. I've chosen to use the QUnit based Sproutcore testing framework for both my end-to-end tests as well as my unit tests. I'm aware of Lebowski, and I've e-mailed Mike Cohen, but it seems (at least at this point) to be a tool that is better fit for verification style testing instead of using the integration tests to drive the unit tests, and features for the application. 

#Breakdown
Initially I had this all in a single post, but it seemed way too long, so I've broken it into pages:

* [Step 1: Viewing an existing task](/2010/01/10/bdd-in-sproutcore-part-1.html)
* [Step 2: Viewing the count of tasks](/2010/01/10/bdd-in-sproutcore-part-2.html)
* [Step 3: Adding a task](/2010/01/10/bdd-in-sproutcore-part-3.html)
