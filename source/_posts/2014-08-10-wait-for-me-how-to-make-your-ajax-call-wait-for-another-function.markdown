---
layout: post
title: "Wait for Me! How to Make Your Ajax Call Wait for Another Function"
date: 2014-08-10 21:58:04 -0700
comments: true
categories:
- ajax
- javascript
---

Recently, while building an app that used a lot of ajax calls, we kept on running into the problem of an ajax call firing before the function above it finished. Given that the function above provided variables that were necessary in our ajax call, this broke everything.

My team and I scoured the internet (i.e. stack overflow) for an answer, but came up with nothing. We finally asked a Flatiron School TA, and she gave a us a great solution.

We were building an app that calculates the half-way point of a given route. Then, we used the latitude and longitude of that point to search the Yelp API. The ajax call helps us get our javascript variables to the ruby method that directly deals with the API. And, here's what it looks like:

```javascript
var check_done = "not done"

function getStopPoint(response, percentage) {
  var totalDist = response.routes[0].legs[0].distance.value,
      totalTime = response.routes[0].legs[0].duration.value,
      distance = (percentage/100) * totalDist,
      time = ((percentage/100) * totalTime/60).toFixed(2),
      polyline = new google.maps.Polyline({
        path: [],
        strokeColor: '#FF0000',
        strokeWeight: 3
      });
      var bounds = new google.maps.LatLngBounds();
      var steps = response.routes[0].legs[0].steps;
      for (j=0; j<steps.length; j++) {
        var nextSegment = steps[j].path;
        for (k=0; k<nextSegment.length; k++) {
          polyline.getPath().push(nextSegment[k]);
          bounds.extend(nextSegment[k]);
        }
      }
  stopPointLatLonObject = polyline.GetPointAtDistance(distance);
  placeMarker(stopPointLatLonObject);
  stopPointLat = stopPointLatLonObject["d"];
  stopPointLon = stopPointLatLonObject["e"];
  check_done = "done";
}

function check(){ 
  if (check_done === "done"){
    $.ajax({
       url:'/restaurants/yelp_search', 
       type: 'POST',
       data:(
         'lat=' + stopPointLat + '&' +
         'lon=' + stopPointLon + '&' +
         'type=' + $("#type").val() + '&' +
         'sort=' + $("#sort").val() + '&' +
         'mtd=' + $("#mtd").val()
       )
    });
  } else {
    setTimeout(check, 1000);
  }
}
```

The getStopPoint function goes through the polyline (we're dealing with the lovely Google Maps API here) and finds the stopping point. The most important part of the getStopPoint function is where we define our stopPointLat and stopPointLon variables. Those are the lat/lng coordinates we're going to send to the Yelp API via the ajax call. But, alas, alack! All is going to break if the ajax train leaves the station without its lat/lng coordinates!

How we went about fixing this was by using a check_done variable that is set to "not done" when it is defined. It only changes to "done" once stopPointLat and stopPointLon have their values. We use an if/else statement inside the check function to make sure that check_done === "done" before we send off our ajax call. Else, we setTimeout and run the check function all over again.

I hope that helps anyone out there who might have this problem!