# Collected institutional knowledge and advice for using cola.js

[Cola.js website](https://ialab.it.monash.edu/webcola/)

Note that the [Github repository for Cola.js has a Wiki](https://github.com/tgdwyer/WebCola/wiki) that documents some of the API.

There's also the [API](https://ialab.it.monash.edu/webcola/doc/index.html) which lists the available functions without documentation.

## Getting Started & General tips

### Graph Format

A graph is a `JSON` object with two arrays of objects: `nodes` and `links`. Nodes are identified by their index in the `nodes` array. Links have `source` and `target` members that identify by hese `nodes` array indices.

Optionally, a `groups` array exist with objects describing the group. The `leaves` member of a group object lists the indices of the nodes from the `nodes` array. The `groups` member of a group object lists the indices of the groups from the `groups` array. This allows nested groups.

```
{
  "nodes": [ ...objects... ],
  "links": [ { "source": src_id, "target": target_id } ], 
             // ^-- these IDs are indices in the nodes array!
  "groups": [ { "leaves": [ ...nodes indices of member nodes... ], 
                "groups": [ ...groups indices of member groups... ] } ]
}
```

### Cola Layout

```
var d3cola = cola.d3adaptor(d3)
  .size([width, height]); // e.g., SVG size
```

Several other functions dictating layout behavior can be appended here.

Then, give the layout engine the graph and start:

```
d3cola
  .nodes(graph.nodes)
  .links(graph.links)
  .start()
```


### Troubleshooting

**Versions:** If you have having difficulty getting an example to work, make sure the versions of the libraries you're using match the versions in the example. Not all examples have necessarily been updated to the latest version of the library.


## Specifying link lengths

We can set the ideal length for all the links using 

```
cola.d3adaptor().linkDistance(30) 
```
Here, Cola will try to keep the lengths of all edges close to the value of 30.

### Set custom edge lengths for each edge (Link accessor)
To set custom ideal lengths for each edge, use the following code:

```
var d3cola = cola.d3adaptor()
            .linkDistance(function (l) { return l.length })
```

In the above example, the length for each edge is taken from the `links` array where each object has a `length` attribute along with the `source` and `target` attribute.

e.g. 
```
"links": [{"source": src_id, "target": target_id, "length": 20}, ...]

```

The ideal lengths can also be specified using predefined functions such as `symmetricDiffLinkLengths` and `jaccardLinkLengths`.

We set these in the following way 

```
d3cola
  .links(links)
  .jaccardLinkLengths(40)
  .start();
```

The jaccard/symmetric link lengths are computed in the `start` function.
For their definition, visit [cola's wiki page](https://github.com/tgdwyer/WebCola/wiki/link-lengths) .


## Constraints

How to specify the constraints in code

To specify the constraints, we create an array with constraint objects e.g. `myconstraints` and set it as

```
d3cola.constraints(myconstraints);
```

### Alignment Constraints
```
{
    "type": "alignment",
    "axis": "x",
    "offsets": [
        {"node": "1","offset": "0"},
        {"node": "2", "offset": "0"},
        {"node": "3", "offset": "0"}
    ]
}
```

Set the `type` to `alignment`.

The alignment is axis-aligned. The `axis` supports two directions `x` and `y`.

The nodes are listed in the array named `offsets`. This array contains a list of objects.
The `node` field contains the index of the nodes in the `d3.nodes(nodes)` array. The `offset` defines if there is any offset to the left or right when applying the alignment constraints. For e.g. in the example above, nodes with indices 1, 2, and 3 will be aligned along x axis. An `offset` of 0 causes the nodes to be aligned at the center. To align all the nodes along the top, specify the offset to be half the height of the node. 

TODO: Update the direction of the offset. Does -ve offset mean to the left/top and vice-versa.

### Separation(Inequality) Constraints
The separation constraints add a minimum separation between a pair of nodes along x/y axis.

```
{"axis":"y", "left":0, "right":1, "gap":25}
```

This code specifies that the center of `nodes[0]` must be at least 25 pixels
above the center of `nodes[1]`. In other words, it is an inequality constraint of the form

```
nodes[0].y + gap <= nodes[1].y
```

The `axis` supports both `x` and `y` directions. The left/right is analogous to top/bottom when the axis is `y`.

NOTE: If the constraints have cyclic dependency between them, the solver will not apply any constraints. [Click here](https://github.com/tgdwyer/WebCola/wiki/Constraints) for more details.



Separation constraints also support equalities. To change the inequality constraint into an equality constraint, add `"equality":"true"` to the constraint.

```
{"axis":"y", "left":0, "right":1, "gap":25, "equality":"true"}
```

The code specifies the following:
```
nodes[0].y + gap = nodes[1].y
```

### Grouping Constraints

In the graph, specify a `groups` member as described in the [Graph
Format](#graph-format) section:

```
  "groups": [ { "leaves": [ ...nodes indices of member nodes... ], 
                "groups": [ ...groups indices of member groups... ] } ]
```

With groups available, add a `.groups(graph.groups)` call to your `cola`
setup.


[Cola Example (Non-Nested) with Explanation](https://ialab.it.monash.edu/webcola/examples/smallworldwithgroups.html)


### Overlap Constraints

Add `.avoidOverlaps(true)` to your initial `cola` call.

### Edge Routing

There are no explicit edge routing constraints, but `routeEdge` will do shortest path on visibility map.

### Fixed node positions

To fix the node positions, set the `fixed` field to `true` inside the node object.

### Setting explicit x/y coordinates

To set the x and y coordinates of the node, set `x` and `y` fields inside the node object. 
NOTE: If the coordinates are not fixed, these values will be updated on the next tick.


## Drawing Help

### Arrow Heads
To have a directed graph layout, you have to add arrowhead on the edges manually. The code structure for adding an arrowhead should look like this:

```
svg.append("svg:defs").append("svg:marker")//SVG defs are a way of defining graphical objects which can be applied to elements
    .attr("id", "triangle")                //we are basically adding a marker 
    .attr("refX", 15)                
    .attr("refY", -1.5)
    .attr("markerWidth", 6)
    .attr("markerHeight", 6)
    .attr("orient", "auto")               //orientation of the marker is auto so that it can fit the direction of the path that uses it   
    .append("path")
    .attr("d", "M 0 0 12 6 0 12 3 6")     //defining the triangle(arrowhead)
    .style("fill", "black");


```
[Click here to see more explanation] (http://tutorials.jenkov.com/svg/marker-element.html)

### Drawing directed Graphs (Flow Layout)
Flow layout adds downward separation constraints for each edge. For a directed edge (1,2) where 1 is the node index of the source node and 2 is the index for the target node, it adds constraints of the form 

```
nodes[1].y + gap <= nodes[2].y
```

This can be done using the following code

```
d3cola
  .flowLayout('y', gap)
  ...
```

The [example from webcola](https://ialab.it.monash.edu/webcola/examples/unix.html) says "flowLayout causes all edges not involved in a cycle (strongly connected component) to have a separation constraint generated between their source and sink, with a minimum spacing set to gap"


The default direction for flow layout is along `y` axis which achieves vertical layout.
Flow layout also supports `x` axis which achieves left-to-right layout.

Instead of setting a global minimum separation value of `gap` between each edge endpoints, we can set custom function with link accessor to set different minimum separation for each edge.


### Text on Nodes

You will need to calculate the the width and height of the node based on the
text and draw. The text is appended in a separate D3 call. Like most text
needs in SVG, much of the behavior should be set in the CSS, such as the
anchor (e.g., `middle`). 

The [Sucrose
Breakdown
Example](https://ialab.it.monash.edu/webcola/examples/SucroseBreakdown.html)
uses `tspan` with `dy` to make each word in the level on a separate line. It
uses `getBBox` to then determine the extents of the text as it updates. Using
these extents, it is able to update `width` and `height` members of the node
object, which are then used to draw the boxes. Much of the detail is in the
cola `tick` function.

TODO: Add shorter example with copy-paste code. 
