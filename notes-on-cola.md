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

NOTE: If the constraints have cyclic dependency between them, the solver will not apply any constraints. [See here](https://github.com/tgdwyer/WebCola/wiki/Constraints) for more details.



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
