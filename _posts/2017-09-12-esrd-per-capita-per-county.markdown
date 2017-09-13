---
title: ":moneybag: The Variable Cost of Kidney Failure"
layout: post
date: 2017-09-12 21:37
image: 
headerImage: false
tag:
- healthcare
- healthcare-costs
- data-exploration
- visualization
projects: false
description: Exploration of the cost of end-stage renal disease
category: blog
author: aimeebarciauskas
externalLink: false
---

At Nava, our work on the ACA and Medicare supports a mission to improve how government serves it's citizens through better healthcare systems.

If you heard anything about healthcare in the United States, you've likely heard about how expensive it is.

One of the likely culprits of high costs is variation. I wanted to explore just _how_ variable costs are, and discovered some [cost data for end-stage renal disease (ESRD)](https://data.cms.gov/Special-Programs-Initiatives-Medicare-Shared-Savin/2015-County-level-Fee-for-Service-FFS-Data-for-Sha/njws-h2xd), also known as [kidney failure](https://en.wikipedia.org/wiki/Chronic_kidney_disease#Severity-based_stages). Using this data, I explored per capita ESRD costss by county and the correlation to [HCC risk scores](https://www.securityhealth.org/provider-manual/shared-content/claims-processing-policies-and-procedures/risk-adjustment---hcc-coding).

Observations:

* The highest cost ($148,861) is **5 times** the lowest cost ($29,549).
* There is an obvious positive correlation between average HCC risk score and average cost.

## ESRD Per Capita Expenditures by County

<style>
  @import url(https://fonts.googleapis.com/css?family=Open+Sans+Condensed:300|Josefin+Slab|Arvo|Lato|Vollkorn|Abril+Fatface|Old+Standard+TT|Droid+Sans|Lobster|Inconsolata|Montserrat|Playfair+Display|Karla|Alegreya|Libre+Baskerville|Merriweather|Lora|Archivo+Narrow|Neuton|Signika|Questrial|Fjalla+One|Bitter|Varela+Round);
  .background {
    fill: none;
    pointer-events: all;
  }
  svg {
    margin: 0px auto;
    display: block;
  }
  .map-layer {
    fill: #fff;
    stroke: #aaa;
  }
  .effect-layer{
    pointer-events:none;
  }
  text {
    font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif;
    font-weight: 300;
  }
  .d3-tip strong {
    color: white;
  }
  text.big-text{
    font-size: 30px;
    font-weight: 400;
  }
  .effect-layer text, text.dummy-text{
    font-size: 12px;
  }
  .d3-tip {
    line-height: 1;
    font-weight: bold;
    padding: 12px;
    background: rgba(0, 0, 0, 0.8);
    color: #fff;
    border-radius: 2px;
  }
  /* Creates a small triangle extender for the tooltip */
  .d3-tip:after {
    box-sizing: border-box;
    display: inline;
    font-size: 10px;
    width: 100%;
    line-height: 1;
    color: rgba(0, 0, 0, 0.8);
    content: "\25BC";
    position: absolute;
    text-align: center;
  }
  /* Style northward tooltips differently */
  .d3-tip.n:after {
    margin: -1px 0 0 0;
    top: 100%;
    left: 0;
  }
  .axis path {
    stroke-width: 1px;
  }
  .tick text {
    font-size: x-small;
  }
</style>

<div id="svg"></div>

<div>
  <script src="//d3js.org/d3.v3.min.js" charset="utf-8"></script>
  <script src="//d3js.org/topojson.v1.min.js"></script>
  <script src="http://labratrevenge.com/d3-tip/javascripts/d3.tip.v0.6.3.js"></script>
  <script>
  var width = 680,
      height = 450,
      marginBottom = 50,
      marginRight = 30,
      centered,
      ffsData;

  var color = d3.scale.linear()
    .domain([29549.29, 148861.78])
    .range(['#fff', '#409A99']);

  var projection = d3.geo.mercator()
    .scale(width)
    .center([-97, 37])
    .translate([width / 2, height / 2]);

  var path = d3.geo.path()
    .projection(projection);

  var svg = d3.select("#svg").append("svg")
    .attr("width", width+marginRight)
    .attr("height", height*2+marginBottom);

  svg.append('rect')
    .attr('class', 'background')
    .attr('width', width)
    .attr('height', height)
    .on('click', clicked);

  var g = svg.append('g');

  var effectLayer = g.append('g')
    .classed('effect-layer', true);

  var mapLayer = g.append('g')
    .classed('map-layer', true);

  var dummyText = g.append('text')
    .classed('dummy-text', true)
    .attr('x', 10)
    .attr('y', 30)
    .style('opacity', 0);
  var bigText = g.append('text')
    .classed('big-text', true)
    .attr('x', 20)
    .attr('y', 45);

  function nameLength(d){
    var n = nameFn(d);
    return n ? n.length : 0;
  }

  function nameFn(d){
    return d && d.properties ? d.properties.NOMBRE_DPT : null;
  }

  function fillFn(d) {
    if (isNaN(d.per_capita_exp_esrd)) {
      return '#eee';
    }
    return color(d.per_capita_exp_esrd);
  }

  function clicked(d) {
    var x, y, k;

    if (d && centered !== d) {
      var centroid = path.centroid(d);
      x = centroid[0];
      y = centroid[1];
      k = 4;
      centered = d;
    } else {
      x = width / 2;
      y = height / 2;
      k = 1;
      centered = null;
    }

    mapLayer.selectAll('path')
      .style('fill', function(d){return centered && d===centered ? '#D5708B' : fillFn(d);});

    g.transition()
      .duration(750)
      .attr('transform', 'translate(' + width / 2 + ',' + height / 2 + ')scale(' + k + ')translate(' + -x + ',' + -y + ')');
  }

  var tip = d3.tip()
    .attr('class', 'd3-tip')
    .offset([-10, 0])
    .html(function(d) {
      var expenditure = isNaN(d.per_capita_exp_esrd) ? d.per_capita_exp_esrd : "$" + d3.format(",.2f")(d.per_capita_exp_esrd);
      if (d.county_name && d.state_name) {
        return "<text><strong>" + d.county_name + ", " + d.state_name + ":</strong> " + expenditure + "</text>";
      } else {
        return "<text>" + expenditure + ", " + d.avg_risk_score_esrd + "</text>";
      }
    });

  svg.call(tip);

  function mouseover(d){
    if (!isNaN(d.per_capita_exp_esrd)) {
      d3.select(this).style('fill', 'orange');
      tip.show(d);
    }
  }

  function mouseout(d){
    mapLayer.selectAll('path')
      .style('fill', '#ffffff');
    d3.selectAll('circle')
      .style('fill', 'black');
    mapLayer.selectAll('path')
      .style('fill', function(d){
        return fillFn(d);
      });
    effectLayer.selectAll('text').transition()
      .style('opacity', 0)
      .remove();
    bigText.text('');
    tip.hide(d);
  };

 d3.json("/assets/data/ffs_keyed_by_state_county.json", function(error, data) {
    ffsData = data;
    d3.json("/assets/data/us_counties_2010.json", function(error, mapData) {
      if (error) return console.error(error);
      var features = mapData.features;
      features.forEach(function(d) {
        var state_and_county_id = d.properties.STATE + '-' + d.properties.NAME;
        var county_data = ffsData[state_and_county_id];
        if (county_data != undefined && county_data.per_capita_exp_esrd != 'nan') {
          d.county_name = county_data.county_name;
          d.state_name = county_data.state_name;
          d.per_capita_exp_esrd = county_data.per_capita_exp_esrd;
        } else {
          d.per_capita_exp_esrd = 'not available';
        }           
      });       

      mapLayer.selectAll('path')
          .data(features)
        .enter().append('path')
          .attr('d', path)
          .attr('vector-effect', 'non-scaling-stroke')
          .style('fill', fillFn)
          .on('mouseover', mouseover)
          .on('mouseout', mouseout)
          .on('click', clicked);

      var main = svg.append('g')
        .attr('width', width)
        .attr('height', height)
        .attr('class', 'main');

      var per_capita_exp_esrds = [];
      var avg_risk_scores = [];

      Object.keys(data).forEach(function(countyKey) {
        var esrd_value = data[countyKey].per_capita_exp_esrd;
        var avg_risk_score_value = data[countyKey].avg_risk_score_esrd;
        if (esrd_value != 'nan') {
          per_capita_exp_esrds.push(parseFloat(esrd_value));
          avg_risk_scores.push(parseFloat(avg_risk_score_value));
        }
      });

      var x = d3.scale.linear()
                .domain([d3.min(avg_risk_scores), d3.max(avg_risk_scores)])
                .range([ 0, width ]);
      var y = d3.scale.linear()
              .domain([0, d3.max(per_capita_exp_esrds)])
              .range([ height, 0 ]);
      var xAxis = d3.svg.axis()
        .scale(x)
        .orient('bottom');

      var secondPlotXOffset = 53;

      main.append('g')
        .attr('transform', 'translate(' + secondPlotXOffset + ',' +(height*2)+ ')')
        .attr('class', 'main axis date')
        .attr('id', 'scatterplot-xaxis')
        .call(xAxis);

      d3.select('#scatterplot-xaxis')
        .append('text')
        .attr('transform', 'translate(' + (width-260) + ',-5)')
        .text('Average ESRD HCC Risk Score');

      var yAxis = d3.svg.axis()
        .scale(y)
        .orient('left')
        .tickFormat(function(d) { return '$' + d3.format(',')(d) });

      main.append('g')
        .attr('transform', 'translate(' + secondPlotXOffset + ',' + height + ')')
        .attr('class', 'main axis date')
        .attr('id', 'scatterplot-yaxis')
        .call(yAxis);

      d3.select('#scatterplot-yaxis')
        .append('text')
        .attr("transform", "translate(16,210) rotate(-90)")
        .text('Per Capita ESRD Expenditures');

      svg.selectAll('.axis path')
         .style({ 'stroke': 'black', 'fill': 'none', 'stroke-width': '1px'});

      var g = main.append("svg:g")
        .attr('transform', 'translate(' + secondPlotXOffset + ',' + height + ')');

      var scatterData = avg_risk_scores.map(function (e, i) {
          return {avg_risk_score_esrd: e, per_capita_exp_esrd: per_capita_exp_esrds[i]};
      });


      g.selectAll("scatter-dots")
        .data(scatterData)
        .enter().append("svg:circle")
            .attr("cx", function (d,i) { return x(d.avg_risk_score_esrd); } )
            .attr("cy", function (d) { return y(d.per_capita_exp_esrd); } )
            .attr("r", 2)
            .on('mouseover', mouseover)
            .on('mouseout', mouseout);              
    }); 
  });  
  </script>
</div>

Why is this data available for ESRD and not for other conditions? A likely reason is a piece of legislation passed in 1972: the [End Stage Renal Disease Program](https://en.wikipedia.org/wiki/End_Stage_Renal_Disease_Program) extends Medicare coverage to those with stage 5 kidney failure, regardless of age.


Further reading:

* [(Yale News) Hospital prices show 'mind-boggling' variation across U.S. driving up health care costs](https://news.yale.edu/2015/12/15/hospital-prices-show-mind-boggling-variation-across-us-driving-health-care-costs)
