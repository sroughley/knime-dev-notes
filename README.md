# knime-dev-notes

This project is aimed at being a place where notes of learnings that may not be obvious around writing KNIME extensions 
(almost entirely nodes, but I'll keep that looser definition)

## Content

### NodeSets

`NodeSetFactory`s are used in place of the standard `NodeFactory` extension point when you need to create one or more (usually the latter) nodes programmatically at 
runtime - e.g. from a custom extension point, user config file, or something else on the classpath (This latter example is how the KNIME WEKA nodes are created)
See [Node Sets](/notes/NodeSets.md) for further details of this lightly-documented feature

## Contributing
If you spot errors or have additions feel free to create an issue or submit a pull request
