# Node Sets

## Introduction
Node Sets are used as an extension point when one or (more usually) more nodes are needed to be generated computationally at runtime.
This can be from a custom extension point, a config file of some sort, or even the classpath. 
The [KNIME WEKA](https://hub.knime.com/knime/extensions/org.knime.features.ext.weka_3.7/latest) uses this extension point to generate
the WEKA KNIME nodes from WEKA classes on the classpath at runtime, but the feature is not very well documented and has a few 'gotcha's
particularly if you are working with KNIME Server.

## Overview
In addition to a `NodeFactory` implementation, as for a normal node, this approach requires an implementation of `NodeSetFactory`.  
The `NodeFactory` class(es) are _not_ included in the `plugin.xml` node extension point as with a normal node implementation.  Instead,
a single `NodeSetFactory` is pointed to, which will itself return one or more `NodeFactory` classes for the nodes generated.
In this case, multiple nodes can share a single `NodeFactory` implementation, _via_ the settings loaded in 
`loadAdditionalFactorySettings(ConfigRO)` method.
The `plugin.xml` file has a block for the Node Set along the following lines:

```xml
    <extension point="org.knime.workbench.repository.nodesets">
        <nodeset
            default-category-icon="icons/my-icon.png"
            deprecated="false"
            factory-class="path.to.MyNodeSetFactory"
            id="MyNodeSetFactory" />
    </extension>
```

* `default-category-icon` points to an icon to use for any node categories created in the node repository for which an icon is not provided
* `deprecated` indicates whether _all_ nodes in this Node Set are deprecated
* `factory-class` - hopefully fairly obvious - in this case a `NodeSetFactory` rather than a `NodeFactory`
* `id` - no longer use, so can be ignored or not included at all

## 'NodeSetFactory' implementation

## NodeFactory implementation

## NodeModel and NodeDialogPane implementations

## Deprecating individual nodes in a Node Set

