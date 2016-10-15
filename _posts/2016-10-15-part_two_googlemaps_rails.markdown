---
layout: post
title:  Easy Rails Datavisualization - GoogleMaps JavaScript API (Part 2 of 3)
date:   2016-10-15 20:49:05 -0400
---

Welcome to Part 2 of 3 in my series Easy Rails Datavisualization.  In [Part 1](http://agdavid.github.io/2016/10/15/part_one_googlemaps_rails/), we created the Rails API endpoint (coordinates#index) which, when called, delivers our coordinates data as JSON.  

Now, things get fun! 

Here in Part 2, we will use AJAX to retreive that information on the front-end so that we can use it together with the GoogleMaps JavaScript API.


**Part II: Using AJAX to Retrieve the Mapping Data in the Front-End**

*What is AJAX?*

AJAX stands for Asynchronous JavaScript and XML (phew!).  

It is a technique used in web applications to retrieve content from a server and display it without having to refresh the entire page.

Ever wonder how those comments appear at the bottom of a page when you hit "Show Comments" or how more shopping items appear when you scroll to the bottom of an Online Store page? AJAX!

*The jQuery.get() Method*

We all love jQuery, our fast and concise JavaScript library with the motto "[w]rite less, do more".  The [jQuery.get()](http://api.jquery.com/jquery.get/) method is a shorthand for an AJAX request to load data from a server using a HTTP GET request - i.e., it is our shortcut to call our API endpoint.

Let's start our JavaScript function in which we will retrieve the coordinates data and work with it.  In your app/assets/javascripts folder create a new file.  I called mine "map.js".

First, we include the jQuery.get() method to call the API endpoint:

```
//app/assets/javascripts/map.js

function initMap() {
  ...
  $.get("/coordinates.json", createCoordinates);
};
``` 

We just included the URL we would like to access and a callback function I called "createCoordinates".  The callback function is our helper because it accepts the returned JSON data so that we can work with it.

Second, create the callback function "createCoordinates" which will iterate over the array of JSON objects to create a JavaScript object for each coordinate:

```
//app/assets/javascripts/map.js

function initMap() {

  function createCoordinates(data) {
    for(var i = 0; i < data.length; i++) {
      var id = data[i].id;
      var lat = parseFloat(data[i].lat);
      var lng = parseFloat(data[i].lng);
      
      var coordinate = new Coordinate(id, lat, lng);
      ...
    }
  };
  ...
};

```

We just iterated over the JSON array returned by our AJAX GET request.  For each run of the loop, we set three variables (id, lat, lng) to a value then inserted those into a constructer function called "Coordinate".  Why?  We create a new JavaScript object from our JSON data, set it to a variable named "coordinate" so that we can then work with it some more!

Third, create the constructor function "Coordinate" which allows you create a JavaScript model object:

```
//app/assets/javascripts/map.js

function initMap() {
    
    function Coordinate(id, lat, lng) {
        this.id = id
        this.lat = lat
        this.lng = lng
    };
  ...
};
```

Above we took the values from the "createCoordinates" function and then created a JavaScript model object that we can work with.

Let's recap what you just completed:
1. Used AJAX to retrieve an array of JSON data from a Rails API back-end
2. Iterated over that array using the "createCoordinates" function
3. Created, with each loop, a JavaScript model object using the "Coordinate" constructer function

Great work!  

**Coming Up: Part 3 - Creating a map with the GoogleMaps JavaScript API**

