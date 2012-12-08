---
layout: post
title: "Rails ActiveRecord readonly-mode"
date: 2012-12-08 12:07
comments: true
categories: 
---
I thought about PostgreSQL failover yesterday, and there is one downside to that: When using hot-standby readslaves, one has to promote on of the slaves to master, and make sure the ex-master never comes up again, thinking he is master.
There are 2 ways of doing that: Choose a master by script, or let the operator choose one.
In my eyes, doing it by hand keeps the risk lower, that is introduced by letting a machine elect the new master (read about that at [github](https://github.com/blog/1261-github-availability-this-week)).

When using readslaves, you most propably want to use them in hot-standby mode and let your applications read from it. For Rails there is [Seamless Database Pool](https://github.com/bdurand/seamless_database_pool). This way, you get several benefits: 

- Improved read-performance (which should make up most of your apps traffic)
- Increased failure resilience (when one of your read-slaves fails, SDP takes it out of the pool, and retries later on that slave)
- The theoretical chance of being able to run the app in read-only mode

To be able to switch Rails to read-only while running, you have to patch ActiveRecords _readonly?_ base method:

``` ruby   
module ActiveRecord
    class Base
        def readonly?
            ($redis.get('ar:readonly') == "true") || false
        end
    end
end
```

This way, every time you want to somehow change a record, ActiveRecord calls _readonly?_ and therefore tries to get the key _ar:readonly_ from redis. You can change this key to true or false, and by that change the read-only behaviour.

You have to do some changes to ApplicationController:

``` ruby   
class ApplicationController < ActionController::Base
  protect_from_forgery
  rescue_from ActiveRecord::ReadOnlyRecord, :with => :readonly_error

  private

  def readonly_error(error)
    @error = error
    render :template => "/error/readonly.html.erb", :status => 404
  end
end
```
This rescues the AR ReadOnlyRecord error, and renders the readonly.html.erb file in _app/views/error_
You can change this file and by that render some page for the client, that tells what is going on currently.

To deploy this, you should make sure that the redis server is really close to the app (yep, every write-request triggers one redis read-request). You could use a local redis instance as a slave to your redis cluster, or build a completely own redis network, just for management purposes. Keeping a local slave to all the redis data might be problematic when having a high load on the main redis systems. For that purpose, use distinct servers with a small keyspace and load.

I like the idea of read-only redis on app servers, used exclusively for management purposes. By bringing the management datastore closer, a failure on your main datastore will not lead to undesired effects.
