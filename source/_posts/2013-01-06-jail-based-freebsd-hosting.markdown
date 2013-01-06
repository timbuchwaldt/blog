---
layout: post
title: "Jail-Based FreeBSD Hosting"
date: 2013-01-06 13:03
comments: true
categories: 
---

While switching my E-Mail hosting from my own server to Google Apps for Business, I noticed having a spare vServer that will then be completely bored. As I played with FreeBSD a lot during the holidays, I figured I could use the server for FreeBSD-based hosting.

And of course - talking about hosting on FreeBSD one has to talk about using jails. And when having only one public IP(-v4)-Address also about NATting the servers. I tried the "expand-127.0.0.1-approach" - which sucked. New idea was then just to use epair-devices that go to a virtual bridge in the host system. The host-systems runs PF (which I totally love) to NAT the virtual boxes to my public address.

To give you a little insight, here are the interesting parts of the config:

``` bash /etc/rc.conf
gateway_enable="YES"
ezjail_enable="YES"
pf_enable="YES"

cloned_interfaces="bridge0 epair0 epair1"
ifconfig_bridge0="addm re0 up"
ifconfig_epair0a="172.16.0.1"
ifconfig_epair0b="172.16.0.2"
ifconfig_epair1a="172.16.1.1"
ifconfig_epair1b="172.16.1.2"
``` 

``` bash /etc/pf.conf
jail_ips = "{172.16.0.1,172.16.0.2}"
nat on $ext_if proto {tcp udp icmp} from $jail_ips to any -> ($ext_if)
rdr on $ext_if proto tcp from any to any port {80,443} -> 172.16.0.2
pass in on $ext_if inet proto tcp from any to 172.16.0.2 port {80, 443}
``` 

```  bash /usr/local/etc/ezjail/www01
export jail_www01_exec_prestart0="ifconfig bridge0 addm epair0a"
export jail_www01_exec_poststop0="ifconfig bridge0 deletem epair0a"
``` 

``` bash /usr/local/etc/rc.d/ezjail.sh
# REQUIRE: LOGIN cleanvar sshd networking
``` 

The rc.conf just starts the neccessary services and creates the shared networking infrastructure as well as the jails epairs. The pf.conf just declares a variable for all jail IPs and adds NAT to external services as well as a portforwarding to the first jail.
The ezjail config creates pre and post start scripts, that add the epairs to the brige and sets up their IPs. I added the "networking" requirement to the ezjails rc-script to wait for all interfaces to get booted, I had some problems with the bridge not being created before jail-boot.

The system comes up cleanly after boots now and works just fine! Time to install some services to this monster ;D 