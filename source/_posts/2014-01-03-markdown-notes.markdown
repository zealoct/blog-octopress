---
layout: post
title: "Markdown Notes"
date: 2014-01-03 14:16:17 +0800
comments: true
categories: 
---



Header
-----

# Header 1

Header 1
=====

## Header 2

Header 2
-----

### Header 3

#### Header 4

##### Header 5 


Normal Content


Source Code:

	# Header 1

	Header 1
	=====

	## Header 2

	Header 2
	-----

	### Header 3




<!--more-->

List
-----
- item 1
    - item 1.1
- item 2
- item 3


Source Code:

	- item 1
	    - item 1.1
	- item 2
	- item 3




Ordered list
-----
1. item 1
    1. item 1.a
2. item 2
3. item 3


Source Code:

	1. item 1
	    1. item 1.a
	2. item 2
	3. item 3





Code
-----

``` c
fbfd = open("/dev/fb0", O_RDWR);
if (fbfd <0 ) {
    perror("device open failed\n");
}
```

Source Code:

	``` c
	fbfd = open("/dev/fb0", O_RDWR);
	if (fbfd <0 ) {
	    perror("device open failed\n");
	}
	```


Pic & Hyper link
-----

![](/images/profile.jpg)

[Pic link](/images/profile.jpg)


Source Code:

	![](/images/profile.jpg)

	[Pic link](/images/profile.jpg)



Table
-----

Column1     | Column2      
----------  | ------------ 
foo         | foo
foo         | foo
foo         | foo
foo         | foo

</br>
Source Code:

	Column1     | Column2
	----------  | ------------ 
	foo         | foo
	foo         | foo
	foo         | foo
	foo         | foo

