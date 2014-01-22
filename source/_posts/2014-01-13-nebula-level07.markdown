---
layout: post
title: "Exploit Exercises - Nebula Level 07"
date: 2014-01-13 15:49:10 +0800
comments: true
categories: Security, Exercises, Nebula
---

### About
The *flag07* user was writing their very first perl program that allowed them to ping hosts to see if they were reachable from the web server.

To do this level, log in as the *level07* account with the password *level07*. Files for this level can be found in /home/flag07.

### Source Code

``` perl
#!/usr/bin/perl

use CGI qw{param};

print "Content-type: text/html\n\n";

sub ping {
  $host = $_[0];

  print("<html><head><title>Ping results</title></head><body><pre>");

  @output = `ping -c 3 $host 2>&1`;
  foreach $line (@output) { print "$line"; } 

  print("</pre></body></html>");
  
}

# check if Host set. if not, display normal page, etc

ping(param("Host"));
```

<!-- more -->

### Solution

There is a *thttpd* server started with the configuration file */home/flag07/thttpd.conf*, in which there is only one page named *index.cgi* with the above code. This perl script mainly executes a `ping` command with the user specified argument `Host`, and prints the output onto the webpage.

First access the web page through a browser with the following url:

	http://192.168.11.118:7007/index.cgi?Host=127.0.0.1

And the page looks fine. As the perl script does not check the user input, so we can leaverage it to execute any commands with user *flag07* (as the http server starts with user *flag07's* privileage, which is configured in *thttpd.conf*). Similar to previous exercises, I would like to pass in the argument Host="127.0.0.1;/bin/getflag", so I accessed the following url directly 

	http://192.168.11.118:7007/index.cgi?Host=127.0.0.1%3B%2Fbin%2Fgetflag

And it hacks!
