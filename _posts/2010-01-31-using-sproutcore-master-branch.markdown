---
layout: post
title: Using the Sproutcore Master Branch
---
#Using the Sproutcore Master Branch
The question has come up several times in the irc the last few days, as to how to use the Sproutcore master branch. Peter Wagenet keeps answering it, and i figured it would be helpful to write it down.

##NOTE
The following is currently only working on Ruby 1.9.2. The easiest way to set that up is to use [rvm](http://rvm.beginrescueend.com/interpreters/ruby/).

* First `gem install bundler` if you don't have it
* In your project dir make a file called Gemfile
* Put in `source 'http://rubygems.org'` and `gem 'sproutcore', '1.5.0.pre.3'`
* Run `bundle install --binstubs`
* mkdir frameworks && cd frameworks
* git clone git://github.com/sproutcore/sproutcore.git
* Instead of using `sc-server` use `./bin/sc-server`

###Windows
Windows also needs to add `gem 'eventmachine', :git => 'git://github.com/eventmachine/eventmachine'` because otherwise it won't compile

##UPDATED 2011-02-22

