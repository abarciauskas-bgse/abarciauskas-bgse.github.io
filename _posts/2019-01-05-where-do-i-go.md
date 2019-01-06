---
title: ":clock11: Where do I go from here??"
layout: post
date: 2019-01-05 22:25
image: 
headerImage: false
tag:
- visualization
projects: false
category: blog
author: aimeebarciauskas
externalLink: false
---

<style>
  .shadow:hover {
    font-weight: bold;
  }

  .wrapper-normal {
    max-width: 800px;
  }
</style>

<div>
<!--   <link rel="stylesheet" type="text/css" href="https://storage.googleapis.com/xliberation.com/cdn/css/d3direct.css"> -->
  <script type="text/javascript" src="https://d3js.org/d3.v3.min.js"></script>
  <script type="text/javascript" src="https://www.google.com/jsapi"></script>

  <script>
      google.load("jquery", "1");
      google.setOnLoadCallback(function() {
          initialize().then(
              function(control) {
                  doTheTreeViz(control);
              }
          );
      });

      var colorMap = {
          "goal": "#66c2a5",
          "action": "#f46d43",
          "theme": "#3288bd",
          "reference": "#5e4fa2"
      };

      function doTheTreeViz(control) {
          var svg = control.svg;
          var nodes = d3.selectAll('g.node');
          nodes.remove();

          var force = control.force;
          force.nodes(control.nodes)
              .links(control.links)
              .start();


          var link = svg.selectAll("line.link")
              .data(control.links, function(d) {
                  return d.key;
              });

          var linkEnter = link.enter()
              .insert("svg:line", ".node")
              .attr("class", "link")
              .attr("x1", function(d) {
                  return d.source.x;
              })
              .attr("y1", function(d) {
                  return d.source.y;
              })
              .attr("x2", function(d) {
                  return d.target.x;
              })
              .attr("y2", function(d) {
                  return d.target.y;
              })
              .style("stroke", d => colorMap[d.source.nodeType] || colorMap['action'])
              .append("svg:title")
              .text(function(d) {
                  return d.target.name + ":" + d.source.name;
              });


          link.exit().remove();


          var node = svg.selectAll("g.node")
              .data(control.nodes, function(d) {
                  return d.key;
              });

          node.select("circle")
              .style("fill", function(d) {
                  return getColor(d);
              })
              .attr("r", function(d) {
                  return getRadius(d);
              });


          var nodeEnter = node.enter()
              .append("svg:g")
              .attr("class", (d) => `node ${d.nodeType}`)
              .attr("transform", function(d) {
                  return "translate(" + d.x + "," + d.y + ")";
              })
              .on("dblclick", function(d) {
                  control.nodeClickInProgress = false;
                  if (d.url) window.open(d.url);
              })
              .on("click", function(d) {

                  if (!control.nodeClickInProgress) {
                      control.nodeClickInProgress = true;
                      setTimeout(function() {
                          if (control.nodeClickInProgress) {
                              control.nodeClickInProgress = false;
                              if (control.options.nodeFocus) {
                                  d.isCurrentlyFocused = !d.isCurrentlyFocused;
                                  doTheTreeViz(makeFilteredData(control));
                              }
                          }
                      }, control.clickHack);
                  }
              })
              .call(force.drag);

          d3.selectAll(".theme")
              .append("svg:rect")
              .attr("width", 120)
              .attr("height", 30)
              .attr("fill", d => colorMap[d.nodeType])
              .attr("y", -15)
              .attr("x", -3)
              .attr("ry", 5)
              .attr("rx", 5)
              .on("mouseover", function(d) {
                  enhanceNode(d);
              })
              .on("mouseout", function(d) {
                  resetNode(d);
              })
              .append("svg:title").text(function(d) {
                  return d[control.options.nodeLabel]
              });

          d3.selectAll(".goal")
              .append("svg:circle")
              .attr("r", function(d) {
                  return getRadius(d);
              })
              .style("fill", function(d) {
                  return getColor(d);
              })
              .style("stroke", "none")
              .on("mouseover", function(d) {
                  enhanceNode(d);
              })
              .on("mouseout", function(d) {
                  resetNode(d);
              })
              .append("svg:title").text(function(d) {
                  return d[control.options.nodeLabel]
              });

          function enhanceNode(selectedNode) {
              link.filter(function(d) {
                      return d.source.key == selectedNode.key || d.target.key == selectedNode.key;
                  })
                  .style("stroke", d => colorMap[d.nodeType] || colorMap[d.source.nodeType] || colorMap['action'])
                  .style("stroke-width", control.options.routeFocusStrokeWidth);

              if (text) {
                  text.filter(function(d) {
                          return areWeConnected(selectedNode, d);
                      })
                      .style("fill", d => colorMap[d.nodeType] || colorMap['action'])
                      .style("font-weight", "bold");
              }
          }

          function areWeConnected(node1, node2) {
              for (var i = 0; i < control.data.links.length; i++) {
                  var lnk = control.data.links[i];
                  if ((lnk.source.key === node1.key && lnk.target.key === node2.key) ||
                      (lnk.source.key === node2.key && lnk.target.key === node1.key)) return lnk;
              }
              return null;
          }

          function resetNode(selectedNode) {
              link.style("stroke", (d) => {
                      return colorMap[d.nodeType] || colorMap[d.source.nodeType] || colorMap['action'];
                  })
                  .style("stroke-width", control.options.routeStrokeWidth);
              if (text) text.style("font-weight", 'normal');
          }

          if (control.options.nodeLabel) {
              var textShadow = nodeEnter.append("svg:text")
                  .attr("x", function(d) {
                      var x = (d.right || !d.fixed) ?
                          control.options.labelOffset :
                          (-d.dim.width - control.options.labelOffset);
                      return x;
                  })
                  .attr("dy", ".32em")
                  .attr("class", "shadow")
                  .attr("text-anchor", function(d) {
                      return !d.right ? 'start' : 'start';
                  })
                  .style("font-size", control.options.labelFontSize + "px")
                  .style("fill", "white")
                  .style("stroke", "white")
                  .style("stroke-width", "2px")           
                  .text(d => d.name);

              var text = nodeEnter.append("svg:text")
                  .attr("x", function(d) {
                      var x = (d.right || !d.fixed) ?
                          control.options.labelOffset :
                          (-d.dim.width - control.options.labelOffset);
                      return x;
                  })
                  .attr("dy", ".35em")
                  .attr("class", "text")
                  .attr("text-anchor", function(d) {
                      return !d.right ? 'start' : 'start';
                  })
                  .style("font-size", control.options.labelFontSize + "px")
                  .text((d) => d.name)
                  .attr('fill', d => colorMap[d.nodeType] || colorMap['action'])
                  .on("mouseover", function(d) {
                      enhanceNode(d);
                      d3.select(this.parentNode)
                        .selectAll("text")
                        .style('font-weight', 'bold');
                  })
                  .on("mouseout", function(d) {
                      resetNode(d);
                      d3.select(this.parentNode)
                        .selectAll("text")
                        .style('font-weight', 'normal');                      
                  });
          }


          node.exit().remove();
          control.link = svg.selectAll("line.link");
          control.node = svg.selectAll("g.node");
          force.on("tick", tick);

          if (control.options.linkName) {
              link.append("title")
                  .text(function(d) {
                      return d[control.options.linkName];
                  });
          }


          function tick() {
              link.attr("x1", function(d) {
                      return d.source.x;
                  })
                  .attr("y1", function(d) {
                      return d.source.y;
                  })
                  .attr("x2", function(d) {
                      return d.target.x;
                  })
                  .attr("y2", function(d) {
                      return d.target.y;
                  });
              node.attr("transform", function(d) {
                  return "translate(" + d.x + "," + d.y + ")";
              });

          }

          function getRadius(d) {
              return makeRadius(control, d);
          }

          function getColor(d) {
              return control.options.nodeFocus && d.isCurrentlyFocused ? control.options.nodeFocusColor : control.color(d.nodeType);
          }
      }

      function makeRadius(control, d) {
          var r = d.pages ? control.options.radius * d.pages.length : control.options.radius;
          return control.options.nodeFocus && d.isCurrentlyFocused ? control.options.nodeFocusRadius : r;
      }

      function makeFilteredData(control, selectedNode) {

          var newNodes = [];
          var newLinks = [];

          for (var i = 0; i < control.data.links.length; i++) {
              var link = control.data.links[i];
              if (link.target.isCurrentlyFocused || link.source.isCurrentlyFocused) {
                  newLinks.push(link);
                  addNodeIfNotThere(link.source, newNodes);
                  addNodeIfNotThere(link.target, newNodes);
              }
          }

          if (newNodes.length > 0) {
              control.links = newLinks;
              control.nodes = newNodes;
          } else {
              control.nodes = control.data.nodes;
              control.links = control.data.links;
          }
          return control;

          function addNodeIfNotThere(node, nodes) {
              for (var i = 0; i < nodes.length; i++) {
                  if (nodes[i].key == node.key) return i;
              }
              return nodes.push(node) - 1;
          }
      }

      function getPixelDims(scratch, t) {

          scratch.empty();
          scratch.append(document.createTextNode(t));
          return {
              width: scratch.outerWidth(),
              height: scratch.outerHeight()
          };
      }

      function initialize() {
          var initPromise = $.Deferred();
          var control = {};
          control.divName = "#chart";

          var newoptions = {
              nodeLabel: "label",
              nodeResize: "count",
              height: 600,
              nodeFocus: true,
              radius: 5,
              charge: -500
          };

          control.options = $.extend({
              radius: 5,
              labelFontSize: 12,
              width: $(control.divName).outerWidth(),
              linkDistance: 200,
              charge: -150,
              styles: null,
              linkName: null,
              nodeFocus: true,
              nodeFocusRadius: 25,
              nodeFocusColor: colorMap['goal'],
              labelOffset: 5,
              gravity: 0.2,
              routeFocusStroke: colorMap['goal'],
              routeFocusStrokeWidth: 3,
              circleFill: colorMap['action'],
              routeStroke: colorMap['action'],
              routeStrokeWidth: 1,
              height: $(control.divName).outerHeight()

          }, newoptions);

          var options = control.options;
          control.width = options.width;
          control.height = options.height;


          control.scratch = $(document.createElement('span'))
              .addClass('shadow')
              .css('display', 'none')
              .css("font-size", control.options.labelFontSize + "px");
          $('body').append(control.scratch);

          getTheData(control).then((data) => {
              control.data = data;
              control.nodes = data.nodes;
              control.links = data.links;
              control.color = (data) => {
                  return data === 'goal' ? colorMap['goal'] : 'blue'
              };
              control.clickHack = 200;

              control.svg = d3.select(control.divName)
                  .append("svg:svg")
                  .attr("width", control.width)
                  .attr("height", control.height);

              control.force = d3.layout.force()
                  .size([control.width, control.height])
                  .linkDistance(control.options.linkDistance)
                  .charge(control.options.charge)
                  .gravity(control.options.gravity);
              initPromise.resolve(control);
          });
          return initPromise.promise();
      }

      function getTheData(control) {
          var dataPromise = getTheRawData();
          var massage = $.Deferred();
          dataPromise.done((data) => {

                  massage.resolve(dataMassage(control, data));
              })
              .fail(function(error) {
                  console.log(error);
                  massage.reject(error);
              });
          return massage.promise();
      }

      function dataMassage(control, data) {
          var ind = data;
          var nodes = [];
          var links = [];

          for (var i = 0; i < ind.length; i++) {
              ind[i].isCurrentlyFocused = false;
              nodes.push(ind[i]);

              for (var j = 0; j < ind[i].pages.length; j++) {
                  var node = findOrAddPage(control, ind[i].pages[j], nodes);
                  node.isCurrentlyFocused = false;

                  var link = {
                      source: node,
                      target: ind[i],
                      key: node.key + "_" + ind[i].key
                  };
                  links.push(link);
              }
          }

          nodes.sort((a, b) => {
              return a.name < b.name ? -1 : (a.name == b.name ? 0 : 1);
          });
          control.pageCount = 0;
          control.pageRectSize = {
              width: 0,
              height: 0,
              radius: 0
          };
          for (var i = 0; i < nodes.length; i++) {
              page = nodes[i];
              page.group = 0;
              page.dim = getPixelDims(control.scratch, page.name);
              if (page.fixed) {
                  control.pageCount++;

                  control.pageRectSize.width = Math.max(control.pageRectSize.width, page.dim.width);
                  control.pageRectSize.height = Math.max(control.pageRectSize.height, page.dim.height);
                  control.pageRectSize.radius = Math.max(control.pageRectSize.radius, makeRadius(control, page));
                  page.group = 1;
              }

          }
          var options = control.options;


          for (var i = 0, c = 0; i < nodes.length; i++) {
              var page = nodes[i];
              if (page.fixed) {
                  page.right = false;

                  page.y = ((c % control.pageCount) + 5) * (control.pageRectSize.height) + c*5;

                  page.x = page.dim.width + options.labelOffset;
                  c++;
              }

          }

          return {
              nodes: nodes,
              links: links
          };

      }

      function findOrAddPage(control, page, nodes) {
          for (var i = 0; i < nodes.length; i++) {
              if (nodes[i].key === page.key) {
                  nodes[i].count++;
                  return nodes[i];
              }
          }
          page.fixed = true;
          page.count = 0;
          return nodes[nodes.push(page) - 1];
      }




      function getParameterByName(name) {
          name = name.replace(/[\[]/, "\\\[").replace(/[\]]/, "\\\]");
          var regex = new RegExp("[\\?&]" + name + "=([^&#]*)"),
              results = regex.exec(location.search);
          return results == null ? "" : decodeURIComponent(results[1].replace(/\+/g, " "));
      }


      function getTheRawData() {
          var dataUrl = '/assets/data/brainwheel.json';


          return getPromiseData(dataUrl);
      }

      function getPromiseData(url) {
          var deferred = $.Deferred();
          $.getJSON(url, null, (data) => {
                  deferred.resolve(data);
              })
              .error(function(res, status, err) {
                  deferred.reject("error " + err + " for " + url);
              });

          return deferred.promise();
      }
  </script> 
  <div>
    <h2 style="color: #666;">Aimee's Brainwheel</h2>
    <span style="color: #666;">I struggle to focus my free time, having diverse interests and hobbies. I wanted to document everything I have been brainstorming about doing in 2019 and beyond. I have organized and found connections across these interests and hobbies into
        <span style="color: #f46d43; font-weight: bold">actions</span>,
        <span style="color: #5e4fa2; font-weight: bold">references</span>,
        <span style="color: #66c2a5; font-weight: bold">goals</span> and
        <span style="color: #3288bd; font-weight: bold;">themes</span>. <br /><br />
    Next, I am tracking how I spend my free time to see if what I spend the most time on matches my hierarchy of goals.</span>
  </div>
  <br />
  <div id="chart"></div>
  <aside>
      <small>
       ackowledgements:
       <br />
       <a href='http://bost.ocks.org/mike/'>Mike Bostok(d3.js)</a>
       <br />
       <a href='http://ramblings.mcpher.com'>ramblings.mcpher.com</a>
   </small>
  </aside>   
</div>
