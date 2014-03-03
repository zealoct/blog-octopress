---
layout: post
title: "Using theme Greyshade"
date: 2014-02-27 14:11:32 +0800
comments: true
published: true
categories: 
---

Sometimes ago I came across a blog [samwize](http://samwize.com/), who uses a slightly
modified theme called [Greyshade](https://github.com/shashankmehta/greyshade), which I
think is very beautiful, so I applied that theme immediately. Saddly this theme does
not support chinese very well, so I made some customization myself.

<!-- more -->

### Font and Size
- add new font "Microsoft Yahei" for Chinese, add font "Consolas" for monospace
- enlarge the font size of an article
- enlarge the line height to make it comfortable to read Chinese
</br> may be it is not large enough for Chinese, but I think it is too large for English
- enlarge the font size of meta info of an article
- decrease the font size for archieve view


### Layout
- abandon fixed width layout
- change the way meta info is displayed
- display meta info in the article view
- add Next and Prev link in the article view
- change a sharing provider
- remove the shadow below the sharing


### Profile
- use a local image instead of that of Gravatar.com, too slow


I would soon post my modified theme on my Github, 
it is sad that I did not modify the style directly on the theme but in my *sass/* and *source/*, 
so I need to patch my modification and merge it into the Greyshade. I would do this later...

