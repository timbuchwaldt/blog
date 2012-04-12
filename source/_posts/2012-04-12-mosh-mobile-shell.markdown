---
layout: post
title: "mosh - mobile shell"
date: 2012-04-12 22:34
comments: true
categories: 
---
I stumbled across [mosh](http://mosh.mit.edu), a SSH replacement.

Mosh allows you to connect to your ssh-enabled server (after installing it to the server, too). It creates the connection for you, then starts its server-process (mosh-server). After closing the SSH-connection, it connects a local mosh-client to the remote server.
It's based on UDP, according to the website its ~3 times faster than SSH (mean performance of keystroke response time).

But the best thing about mosh is its reconnecting-abilities. When your computer loses its network connection, mosh waits for a route to the remote server to come up again, it then reconnects to your existing session, which is untouched. Even triggered events (like keystrokes) are sent to the server after reconnection.