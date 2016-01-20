---
layout: post
title: "Iterators Schmiterators"
date: 2014-06-25 15:32:49 -0400
comments: true
categories:
- Flatiron&nbsp;School
- Ruby
- array
---
##Choosing Between Each, Map, Collect, Select, or Detect

Sometimes it can be confusing for a ruby newbie to know what method to use when iterating over an array or hash. Most tutorials focus on the each method like a prized first child and then gloss over the others. That can leave a lot of newbies writing extra code trying to make the each method work when they could use a different iterator with more ease.

The easiest way to decide which iterator to use is to ask yourself a few questions about the block of code you are passing in your iteration.

{% img center /images/Iterator_Tree.jpg 750 750 'Iterator Tree' 'Iterator Tree' %}

The first question is, "Are you 'puts'ing elements?" For that, you would want to use 'each' so that you can simply puts without altering the original array.

```ruby
array = [1,2,3,4]
array.each { |n| puts "The number is #{n}."}
  #=> The number is 1.
  #   The number is 2.
  #   The number is 3.
  #   The number is 4.
```
If you aren't 'puts'ing elements, you need to ask, "Do you want to pass each element through the block and then return a new array or hash?" The key words here are 'return a new array or hash'. The 'each' method won't do that. It will just return the old array or hash no matter what you do with the elements in the block. If you know you want to create a new array or hash, you have to then ask, "Are you only evaluating truthiness?" If you are, 'select' or 'detect' are the way to go. 
```ruby
array.select { |n| n.odd? }
  #=> [1,3]
array.detect { |n| n.odd? }
  #=> 1
```
'Select' will return an array of all the elements that evaluate to true, while 'detect' will return only the first element that evaluates to true.

If you want to create a new array or hash AND you are doing something other than evaluating truthiness, you'll want to use 'map' or 'collect'. These two do the same thing. The only differences being: you have four less characters to type with 'map', but 'collect' describes a bit better what you are actually doing. Choose as you will.
```ruby
array.collect { |n| n*10 }
  #=> [10, 20, 30, 40] 
```
One piece of advice: beware of sandwich coding. This happens if you are trying to use 'each' where you want to use 'map'/'collect'.
```ruby
new_array = []
array.each { |n| new_array << n*10 }
new_array 
  #=> [10, 20, 30, 40]
```
You see how I had to add two extra lines to set up the variable as an empty array and then call the variable after the code was passed? Those lines are the bread to the 'each' meat. Avoid this. Even if you love sandwiches. I love sandwiches.