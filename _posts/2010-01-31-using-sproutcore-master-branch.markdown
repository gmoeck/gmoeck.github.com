---
layout: post
title: Using the Sproutcore Master Branch
---
#Using the Sproutcore Master Branch
The question has come up several times in the irc the last few days, and Peter Wagenet keeps answering it, so i figured it would be helpful to write it down.

* First `gem install bundler` if you don't have it
* In your project dir make a file called Gemfile
* Put in `source 'http://rubygems.org'` and `gem 'sproutcore', :git => 'git://github.com/sproutcore/abbot.git'`
* Run `bundle install --binstubs`
* Instead of using `sc-server` use `./bin/sc-server`
