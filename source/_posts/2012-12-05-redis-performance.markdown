---
layout: post
title: "About redis performance"
date: 2012-12-05 18:36
comments: true
categories: 
---

##Why redis?

Redis is a high performance key-value store with capabilites going much further then e.g. memcached. It features several higher-level datastructures that can be used for storing more than simple key-value pairs. Additionally it has the capability to save data to disk on regular intervals (RDF) or by a append-only log (AOF). With that features it could easily replace a database, while being potentially higher in performance terms.

I use redis for all kinds of experiments and in some production apps, where I need high performance while keeping a small footprint, but also want great flexibility in regards to my _database schema_..

##Benchmarking redis

The _redis-benchmark_ utility is surely the first way to test out your instance of redis. On my iMac it shows some 15k reads/sec, while watching movies and writing this article. But that isn't the point in this article, as I want to point out a big difference between querying keys sequentially and using some more avdanced techniques.

I wrote a small, single threaded [ruby script](https://gist.github.com/4215376 "redis performance benchmark") that just gets some keys via redis and hiredis.
It gets 12 keys in a row, repeating the action 10 000 times.

The comparison is made between just using _redis.get_ 12 times in a row, using the _multi_ command to just send one query over the wire and using a _redis script_ that is run via the embedded Lua interpreter. Of course for this querying scenario the _mget_ command would be the one to go with, but this example is just for comparing the various speeds. 

Running the script I get the following output:

	Simple:		799 req/s
	Multi:		3119 req/s
	Script:		6441 req/s

__Simple__ is just getting the keys in a sequence, __Multi__ via a _redis.multi do_ block and __Script__ via a preloaded Lua-Script.

As you see the _multi_-performance is some four times better than just querying it sequentially, and _script_ is another two times better. The Lua script is compiled once and then accessible via it's SHA1-Hash. After that it can access the redis-internal via very fast methods, in opposite to querying everything over the TCP-Interface.
The overhead when the Lua-script runs is very low as I saw during the experiments, and the difference between querying 6 and 12 keys 10 000 times is marginal when using the script:

	Simple:		1676 req/s
	Multi:		4518 req/s
	Script:		6704 req/s

As you see, the Simple and Multi-Method improve quite noticeable - in opposite to the Script.

My experience shows a great improvement in performance when doing operations on multiple keys, that might be dependent on each other. To overcome performance bottlenecks during such queries, and when your requests are very similar, I would recommend anybody using redis to use the 2.6 version, that has scripting support.
I wrote an application lately that had to get the same key from 2 different hashes, which resulted in two queries. Because the app is written in Ruby, I just let them run sequentially. The performance impact for now is minimal, but in the future this might turn out to be a dumb idea. 

Another possibility is __pipelining__ the actions. You loose the guarantee of _multi_ that it's an atomical call, but gain quite a bit of speed:

	Simple:		1412 req/s
	Multi:		3128 req/s
	Pipeline:	4813 req/s
	Script:		6822 req/s

## JSON and MessagePack

The redis scripting API has some nice capabilities built in, two of which are support for [JSON](http://www.json.org) and [MessagePack](http://msgpack.org). With those two libraries, you can build parts of the application right inside of redis. For example you could prepare the JSON that your app will reply to some sort of client directly within redis or collect together big chunks fo data and return them using MessagePack, which is really small can be easyly (and __fast__) parsed in a variety of languages

## Summary

- Use redis... where it fits in. Don't build your bank on top of redis ;D 
- Benchmark your stuff
- Keep watching the changes. Redis improves every release!