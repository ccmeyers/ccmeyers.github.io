---
layout: post
title: "Boot it Up, Baby"
date: 2014-09-01 18:00:52 -0400
comments: true
categories: "Flatiron&nbsp;School"
---
##How to Start a Simple Local Server

Sometimes when building a simple website, it's tempting to check your changes by viewing your file(s) in a browser. But have you ever gotten stuck when some of your more complicated code isn't working even though you KNOW IT SHOULD?!?!?! 

Maybe you're getting errors in the console like "XMLHttpRequest cannot load file:... Cross origin requests are only supported for HTTP"?

Well, the problem might not be you. It might just be your not running a local server. This happened to me while working with AngularJS's custom directives and to a friend while working with popcorn.js. 

"But I'm not using Rails or Sinatra," you say? 

"But I can't type 'shotgun' or 'rails s' into my terminal," you declare?

Want to know a trick? 

```bash
brew install python
```
I know, it feels like you're cheating on Ruby. But, she'll never know, I swear.

```bash
python -m SimpleHTTPServer 8000
```
Then, just open up http://localhost:8000/ in your browser, and you'll be on your way!

If your code still doesn't work while on this local server...it might just be your code. So...there's that.