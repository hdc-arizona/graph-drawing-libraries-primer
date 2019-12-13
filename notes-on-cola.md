## Collected institutional knowledge and advice for using cola.js

[Cola.js website](https://ialab.it.monash.edu/webcola/)

### Getting Started & General tips

#### Graph Format

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

#### Versions

If you have having difficulty getting an example to work, make sure the versions of the libraries you're using match the versions in the example. Not all examples have necessarily been updated to the latest version of the library.

### Constraints

How to specify the constraints in code

#### Alignment Constraints

#### Grouping Constraints
