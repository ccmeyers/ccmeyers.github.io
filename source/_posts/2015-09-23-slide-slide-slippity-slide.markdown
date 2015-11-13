---
layout: post
title: "Slide Slide Slippity Slide"
date: 2015-09-23 16:47:44 -0400
comments: true
categories: "JavaScript"
---

##How to Create a Slide Presentation in less than 200 Lines of JavaScript


Recently a client needed a website that could be used both as a presentation tool (like a Keynote or PowerPoint presentation) and as a stand-alone website to be sent to possible investors to scroll through.

There are a lot of great libraries out there that help you mimic the functionality of a Keynote presentation. I took a look at <a href="http://flowtime-js.marcolago.com/" target="_blank">flowtime</a>, <a href="http://lab.hakim.se/reveal-js/#/" target="_blank">reveal.js</a>, and <a href="http://impress.github.io/impress.js/#/bored" target="_blank">impress.js</a>. But, these libraries are big and seemed like overkill for what I needed to do. Also, I really wanted to see if I could build this on my own.

(If you want to jump ahead to the finished product, here's <a href="http://slide-presentation.divshot.io/" target="_blank">an example</a> of what I built along with <a href="https://github.com/ccmeyers/slide-presentation" target="_blank">the code on github</a>.)

First I made my panels. I broke up the site into sections (literally -- using `<section></section>` tags) and made each section `height = 100vh`. Actually, I had to make room for my fixed nav up top, so it was really `height = calc(100vh - 103)`. Now that I had things looking the way they should, I needed to make them act the way they should.

I soon realized that my biggest challenge was to designate which section was the current panel so we could find the previous and next panels. I found this <a href="http://stackoverflow.com/questions/22659903/jquery-scroll-to-next-prev-section-by-keyup-keydown" target="_blank">stackoverflow answer</a> that seemed like it could be a solution. It used the <a href="https://github.com/customd/jquery-visible" target="_blank">jquery-visible plugin</a> to tell if a section was in view and then found next and previous based on that. However, after the better part of a day going down that path, I realized it wouldn't work for my needs. Before giving up, I asked another engineer if he had any idea how I could make this work (shout out, Kevin!). He said, "Why don't you try using waypoints?" YES! Why didn't I think of that?

<a href="http://imakewebthings.com/waypoints/" target="_blank">Waypoints</a> is a great library that lets you trigger a function on scroll. I could make each section have its own waypoint where it gets the class 'currentPanel' as soon as it's almost to the top of the screen.

```javascript
  $('section').each(function(){
    var index = $(this).index('section'),
        currentSection = $('.section'+index),
        nextSection = currentSection.next(),
        prevSection = currentSection.prev();
        currentPanelClass = 'current-panel';
    var waypointsDown = new Waypoint({
      element: currentSection,
      handler: function(direction) {
        if (direction === 'down') {
          currentSection.addClass(currentPanelClass);
          if (prevSection.length > 0) {
            prevSection.removeClass(currentPanelClass);
          }
        }
      },
      offset: 200
    });
    var waypointsUp = new Waypoint({
      element: currentSection,
      handler: function(direction) {
        if (direction === 'up') {
          currentSection.addClass(currentPanelClass);
          if (currentSection.length > 0) {
            nextSection.removeClass(currentPanelClass);
          }
        }
      },
      offset: 100
    });
  });
}
```

In that function, I'm looping through all the sections and assigning a waypoint to each based on its index. I set up my HTML so that each `<section>` had a class based on its position (i.e. the first section has a class of 'section0'). As you're scrolling down, a section will get the class 'currentPanel' when it's 200px from the top. On the way back up, a section would get that class 100px from the top. I'm also making sure that as the 'currentPanel' class is added to one section, it is removed from the section that just had it.

Now that I could find the current panel, I needed to designate next and previous panels and animate the scroll.

```javascript
scrollToElement: function(panel) {
  $('html, body').animate({
    scrollTop: panel.offset().top - 102
  }, 200);
},

findNext: function() {
  var that = this,
      currentPanel = $('.current-panel'),
      nextPanel = currentPanel.next('section');
  if (nextPanel.length > 0) {
    that.scrollToElement(nextPanel);
  }
},

findPrev: function() {
  var that = this,
      currentPanel = $('.current-panel'),
      prevPanel = currentPanel.prev('section');
  if (prevPanel.length > 0) {
    that.scrollToElement(prevPanel);
  }
}
```
Now I needed that scroll to happen in response to a presentation clicker. We had <a href="http://www.logitech.com/en-us/product/wireless-presenter-r400?crid=11" target="_blank">this one</a> in the office, so I connected it to my computer and console logged the <a href="https://css-tricks.com/snippets/javascript/javascript-keycodes/" target="_blank">keyCodes</a> to find which it simulated. It was using 33 and 34 (page up and page down). I also added a few more keyCodes in case the client forgot his clicker and needed to use the keyboard to move through the site.

```javascript
slider: function(e) {
  var that = this;
  $(document).on('keydown', function(e){
    var $currentPanel = $('.currentPanel');
    if (e.keyCode === 40 || e.keyCode === 32 || e.keyCode === 13  || e.keyCode === 34 || e.keyCode === 39) {
      e.preventDefault();
      that.findNext();
    } else if (e.keyCode === 38 || e.keyCode === 33 || e.keyCode === 37) {
      e.preventDefault();
      that.findPrev();
    }
  });
}
```

Thanks to my 'getCurrentPanel' function, I knew that the panel with the class 'currentPanel' was my current panel, and could fire my 'findNext' or 'findPrev' functions based on whether the keyCodes called for up or down movement. Yay!

On to what I call 'fragmented' panels. By this I mean sections that call for a halt in the movement up and down and instead move through the content inside the panel itself.

```javascript
fragmentedPanel: function(movement) {
  var that = this,
      currentFragmentedPanel = $('.current-panel.fragmented'),
      currentFragmentedParts = currentFragmentedPanel.find('.fragmented-part.active');
  currentFragmentedParts.each(function(){
    var fragmentIndex = $(this).index('.current-panel .fragmented-part'),
        nextFragment = $('.current-panel .fragmented-part.part'+fragmentIndex).next(),
        prevFragment = $('.current-panel .fragmented-part.part'+fragmentIndex).prev(),
        checkForLast = nextFragment.next(),
        checkForFirst = fragmentIndex - 2;
    if (movement === 'down' && nextFragment.length > 0) {
      if (checkForLast.length > 0) {
        currentFragmentedParts.removeClass('active');
        nextFragment.addClass('active');
        currentFragmentedPanel.removeClass('first-fragment');
        currentFragmentedPanel.removeClass('last-fragment');
      } else {
        currentFragmentedParts.removeClass('active');
        nextFragment.addClass('active');
        currentFragmentedPanel.removeClass('first-fragment');
        currentFragmentedPanel.addClass('last-fragment');
      }
    } else if (movement === 'up' && prevFragment.length > 0) {
      if (checkForFirst >= 0) {
        currentFragmentedParts.removeClass('active');
        prevFragment.addClass('active');
        currentFragmentedPanel.removeClass('first-fragment').removeClass('last-fragment');
      } else {
        currentFragmentedParts.removeClass('active');
        prevFragment.addClass('active');
        currentFragmentedPanel.removeClass('last-fragment').addClass('first-fragment');
      }
    }
  });
}
```

For this code to work, you need to set up your HTML with certain classes. The 'section' will need to get the classes 'fragmented' and 'first-fragment'. The parts within that fragmented section need to get two classes: 'fragmented-part' and part+index (i.e. 'part0'). Here's an example of how to set up the HTML:

```html
<section class="section0">
  <!-- this is a slide without fragments -->
</section>
<section class="section1 fragmented first-fragment">
  <!-- this is a slide with fragments -->
  <div class="fragmented-part part0 active"></div>
  <div class="fragmented-part part1"></div>
  <div class="fragmented-part part2"></div>
</section>
```

A big thing in the fragmentedPanel function is to know when you're reaching the end of the fragmented-parts so you can move on to the next slide. That's why I have the 'checkForLast' and 'checkForFirst' variables. With every movement, I'm checking to see if an element exists in the next slot.

Great, now we need to update our slider function to halt movement if we see a 'fragmented' section.

```javascript
slider: function(e) {
  var that = this;
  $(document).on('keydown', function(e){
    var $currentPanel = $('.currentPanel');
    if (e.keyCode === 40 || e.keyCode === 32 || e.keyCode === 13  || e.keyCode === 34 || e.keyCode === 39) {
      e.preventDefault();
      if (currentPanel.hasClass('fragmented') && !(currentPanel.hasClass('last-fragment'))) {
        that.fragmentedPanel('down');
      } else {
        that.findNext();
      }
    } else if (e.keyCode === 38 || e.keyCode === 33 || e.keyCode === 37) {
      e.preventDefault();
      if ( currentPanel.hasClass('fragmented') && !(currentPanel.hasClass('first-fragment')) ) {
        that.fragmentedPanel('up');
      } else {
        that.findPrev();
      }
    }
  });
}
```

That's pretty much it. I hope this makes some kind of sense to you. You can take a look at the <a href="https://github.com/ccmeyers/slide-presentation" target="_blank">full code here</a> and a <a href="http://slide-presentation.divshot.io/" target="_blank">demo here</a>.

And in honor of this blog post's title. I leave you with a little Coolio circa 1994.

<iframe width="420" height="315" src="https://www.youtube.com/embed/cbhkuu4e0iw" frameborder="0" allowfullscreen></iframe>
