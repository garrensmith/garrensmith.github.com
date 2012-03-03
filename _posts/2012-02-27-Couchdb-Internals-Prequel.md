---
layout: post
title: Couchdb Internals - The Prequel
---

{{ page.title }}
================

<p id="meta" class="meta"> . 27 Feb 2012 - Johannesburg </p>

I'm currently taking the [Introduction to CouchDB Development](http://moodle.wohmart.com/course/view.php?id=2). The first part, obviously, was focused on learning Erlang. 
This alone has made the course worth it. I still have a long way to go in understanding Erlang but I now at least have a basic knowledge of what it's about. 
We have now reached the part where we are digging into the internals of Couchdb and I will be documentating some of my discoveries, assumptions and bad ideas as I grapple with the source code.

Before I start diving into the internals of Couchdb I want to understand how to get Couchdb up and running, how to run its tests, the layout of the code and how to configure it.

## Setup

To get up and running is pretty straight forward. The Couchdb wiki has all this information it's just a matter of searching through it to find it. The first thing we want to do
is get [Couchdb running in dev mode](http://wiki.apache.org/couchdb/Running%20CouchDB%20in%20Dev%20Mode). The wiki has a list of dependancies that need to be installed before looking at Couchdb. 
Using [Homebrew](https://github.com/mxcl/homebrew) it was pretty simple. 

We need to checkout the code from its [git repository](git://git.apache.org/couchdb.git)

    git checkout git://git.apache.org/couchdb.git
    cd couchdb
    ./bootstrap &&  ./configure
    make dev
    make check

```./bootstrap && ./configure``` sets up the build process. ```make dev``` will create a compiled version of couchdb that can be run by ```./utils/run```, finally ```make check``` will run the javascript tests located at ```/test/javasript```

## Project layout

The layout of the code is pretty self-explanatory, with the ```src``` directory being the meat of the project. 

![couchdb src directory](/images/couchdb_src.png)

This folder breaks Couchdb's code up quite nicely. Any directory prefixed with couchdb is couchdb specific code, 
the other directories are all the 3rd party libraries that Couchdb uses. I will be explaining all this code more in the next couple blogs as I dive into it.

## Running the tests
Couchdb is tested in two ways. It has a selection of javascript tests that can be run from the browser by visiting your local [Futon](http://localhost:5984/_utils) and clicking on **Test Suite** and **Run All**. 
They can also be run from the command line ```/test/javascript/run```. There is also a collection of [etap](https://github.com/ngerakines/etap) tests found under, not suprisingly, ```/test/etap```. 
These can be run individually by ```./test/etap/run``` and the file you want to run eg ```./test/etap/run ./test/etap/001-load.t```. I haven't found a way to run them all at once. Currently I'm not 
sure when you would write a javascript test or an etap test. My best guess would be to use etap to test some of the internal api's that you couldn't get at by writing a javascript test. That would possibly
 mean that the etap tests are more like unit tests while the javascript tests are more functional. 

## Configuration files

Couchdb has two main configuration files, ```etc/couchdb/default.ini``` is the main configuration file with all the standard defaults and configurations. I recommend reading through it as it gives you a good idea of 
how Couchdb is configured. It also lists all the default handlers.

    [httpd]
     port = 5984
     bind_address = 127.0.0.1
     authentication_handlers = {couch_httpd_oauth, oauth_authentication_handler}, \
        {couch_httpd_auth, cookie_authentication_handler}, \
        {couch_httpd_auth, default_authentication_handler}
     default_handler = {couch_httpd_db, handle_request}
     secure_rewrites = true
     vhost_global_handlers = _utils, _uuids, _session, _oauth, _users
     allow_jsonp = false

The ```[httpd]``` section shown above can be found in ```default.ini```. What is interesting for the ```default_handler``` and ```authentication_handlers```, if you wanted to, you could write your own handlers. You could 
then change the handler config to use your one instead. 

The other important config file is ```local.ini```, this is used for overriding any settings found in ```default.ini```. From what I understand of it, you can add your own handlers there for specific requests, set a different port address and a multitude of other options.

## Going forward
I found that before I dive into code I always need to understand how the project works, hence this post. Maybe I need to see the forest before looking at individual trees. Exploring the code and writing this
 post helped me quite a bit. Leave a comment if I have missed anything or have documented something wrong.


