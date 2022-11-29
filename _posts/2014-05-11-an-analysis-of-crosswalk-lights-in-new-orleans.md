---
layout: post
title: An Analysis of Crosswalk Lights in New Orleans
category: articles
tags: [new orleans, maps, GIS, visualization, data, crosswalks, safety]
image:
  feature: yield_to_pedestrians.jpg
---

I think one thing that noticeably separates other major cities from New Orleans are well thought-out and well maintained infrastructures for pedestrians. When you visit a city like San Francisco or Washington DC, there is hardly a crosswalk without a pedestrian light (even in residential zones). They usually do nice things like count downs and make clicking noises for the visually impaired.

I've been traveling back and forth lately between New Orleans and San Francisco and I've been doing a lot of walking in both cities. I have noticed not just infrastructure differences, but behavioral differences. In San Francisco I rarely see people jaywalking or vehicles failing to yield to pedestrians (your mileage may vary). Meanwhile, back in New Orleans, both of those things are so commonplace they are expected. I've even noticed my own behavior changes from city to city. I started to wonder, **is my behavior being affected by the infrastructure**?

I was sitting at the coffee shop yesterday thinking about this hypothesis. The first question that popped into my mind was **"What is the current state of our pedestrian infrastructure?"**. I only had anecdotal information from my 5 years of walking around the city so I tried googling around for some hard data. I could not find any, but [I did find this report](http://www.wdsu.com/news/local-news/new-orleans/faulty-crosswalk-signal-lights-cause-danger-in-cbd/23595812) which I highly suggest you view now to familiarize yourself with the problem and what the city is planning to do about it.

The statement that stuck out to me was this: **"About a dozen lights at pedestrian crosswalks are broken along Canal Street"**.

That dozen number seemed significantly low. I did a quick estimate in my head, I figured there are roughly 100 lights on canal and about 3 out of 4 of the lights I've encountered in the city are broken. Well, now I knew I was going to have to go count them myself. Since I am in between jobs and homeless, I have some idle time on my hands. So I hacked together an app, picked up some road beers, and started zig-zagging the streets like a confused tourist to collect the data myself.

### Collecting Data

I looked around for some mobile software that would help me collect the data but I couldn't find anything that wasn't cumbersome or too granular. Luckily I was able to design and hack together a database-backed mobile-web app in a few hours. It is ugly but gets the job done.

<img alt="crosswalk app demo" src="/public/images/crosswalk_app_demo_optimized.gif" style="height:35%;width:35%">

The way this works is pretty simple. I have this big blue marker that I use as my **cursor**. It starts off pinned to your physical location, but you can move it by pressing and dragging it. It's so huge and ugly because you need to be able to see it under your thumb and I didn't have time for a more elegant solution. You drag it to a location then hit the **Add Light** button. That presents you with 3 options **Working**, **Broken**, or **Unusable**. Once you choose one, it sends the information off to the server for storage then renders the marker on the map with the respective color (green, yellow, red).

### Criteria

I tried to keep the criteria as objective as possible but I admit some of it had to be subjective. Here are my criteria for the 3 categories:

#### - Unusable

I would mark a light **Unusable** [red] when it was completely broken or I thought there was no way to figure out whether I could cross or not. For example, if the light was completely blank at any point of its cycle, or it has a single state that does not show, it's **Unusable**. Also keep in mind that I am NOT counting lights that aren't even there (there are plenty).

#### - Broken

I would mark a light **Broken** [yellow] when it was basically working but it was unclear whether I could cross or not. This would include situations where both states are lit at the same time. You could figure out what you were supposed to do but only by waiting long enough and using deductive logic. **Broken** also includes very low visibility (almost imperceptible) and situations where the light was working but turned away from view.

#### - Working

I would mark a light **Working** [green] when it appeared to be operating normally. I was very forgiving with this category. The minimum criteria was that I just needed to be able to figure out whether I was supposed to cross or not. Most of the lights that made it into this category still had some problems that should be addressed. Many of them had partially broken lights or their protective grate was shattered. I didn't penalize them as long as I could figure it out.

I also decided that since I wouldn't be able to walk the entire town, I would just walk the Central Business District. The way I saw it, the CBD is one of the highest foot-traffic areas in town. It has a high concentration of businesses and employees, tourists, and city officials moving about on foot.

### Analysis

The end goal was to analyze the data to create a map and some statistics. First let's look at the stats through some [matplotlib](http://matplotlib.org/) plots. 

Here is the breakdown per status of the *171* lights I recorded:

<img alt="crosswalk lights status counts" src="/public/images/crosswalk_bar_chart.png">

So I counted **90** (65 + 25) broken overall (**59** on the parts of **Canal** I checked). And here is a pie chart of the percentages:

<img alt="crosswalk lights pie chart percentages" src="/public/images/crosswalk_pie_chart.png">

As you can see, **over half** of the lights are broken or unusable. Here is an interactive map of the raw data:

<style>
	#map-canvas {
        height: 500px;
        width: 100%;
        margin: 0px;
        padding: 0px
	}
</style>
<div id="map-canvas"></div>
<script src="https://maps.googleapis.com/maps/api/js?v=3.exp&sensor=false"></script>
<script>
	function initialize() {
	  var mapOptions = {
	    zoom: 12,
	    center: new google.maps.LatLng(29.953611866615528, -90.0817357447147)
	  }
	  var map = new google.maps.Map(document.getElementById('map-canvas'), mapOptions);

	  var crosswalkLayer = new google.maps.KmlLayer({
	    url: 'https://bhelx.simst.im/public/images/crosswalk_full_map.kml',
	    map: map
	  });
	}
	google.maps.event.addDomListener(window, 'load', initialize);
</script>

<br />

The blue polygon is the region that I walked looking for lights and the markers are colored according to the red-yellow-green color code established earlier.

### Conclusions and More Questions

I can't say whether or not this data actually helped support my original hypothesis. Unfortunately the questions I had in mind were governed by too many independent variables. Not that I thought it would, the data and the collection of the data were really just to elevate the discussion and illicit better questions, not provide answers.

At the end of the day though, I think I learned more while collecting the data than I did analyzing it. I spent about 3 minutes at each major intersection. While standing there I wasn't just observing the lights, I was observing the people. While waiting for the lights to change, I would watch people's reactions to broken or confusing lights. I am not exaggerating when I say that at a majority of the intersections I experienced at least one troubling situation.

At one intersection a young family misstepped into a street with a broken crosswalk light. The father pulled the little girl back from an oncoming car just in time. They had a nervous chuckle about it but the mother was mortified. At another intersection some bachelor party guys drunk from the casino got tired of waiting to see if the light would turn on or not (it wasn't going to because it was completely shut off) and just bolted across the intersection only to have a taxi cab slam on its brakes and honk loudly. The taxi driver had that look on his face, **"typical drunk bros ruining our city"**.

At nearly every intersection I experienced tourists looking worriedly confused and muttering to themselves and each other **"Should we just go?"**. Nearby a local more familiar with the situation just ignored the rules altogether and casually ran across as soon as there was a gap in the cars. 

<img alt="sign urging drivers to yield to pedestrians" src="/public/images/yield_to_pedestrians.jpg">

The drivers don't know how to handle it very well either. As many times as I have walked through it, I was surprised to notice that Poydras has no crosswalk lights. Instead they have these signs posted above the crosswalks urging drivers to yield to pedestrians. Unfortunately, the drivers rarely comply. I tested a few cars only to be honked at or aggressively **hurried** along. If drivers don't follow this rule, what's even the point of a crosswalk? Why don't we all just cross the street where we wish and try our best to avoid cars? Is this why everyone behaves as if the rules don't matter anyway?

And once again, I was kind of shocked at my own behavior too. When I come up to a crosswalk I just try to cross it if no cars are coming, even if the lights appear to be working (I just assume they are broken or it doesn't matter). I'm still unsure why I have no problem doing that here and not in San Francisco though. Is it the laws? Is it just because everyone else is doing it? Is it just an Every-Other-Coast vs South-Coast attitude? Or is it because the infrastructure is hopeless and we have just given up on it? I am still not sure.

### Resources

All the code and data for this experiment are up on [a public github repo](https://github.com/bhelx/crosswalk-light-analysis).


