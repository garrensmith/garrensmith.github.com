---
layout: post
title: Staying up with Unicorn, Upstart and Bluepill
---

{{ page.title }}
================

<p id="meta" class="meta"> 24 Sep 2012 - Johannesburg </p>

Deploying your new Rails application is quite easy and there is plenty of documentation on the web. However keeping it running is a little trickier. I'm no Linux system admin guru so when I launched [Classroom 7](http://www.classroom7.com)
I chatted to as many people as I could to make sure I was doing the write things on my servers. I was lucky enough to get Glenn and Dalibor of [Siyelo](http://www.siyelo.com), a great Rails consultancy based in Cape Town,  to do a review of the Classroom 7 server. 
They wrote a great blog [post](http://blog.siyelo.com/rails-deployment-audit) about it. From their review I seemed to be on the right path but one of the things they did mention was that if my server rebooted my Rails app would not automatically start. Last week I decided to rectify that problem. Below is the solution I came up with.

Classroom 7 has two main components, a Rails app which is the main app and Resque for some background tasks. The main requirements I had was that if anything weird happened to my processes that they would be restarted and if my server was restarted that my processes
would automatically start. 

## Bluepill

I initially took a look at [Monit](http://mmonit.com/monit/), [God](http://godrb.com/) and [Bluepill](https://github.com/arya/Bluepill) for process monitoring of Classroom 7. I've used Monit before and although really powerful I found it quite difficult to setup, God seemed good but I've heard of memory leak issues
and didn't feel like having something else monitoring my monitoring process. Bluepill was the eventual winner. Bluepill is really easy to install just follow the [README](https://github.com/arya/Bluepill/blob/master/README.md).
Below is my Bluepill configuration file. I created seperate ones for Production and Staging. 

<script src="https://gist.github.com/3777350.js?file=Bluepill_config.rb"></script>

The configuration monitors two different apps, Unicorn to serve my rails app and Resque for my background processes. With Bluepill's great DSL the configuration should be quite self explanatory but one thing you need to be aware of
is setting the `grace_time` quite high. If the `grace_time` is lower than the time it takes to start rails, Bluepill fails and cant monitor your app. It took me a couple tries and a fair amount of googling to sort that out. A fair amount of this code could
be DRYed up at this point I dont want to change a winning game. 

To test that your Bluepill configuration is working you can start Bluepill with your configuration file `sudo Bluepill load path/to/my/app/current/config/Bluepill_configuration.pill`. Once its up run `sudo Bluepill status` to see how the processes are doing.

## Upstart

Now that Bluepill is running we can relax in the knowledge that anytime of the processes stops or goes out of control, Bluepill will handle it for us. The only problem is that if our server had to restart our processes would not be started again. That would be a bit awkward. 
Ubuntu has created [Upstart](http://upstart.ubuntu.com/) for these very problems. Upstart is a "event-based replacement for the /sbin/init daemon which handles starting of tasks and services during boot". All we need to do then is create a Upstart configuration file to load Bluepill.
Once Bluepill starts it will then load our application for us. Below is my Upstart configuration file. Remember to copy it into `/etc/init`. If you having issues getting Upstart working you can check the logs in `/var/log/upstart`. 

<script src="https://gist.github.com/3780526.js?file=bluepill_config.conf"></script>

Starting and stopping that is easy, `start bluepill_config.conf` or `stop bluepill_config.conf`

## Capistrano

At this point you probably thinking, nice work but how do I use it with Capistrano. At the answer is quite easily. You can stop and start both processes by stopping and starting Upstart. You can restart Unicorn via Bluepill so that you still get the hot reload power. I've added a simplified Capistrano script that should help you get started with integrating this into your Capistrano deploy.
<script src="https://gist.github.com/3780575.js?file=deploy.rb"></script>

## Sudoers.d

There is one more important ingredient for this to work properly. When running the above commands via Capistrano, a password is required. This 
