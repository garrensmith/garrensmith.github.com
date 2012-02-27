notes:
https://github.com/halorgium/couchdb
http://wiki.apache.org/couchdb/Running%20CouchDB%20in%20Dev%20Mode
http://wiki.apache.org/couchdb/How_to_create_tests?action=show
http://wiki.apache.org/couchdb/How_to_create_tests?action=show

---
layout: post
title: Couchdb Internals - prequel
---

{{ page.title }}
================

<p id="meta" class="meta"> . 26 Feb 2012 - Johannesburg </p>

I'm currently taking the [Introduction to CouchDB Development](http://moodle.wohmart.com/course/view.php?id=2), the first part has been focused on learning Erlang. 
This alone has made the course worth it. I still have a long way in understanding Erlang but I now at least have a basic working knowledge of what its about. 
We have now reached the part where we are digging into the internals of Couchdb and I will be documentating some of my discoveries, misassumptions and bad ideas as I grapple with the source code.

## Setup

To get up and running is pretty state forward. The Couchdb wiki has all this information its just a matter of searching through it to find it. The first thing we want to do
is get [Couchdb running in dev mode](http://wiki.apache.org/couchdb/Running%20CouchDB%20in%20Dev%20Mode). The wiki has a list of dependancies that are needed to be installed before looking at Couchdb. 
Using [Homebrew](https://github.com/mxcl/homebrew) it was pretty simple. 

We need to checkout the code from its [git repository](git://git.apache.org/couchdb.git)

    git checkout git://git.apache.org/couchdb.git
    cd couchdb
    ./bootstrap &&  ./configure
    make dev
    make check

```./bootstrap && ./configure``` sets up the build process. ```make dev``` will create a compiled version of couchdb that can be run by ```./utils/run```, finally ```make test``` will run the javascript tests located at ```/test/javasript```

## Project layout

Most the way that Couchdb is layed out is self explanatory, but the src directory is worth mentioning. 
![/images/couchdb-src](couchdb src directory)

It breaks Couchdb's code up quite nicely. Any directory prefixed with couchdb is couchdb specific code, the other directories are all the 3rd party libraries that Couchdb uses. I will be explaining all this code more in the next couple blogs as I dive more into the code.

## Running the tests
Couchdb is tested in two ways. It has a selection of javascript tests that can be run from the browser by visiting your local [Futon](http://localhost:5984/_utils) and clicking on **Test Suite** and **Run All**. They can also be run 
from the command line ```/test/javascript/run```. There is also a collection of [etap](https://github.com/ngerakines/etap) tests found under,not suprisingly, ```/test/etap```. These can be run individually by ```./test/etap/run``` and the file you want to run eg ```./test/etap/run ./test/etap/001-load.t```. I haven't found a way to run them all at once. Currently I'm not 
sure when you would write a javascript test or an etap test. My best guess would be to use etap to test some of the internal api's that you couldn't get to by writing a javascript test. That would possibly mean that the etap test are
more like unit test while the javascript tests are more functional. 

## Configuration files

Couchdb has two main configuration files, ```etc/couchdb/default.ini``` is the main configuration file with all the standard defaults and configurations. I recommend reading through it as it gives you a good idea of how Couchdb is configured. It also lists all the default handlers.

    [httpd]
     port = 5984
     bind_address = 127.0.0.1
     authentication_handlers = {couch_httpd_oauth, oauth_authentication_handler}, {couch_httpd_auth, cookie_authentication_handler}, {couch_httpd_auth, default_authentication_handler}
     default_handler = {couch_httpd_db, handle_request}
     secure_rewrites = true
     vhost_global_handlers = _utils, _uuids, _session, _oauth, _users
     allow_jsonp = false


located in /etc/couchdb 
    default.ini -- default settings
    local.ini -- override default settings
    



