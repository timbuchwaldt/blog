---
layout: post
title: "Using Varnish"
date: 2012-01-19 11:24
comments: true
categories: 
---
I needed a small VCL for caching assets in Varnish. I used some tutorials and documentation to come to this:
{% codeblock %}
sub vcl_recv{
    if (req.url ~ "(?i)\.(png|gif|jpeg|jpg|ico|swf|css|js|html|htm)(\?[a-z0-9]+)?$") {
     unset req.http.Cookie;
  }
  if(req.http.Cookie ~ "(PHPSESSID|other)") { 
                        # dont do anything, the user is logged in 
                } else { 
                        # dont care about any other cookies 
                        unset req.http.Cookie; 
                } 
}
sub vcl_fetch {
    unset beresp.http.server;
    unset beresp.http.X-Powered-By;
    if(req.url ~ "\.(png|gif|jpg|jpeg|ico)$"){
        set beresp.ttl = 2d;
        set beresp.keep = 2h;
        unset beresp.http.set-cookie;
    }
    if(req.url ~ "\.(css|js)$"){
        set beresp.ttl = 4d;
        set beresp.keep = 4h;
        unset beresp.http.set-cookie;
    }

}
{% endcodeblock %}
It basically unsets cookies for all static contents, so Varnish can cache it. In the vcl_fetch it unsets the server that served the request and the X-Powered-By header to hide the PHP-Version. Then it checks if the request was for an image or an CSS/JS File and writes new TTL-headers and sets the Varnish keep time to cache images for 2 hours, for CSS/JS 4 hours.
The vcl_recv also contains a rule to check for certain cookies to keep them and drop the ones not nessecary. With that rule we enable Varnish to cache content for anonymous users, altough we use Google Analytics / other analytics tools. (Varnish doesnt cache requests containing Cookies).
