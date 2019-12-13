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


## Drawing Help

### Arrow Heads

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

Here's a sketch:

CSS:
```
.labels {
  font: 8pt sans-serif;
  text-anchor: middle;
}
```

JS:
```
var node_group = svg.append('g');
var label_group = svg.append('g'); // Ensure labels always on top of nodes
var margin = 2;

// Append nodes as normal
var node = node_group.selectAll('.nodes')
    .data(graph.nodes)
  .enter.append('rect')
    .attr('class', 'nodes');

// Append labels as normal
var label = link_group.selectAll('.labels')
    .data(graph.nodes)
  .enter.append('text')
    .attr('class', 'labels')
    .text(d => d.name);

// While Cola updates the layout, we can update the size
// and placements of node and text for the labels
d3cola.on('tick', function() {
  // Find the bounding box of the text labels.
  // They determine how big the node has to be.
  label.each(function(d) {
    var b = this.getBBox();
    d.width = b.width + 2 * margin + 2;
    d.height = b.height + 2 * margin + 2;
  })

  // Update node position with the tick.
  // Update width & height based on earlier bounding box calculation
  node.attr('x', d => d.x)
      .attr('y', d => d.y)
      .attr('width', d => d.width)
      .attr('height', d=> d.height);
 
  // Update label position with the tick. Depending on your CSS,
  // adjustments will need to be made to center in the node to your liking.
  label.attr('x', d => { return d.x + d.width/2; })
       .attr('y', d => { return d.y + margin + d.height/2; });
});

```
