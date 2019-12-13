# Collected institutional knowledge and advice for using cola.js

[Cola.js website](https://ialab.it.monash.edu/webcola/)

Note that the [Github repository for Cola.js has a Wiki](https://github.com/tgdwyer/WebCola/wiki) that documents some of the API.

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

To specify the constraints, we create an array named `constraints`
and set it as

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

### Grouping Constraints

In the graph, specify a `groups` member:

```
  "groups": [ { "leaves": [ ...nodes indices of member nodes... ], 
                "groups": [ ...groups indices of member groups... ] } ]
```

### Overlap Constraints

Add `.avoidOverlaps(true)` to your initial `cola` call.
