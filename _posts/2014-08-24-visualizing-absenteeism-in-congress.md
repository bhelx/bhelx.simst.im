---
layout: post
title: Visualizing Absenteeism in Congress
category: articles
tags: [congress, python]
---

Since I left New Orleans, I've continued to watch [Cedric Richmond's](https://www.govtrack.us/congress/members/cedric_richmond/412432) absenteeism climb. I started wondering what the data on attendance looked like in the House and the Senate.

<img alt="cedric on vacation" src="/public/images/cedric_truancy.jpg">

### Collecting Data

I let my computer run last night pulling down all the vote data I could capture from 1989 to present day. I loaded it up into a large pandas DataFrame this morning (about 2 Gigs) and started splitting some data. The first thing I thought about was breaking the data up into states. I decided I would first aggregate the total votes for each state. Then I would aggregate the total **Not Present** votes for each state. Then I would divide those two state-indexed vectors to get the percentage of missed votes per state. That code looks something like this:

{% highlight python %}
total_votes = senate_votes.state.value_counts()
not_voting = senate_votes[senate_votes.result == 'Not Voting'].state.value_counts()
senate_missed_percentages = not_voting / total_votes
{% endhighlight %} 

The same calculation was performed on House data. The **result** field may have a few different values depending on the vote and on whether it's the Senate or the House. The 4 primary values for a typical vote are:

* **Present**
* **Not Voting**
* **Aye/Yea**
* **Nay/No**

A vote of **Present** means the legislator is present for the vote, but is choosing to abstain from making a choice. **Not Voting** means the legislator is not present at all. **Aye** and **Nay** are self-explanatory. **Aye/Yea** and **Nay/No** are [different conventions](https://www.govtrack.us/blog/2009/11/18/aye-versus-yea-whats-the-difference/) used in different procedures but mean the same thing.

### Visualizatons

I knew I wanted to make an interactive [choropleth map](https://en.wikipedia.org/wiki/Choropleth_map) to show the data. I had hoped on modifying [Mike Bostock's d3 example](http://bl.ocks.org/mbostock/4060606) but I was not patient enough to learn d3 and couldn't get it to work. Instead I just used Leaflet and removed the map tiling.

Unfortunately, I could not figure out how to imitate Bostock's nice projection (with Hawaii and Alaska in the frame) so you will need to scroll or zoom a bit to see them. The maps support pinching and dragging. The darker the value here the more missed votes. Hover over each state to see the true percentage.

### Senate

<link rel="stylesheet" href="/public/css/leaflet.css" />
<style>
  #map-senate {
    width: 650px;
    height: 500px;
    margin-bottom: 30px;
  }
  #map-house {
    width: 650px;
    height: 500px;
    margin-bottom: 30px;
  }
  .info {
    padding: 6px 8px;
    font: 14px/16px Arial, Helvetica, sans-serif;
    background: white;
    background: rgba(255,255,255,0.8);
    box-shadow: 0 0 15px rgba(0,0,0,0.2);
    border-radius: 5px;
  }
  .info h4 {
    padding-top: 0px;
    margin: 0 0 5px;
    color: #777;
  }
  .legend {
    text-align: left;
    line-height: 18px;
    color: #555;
  }
  .legend i {
    width: 18px;
    height: 18px;
    float: left;
    margin-right: 8px;
    opacity: 0.7;
  }
</style>

<div id="map-senate"></div>

<script src="/public/js/leaflet.js"></script>
<script type="text/javascript" src="/public/js/us-states-senate-truancy.js"></script>
<script type="text/javascript">

(function() {
  var map = L.map('map-senate', {
    maxZoom: 6.0,
    minZoom: 3.0,
    zoomControl: false
  }).setView([39.8, -96], 3.8);

  // control that shows state info on hover
  var info = L.control();

  info.onAdd = function (map) {
    this._div = L.DomUtil.create('div', 'info');
    this.update();
    return this._div;
  };

  info.update = function (props) {
    this._div.innerHTML = '<h4>Senate Absenteeism (1989 - Present)</h4>' +  (props ?
        '<strong><i>' + props.name + '</i></strong> - ' + props.truancy.toFixed(1) + '% votes missed'
        : 'Hover over a state');
  };

  info.addTo(map);

  // get color depending on population density value
  function getColor(d) {
    return d > 7.0 ? 'rgb(8,48,107)' :
      d > 6.0  ? 'rgb(8,81,156)' :
      d > 5.0  ? 'rgb(33,113,181)' :
      d > 4.0  ? 'rgb(66,146,198)' :
      d > 3.0   ? 'rgb(107,174,214)' :
      d > 2.0   ? 'rgb(158,202,225)' :
      d > 1.0   ? 'rgb(198,219,239)' :
      'rgb(222,235,247)';
  }


  function style(feature) {
    return {
      weight: 2,
      opacity: 1,
      color: 'white',
      dashArray: '3',
      fillOpacity: 0.7,
      fillColor: getColor(feature.properties.truancy)
    };
  }

  function highlightFeature(e) {
    var layer = e.target;

    layer.setStyle({
      weight: 5,
      color: '#666',
      dashArray: '',
      fillOpacity: 0.7
    });

    if (!L.Browser.ie && !L.Browser.opera) {
      layer.bringToFront();
    }

    info.update(layer.feature.properties);
  }

  var geojson;

  function resetHighlight(e) {
    geojson.resetStyle(e.target);
    info.update();
  }

  function zoomToFeature(e) {
    map.fitBounds(e.target.getBounds());
  }

  function onEachFeature(feature, layer) {
    layer.on({
      mouseover: highlightFeature,
      mouseout: resetHighlight
    });
  }

  geojson = L.geoJson(statesData, {
    style: style,
    onEachFeature: onEachFeature
  }).addTo(map);

  map.attributionControl.addAttribution('Vote data: <a href="https://www.govtrack.us/">Govtrack</a>');

  var legend = L.control({position: 'bottomright'});

  legend.onAdd = function (map) {

    var div = L.DomUtil.create('div', 'info legend'),
        grades = [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0],
        labels = [],
        from, to;

    for (var i = 0; i < grades.length; i++) {
      from = grades[i];
      to = grades[i + 1];

      labels.push(
          '<i style="background:' + getColor(from + 1) + '"></i> ' +
          from + (to ? '&ndash;' + to : '+') + '%');
    }

    div.innerHTML = labels.join('<br>');
    return div;
  };

  legend.addTo(map);
})();

</script>

The highest 3 states for the Senate are:

1. MA (7.73%)
2. NC (6.76%)
3. AZ (6.26%)

The lowest 3 states are:

1. ME (0.36%)
2. WI (0.76%)
3. MI (0.91%)

### House

<div id="map-house"></div>

<script type="text/javascript" src="/public/js/us-states-house-truancy.js"></script>
<script type="text/javascript">

(function() {
  var map = L.map('map-house', {
    maxZoom: 6.0, 
    minZoom: 3.0,
    zoomControl: false
  }).setView([39.8, -96], 3.8);

  // control that shows state info on hover
  var info = L.control();

  info.onAdd = function (map) {
    this._div = L.DomUtil.create('div', 'info');
    this.update();
    return this._div;
  };

  info.update = function (props) {
    this._div.innerHTML = '<h4>House Absenteeism (1990 - Present)</h4>' +  (props ?
        '<strong><i>' + props.name + '</i></strong> - ' + props.truancy.toFixed(1) + '% votes missed'
        : 'Hover over a state');
  };

  info.addTo(map);

  // get color depending on population density value
  function getColor(d) {
    return d > 7.0 ? 'rgb(8,48,107)' :
      d > 6.0  ? 'rgb(8,81,156)' :
      d > 5.0  ? 'rgb(33,113,181)' :
      d > 4.0  ? 'rgb(66,146,198)' :
      d > 3.0   ? 'rgb(107,174,214)' :
      d > 2.0   ? 'rgb(158,202,225)' :
      d > 1.0   ? 'rgb(198,219,239)' :
      'rgb(222,235,247)';
  }

  function style(feature) {
    return {
      weight: 2,
      opacity: 1,
      color: 'white',
      dashArray: '3',
      fillOpacity: 0.7,
      fillColor: getColor(feature.properties.truancy)
    };
  }

  function highlightFeature(e) {
    var layer = e.target;

    layer.setStyle({
      weight: 5,
      color: '#666',
      dashArray: '',
      fillOpacity: 0.7
    });

    if (!L.Browser.ie && !L.Browser.opera) {
      layer.bringToFront();
    }

    info.update(layer.feature.properties);
  }

  var geojson;

  function resetHighlight(e) {
    geojson.resetStyle(e.target);
    info.update();
  }

  function zoomToFeature(e) {
    map.fitBounds(e.target.getBounds());
  }

  function onEachFeature(feature, layer) {
    layer.on({
      mouseover: highlightFeature,
      mouseout: resetHighlight
    });
  }

  geojson = L.geoJson(statesData, {
    style: style,
    onEachFeature: onEachFeature
  }).addTo(map);

  map.attributionControl.addAttribution('Vote data: <a href="https://www.govtrack.us/">Govtrack</a>');

  var legend = L.control({
    position: 'bottomright'
  });

  legend.onAdd = function (map) {

    var div = L.DomUtil.create('div', 'info legend'),
        grades = [0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0],
        labels = [],
        from, to;

    for (var i = 0; i < grades.length; i++) {
      from = grades[i];
      to = grades[i + 1];

      labels.push(
          '<i style="background:' + getColor(from + 1) + '"></i> ' +
          from + (to ? '&ndash;' + to : '+') + '%');
    }

    div.innerHTML = labels.join('<br>');
    return div;
  };

  legend.addTo(map);
})();

</script>

The highest 3 states for the House are:

1. WY (12.31%)
2. AK (12.10%)
3. LA (6.74%)

The lowest 3 states are:

1. DE (1.39%)
2. ME (1.93%)
3. NE (2.01%)

## Thoughts

While this was fun and I plan to keep exploring the data, the results don't necessarily tell a story on their own. It's clear that some states' legislators are not showing up to vote more than others, but it's not clear what they are doing instead or whether or not they have a choice (perhaps committee duties interfering with votes?). I have to also consider that while I see missing votes as an unambiguously bad thing, many people in this country see disengaging in national politics as a point of pride. 
