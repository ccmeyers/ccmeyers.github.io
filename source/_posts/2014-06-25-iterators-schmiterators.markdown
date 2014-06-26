---
layout: post
title: "Iterators Schmiterators"
date: 2014-06-25 15:32:49 -0400
comments: true
categories: "Flatiron&nbsp;School"
---
##Choosing Between Each, Map, Collect, Select, or Detect

Sometimes it can be confusing for a ruby newbie to know what method to use when iterating over an array or hash. Most tutorials focus on the each method like a prized first child and then gloss over the others. That can leave a lot of newbies writing extra code trying to make the each method work when they could use a different iterator with more ease.

The easiest way to decide which iterator to use is to look at the block of code you are passing.

{% img left /images/Iterator_Tree.jpg 350 350 'Iterator Tree' 'Iterator Tree' %}