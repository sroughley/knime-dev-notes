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

## NodeSetFactory implementation
This class is used to enumerate all the nodes in the set and put them in the correct places in the node repository.  Each node has a `String` id, 
and a [NodeFactory](#nodefactory-implementation) implementation.  The individual methods are discussed below.  A simple implementation has a 
`Map<String,Class<? extends NodeFactory<? extends NodeModel>>>` field in which the keys are the node IDs, and the values are the corresponding 
`NodeFactory` classes.

For a couple of examples implementations, see 
[VirtualNodeSetFactory](https://github.com/knime/knime-core/blob/master/org.knime.core/src/eclipse/org/knime/core/node/workflow/virtual/VirtualNodeSetFactory.java) 
and `org.knime.ext.weka.WekaNodeSetFactory`

### getNodeFactoryIds()
This method needs to return `Collection<String>` - for a simple implementation using an ID-Factory Class map as suggested above, this can be as simple as:

```java
@Override
public Collection<String> getNodeFactoryIds() {
    return clzMap.keySet();
}
```
(where `clzMap` is the ID-Factory class map)


### getNodeFactory(String)
Slightly confusingly, this method needs to return the _class_ of the `NodeFactory` implementation, rather than an actual instance of the `NodeFactory` 
implementation. Multiple different nodes may return the same value here - the details being resolved by the outcome of the 
[getAdditionalSettings(String)](#getadditionalsettingsstring) method.  Again, for a simple map-based implementation, this can be as simple as:

```java
@Override
public getNodeFactory(String id) {
    return clzMap.get(id);
}
```

### getCategoryPath(String)
This method returns the path in the node category in exactly the same manner as it is usually defined in a standard Node extension point.  
If the node is a 'hidden' node, then `null` may be returned

### getAfterID(String)
This method remains a bit of a mystery to me in terms of what should actually be returned.  It is supposed to return the node id of the node in the 
repository which it is to be placed after, however, nodes in the node set get an ID in the repository based on the node set name and the 'id' field, 
but this isn't recognised by the repository loader at the point at which it is used.  The WEKA implementation returns an empty string, `""`, and 
the `VirtualNodeSetFactory` returns `null`, but these nodes are all 'hidden'.  Currently best to return an empty string and let the nodes order 
themselves

### getAdditionalSettings(String)
This method is where differentiation of nodes sharing a NodeFactory implementation takes place.  When the NodeFactory is constructed, either 
in the enumeration in the node repository, or when adding a copy of the node to a workflow, the result of this method is passed as an argument 
to the `NodeFactory#loadAdditionalFactorySettings(ConfigRO)` method. A typical implementation for which a NodeFactory needs a `String` 'name' 
and an `int` 'value' to initialize itself might look something like:

```java
@Override
public ConfigRO getAdditionalSettings(String id) {
    NodeSettings settings = new NodeSettings("my-settings");
    String name = getNameForId(id);
    int value = getValueForId(id);
    settings.addString("name", name);
    settings.addInt("value", value);
    return settings;
}
```

* The key used to define `settings` is never used anywhere else, and so can be just about any valid String (it is not saved anywhere)
* For the individual setting keys (`"name"`, `"value"` in this example snippet), good practice is to declare these as constants which
* are accessible to all `NodeFactory` implementations, as they will be needed there too in order to correctly load the settings
* If a `NodeFactory` does not need any setting (e.g. because it is the only node in the set using the particular implementation), then
* `null` may be returned here

### isHidden()
This is a `default` method in the `NodeSetFactory` interface.  In most cases, it will not need to be overridden.  However, if the 
node set is a set of 'hidden' nodes, then it does need to be overridden.  NB it is not possible to mix hidden and non-hidden nodes 
in a single set

## NodeFactory implementation

## NodeModel and NodeDialogPane implementations

## Deprecating individual nodes in a Node Set

