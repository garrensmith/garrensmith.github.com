---
layout: post
title: Mongoid and Sinatra duet
---

{{ page.title }}
================

<p id="meta" class="meta"> 3 Sep 2010 - Johannesburg </p>

I am slowly catching up with all the latest hotness in the opensource world. I currently am interested in nosql, specifically [Mongodb](http://www.mongodb.org/).
I decided to do a quick demo with it using [Sinatra](http://sinatrarb.com), my favorite web DSL, to see whats involved with installing it and using a ORD with it. I decided to go with [Mongoid](http://http://mongoid.org/), [MongoMapper](http://mongomapper.com/) is also a very good option if you prefer a more ActiveRecord style.

To set it up is very easy, first I installed mongodb via homebrew
        brew install mongodb

This installs mongodb and creates a deamon process so that it is running in the background.
To test that it is working go to [http://localhost:28017](http://localhost:28017).
You should see page giving a the general information. We are now up and running

Now to get it working with Mongoid and Sinatra.
Create a Gemfile with the following:
<script src="http://gist.github.com/575051.js?file=Gemfile-mongoid-sinatra"></script>

Then install the gems:
        bundle install

Lets create a document: (I've created the demo from [Mongoid Documentation](http://mongoid.org/docs/documents/))]
<script src="http://gist.github.com/575052.js?file=person.rb"></script>

Now configure Mongoid and run a test route to get it going
<script src="http://gist.github.com/575054.js?file=mongoid-sinatra-demo.rb"></script>

We are ready to go! Let me know how it goes.



