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



### Grouping Constraints

In the graph, specify a `groups` member:

```
  "groups": [ { "leaves": [ ...nodes indices of member nodes... ], 
                "groups": [ ...groups indices of member groups... ] } ]
```

[Cola Example (Non-Nested) with Explanation](https://ialab.it.monash.edu/webcola/examples/smallworldwithgroups.html)


### Overlap Constraints

Add `.avoidOverlaps(true)` to your initial `cola` call.

### Edge Routing

There are no explicit edge routing constraints, but `routeEdge` will do shortest path on visibility map.


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
