# Random things on parsing DOT based on experience

## Parsing Dot into Cola
There is an example named [Unix Family tree](https://ialab.it.monash.edu/webcola/examples/unix.html) of parsing a dot file into Javascript and it uses the library [cola.js](https://ialab.it.monash.edu/webcola/). It uses [graphlib-dot.js](https://github.com/dagrejs/graphlib-dot) library which is a dot parser and writer for graphlib. The problem with this parser is - It is able to parse the simple dot instructions such as in this particular example the graphlib-dot parser is parsing only the nodes and the directed edges between them. Other than that, if we want to change the properties of the nodes such as if we want to change the background color of the nodes or if we want to change the shape of any node from rectangular to diamond, the graphlib-dot parser is not able to parse those syntaxes. 

Also, the parser does not parse any arbitrary value. It does not work with HTML strings and ports in dot files. 

All those facts mentioned above is also true for [this](https://github.com/dagrejs/dagre-d3/blob/master/demo/interactive-demo.html) interactive example. Here, it is parsing dot for javascript which is using [dagre.js](https://github.com/dagrejs/dagre/wiki). As, it is also using graphlib-dot.js library, we can only parse the very basic syntaxes of dot. 
