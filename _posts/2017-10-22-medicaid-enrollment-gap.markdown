---
title: ":hospital: Which states are underenrolled in Medicaid?"
layout: post
date: 2017-09-12 21:37
image: 
headerImage: false
tag:
- healthcare
- medicaid
- data-exploration
- visualization
projects: false
description: Exploration of the Medicaid enrollment
category: blog
author: aimeebarciauskas
externalLink: false
---

At Nava we've started working on a cool idea: [integrated benefits](https://www.navapbc.com/work/benefits-partnership/). Integrated benefits is the idea that any given individual or household may be eligible for one or more social services programs and in order to enroll in those programs, they shouldn't have to provide the same information multiple times.

Many of the social services are administered at the state level. How do we prioritize where to develop integrated services?

One simplistic approach I'm trying is to see where there is the greatest need - where are there many people who are missing out on these services?

Using Medicaid, poverty and population data from census.gov and medicaid.gov, we can estimate a "Medicaid enrollment gap": The difference between the percent of population enrolled in Medicaid and the percent living below the poverty line.

One thing I was surprised to find was for most states, this "gap" is positive: more individuals are enrolled in Medicaid than are living under the poverty line. However, this shouldn't have been surprising. Each state is given freedom to enforce it's own Medicaid eligibility rules.

The [ACA included one provision on Medicaid which was intended to be uniform across states](https://en.wikipedia.org/wiki/Medicaid): everyone with income below 133% of the poverty line is eligible for Medicaid. However this provision of the ACA was overruled in the Supreme Court:

> However, the Supreme Court ruled in NFIB v. Sebelius that this provision of the ACA was coercive, and that the federal government must allow states to continue at pre-ACA levels of funding and eligibility if they chose.

So while most states have more Medicaid enrollees than population living under the poverty line, a few states do not, and those states really appear as outliers. And you can really see that those states with smaller or negative gaps between Medicaid enrollees and population in poverty correspond to those states who have chosen not to expand Medicaid under the ACA.

## Difference between Medicaid Enrollment and Population in Poverty

The difference between the percent of population enrolled in Medicaid and the percent living below the poverty line.
<style>

/* stylesheet for your custom graph */

.states {
  fill: none;
  stroke: #fff;
  stroke-linejoin: round;
}

.states-choropleth {
  fill: #ccc;
}

#tooltip-container {
  position: absolute;
  background-color: #fff;
  color: #000;
  padding: 10px;
  border: 1px solid;
  display: none;
}

.tooltip_key {
  font-weight: bold;
}

.tooltip_value {
  margin-left: 20px;
  float: right;
}

</style>

<div>
  <div id="tooltip-container"></div>

  <div id="canvas-svg"></div>

  <script src="https://cdnjs.cloudflare.com/ajax/libs/d3/3.5.5/d3.min.js"></script>
  <script src="//cdnjs.cloudflare.com/ajax/libs/topojson/1.1.0/topojson.min.js"></script>
  <script src="https://ajax.googleapis.com/ajax/libs/jquery/2.1.3/jquery.min.js"></script>
  <script src="https://d3js.org/colorbrewer.v1.min.js"></script>

  <script>
    const dataSource = "/assets/data/medicaid-poverty-gap.csv";
    d3.csv(dataSource, function(err, data) {

    var config = {
      "stateDataColumn":"NAME",
      "valueDataColumn":"gap"
    };

    var WIDTH = 800, HEIGHT = 500;
    
    var SCALE = 0.7;
    
    function valueFormat(d) {
      if (d > 1000000000) {
        return Math.round(d / 1000000000 * 10) / 10 + "B";
      } else if (d > 1000000) {
        return Math.round(d / 1000000 * 10) / 10 + "M";
      } else if (d > 1000) {
        return Math.round(d / 1000 * 10) / 10 + "K";
      } else {
        return d;
      }
    }
    
    var MAP_STATE = config.stateDataColumn;
    var MAP_VALUE = config.valueDataColumn;
    
    var width = WIDTH,
        height = HEIGHT;
    
    var valueById = d3.map();
    
    var COLOR_COUNTS = 9;
    var quantize = d3.scale.quantize()
        .domain([0, 1.0])
        .range(d3.range(COLOR_COUNTS).map(function(i) { return i }));
    
    var colorScale = d3.scale.quantize()
        .range(colorbrewer.RdYlGn[COLOR_COUNTS])
        .domain([0, COLOR_COUNTS]); 

    var path = d3.geo.path();
    
    var svg = d3.select("#canvas-svg").append("svg")
        .attr("width", width)
        .attr("height", height);
    
    d3.tsv("https://s3-us-west-2.amazonaws.com/vida-public/geo/us-state-names.tsv", function(error, names) {
    
    name_id_map = {};
    id_name_map = {};
    
    for (var i = 0; i < names.length; i++) {
      name_id_map[names[i].name] = names[i].id;
      id_name_map[names[i].id] = names[i].name;
    }
    
    data.forEach(function(d) {
      var id = name_id_map[d[MAP_STATE]];
      valueById.set(id, +d[MAP_VALUE]); 
    });
    
    quantize.domain([d3.min(data, function(d){ return +d[MAP_VALUE] }),
      d3.max(data, function(d){ return +d[MAP_VALUE] })]);
    
    d3.json("https://s3-us-west-2.amazonaws.com/vida-public/geo/us.json", function(error, us) {
      svg.append("g")
          .attr("class", "states-choropleth")
        .selectAll("path")
          .data(topojson.feature(us, us.objects.states).features)
        .enter().append("path")
          .attr("transform", "scale(" + SCALE + ")")
          .style("fill", function(d) {
            if (valueById.get(d.id)) {
              var i = quantize(valueById.get(d.id));
              return colorScale(i);
            } else {
              return "";
            }
          })
          .attr("d", path)
          .on("mousemove", function(d) {
              var html = "";
    
              html += "<div class=\"tooltip_kv\">";
              html += "<span class=\"tooltip_key\">";
              html += id_name_map[d.id];
              html += "</span>";
              html += "<span class=\"tooltip_value\">";
              html += (valueById.get(d.id) ? valueFormat(valueById.get(d.id)) + "%": "");
              html += "";
              html += "</span>";
              html += "</div>";
              
              $("#tooltip-container").html(html);
              $(this).attr("fill-opacity", "0.8");
              $("#tooltip-container").show();
              
              var coordinates = d3.mouse(this);
              
              var map_width = $('.states-choropleth')[0].getBoundingClientRect().width;
              
              if (d3.event.layerX < map_width / 2) {
                d3.select("#tooltip-container")
                  .style("top", (d3.event.layerY + 15) + "px")
                  .style("left", (d3.event.layerX + 15) + "px");
              } else {
                var tooltip_width = $("#tooltip-container").width();
                d3.select("#tooltip-container")
                  .style("top", (d3.event.layerY + 15) + "px")
                  .style("left", (d3.event.layerX - tooltip_width - 30) + "px");
              }
          })
          .on("mouseout", function() {
                  $(this).attr("fill-opacity", "1.0");
                  $("#tooltip-container").hide();
              });
    
      svg.append("path")
          .datum(topojson.mesh(us, us.objects.states, function(a, b) { return a !== b; }))
          .attr("class", "states")
          .attr("transform", "scale(" + SCALE + ")")
          .attr("d", path);
    });
    
    });
  });

  </script>
</div>

![Current Status of Medicaid Expansion Decisions](https://kaiserfamilyfoundation.files.wordpress.com/2016/10/current-status-of-the-medicaid-expansion-decisions-healthreform.png)

From the [Kaiser Family Foundation](https://www.kff.org/health-reform/slide/current-status-of-the-medicaid-expansion-decision/)

# Data

* Poverty data from census.gov
  * [page](https://www.census.gov/data/tables/2017/demo/income-poverty/p60-259.html)
  * [xls file](https://www2.census.gov/programs-surveys/demo/tables/p60/259/statepov.xls)
* Population data from census.gov
  * [page](https://www2.census.gov/programs-surveys/popest/datasets/2010-2016/state/asrh/)
  * [csv file](https://www2.census.gov/programs-surveys/popest/datasets/2010-2016/state/asrh/scprc-est2016-18+pop-res.csv)
* Medicaid enrollment from medicaid.gov
  * [page](https://www.medicaid.gov/medicaid/program-information/medicaid-and-chip-enrollment-data/enrollment-mbes/index.html)
  * [pdf file](https://www.medicaid.gov/medicaid/program-information/downloads/cms-64-enrollment-report-jul-aug-2016.pdf) - copy / pasted data to create CSV
  * data not available for South Dakota

