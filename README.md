Build and test status: [![CircleCI](https://circleci.com/gh/envire/envire-envire_core.svg?style=svg)](https://circleci.com/gh/envire/envire-envire_core)


EnviRe Core
=============
Core library for Environment Representation. The goal of this core part is
to deal with the representation commonalities among plugins.

Environment Representation (EnviRe) technologies are meant to close the gap and
provide techniques to store, operate and interchange information within a
robotic system. The application of EnviRe mainly focus to support navigation,
simulation and operations and simplify the interchange of algorithms among software components.

Installation using Rock's build system
------------
The easiest way to build and install this package is to use Rock's build system.
See [this page](http://rock-robotics.org/documentation/installation.html)
on how to install Rock.

Standalone Installation
------------
First make sure that all dependencies are installed.
Most of the dependencies can be installed using apt:
```
apt install build-essential gcc g++ cmake git wget libgoogle-glog-dev libboost-test-dev libboost-filesystem-dev libboost-serialization-dev libboost-system-dev pkg-config libeigen3-dev libclass-loader-dev libtinyxml-dev librosconsole-bridge-dev libeigen3-dev libclass-loader-dev libtinyxml-dev
```

Some dependencies need to be build from source. A script is provided to install those:
```
chmod +x install_dependencies.sh
sudo ./install_dependencies.sh [path_to_prefix]
```
If no prefix is provided, the dependencies will be installed system-wide.


Rock CMake Macros
-----------------

This package uses a set of CMake helper shipped as the Rock CMake macros.
Documentations is available on [this page](http://rock-robotics.org/documentation/packages/cmake_macros.html).

Rock Standard Layout
--------------------

This directory structure follows some simple rules, to allow for generic build
processes and simplify reuse of this project. Following these rules ensures that
the Rock CMake macros automatically handle the project's build process and
install setup properly.

STRUCTURE
-- src/
	Contains all header (*.h/*.hpp) and source files
-- build/
	The target directory for the build process, temporary content
-- bindings/
	Language bindings for this package, e.g. put into subfolders such as
   |-- ruby/
        Ruby language bindings
-- viz/
        Source files for a vizkit plugin / widget related to this library
-- resources/
	General resources such as images that are needed by the program
-- configuration/
	Configuration files for running the program
-- external/
	When including software that needs a non standard installation process, or one that can be
	easily embedded include the external software directly here
-- doc/
	should contain the existing doxygen file: doxygen.conf


Graph Usage Examples
--------------------
This section contains a few simple usage examples that showcase some of the graph's features.

#### Adding Frames
Frames can be added either explicitly by calling ``addFrame()``
```
EnvireGraph g;
const FrameId frame = "frame_a";
g.addFrame(frame);
```
or implicitly by using a unknown frame id in ``addTransform()``.
```
EnvireGraph g;
const FrameId frameA = "frame_a";
const FrameId frameB = "frame_b";
Transform tf;
g.addTransform(frameA, frameB, tf);
```
Frames cannot be added twice. If a frame with the given name already exists,
an exception will be thrown.

The above examples will create the frame property using the default constructor.
Another constructor can be used by calling ``emplaceFrame()``. Calling
``emplaceFrame()`` does only make sense, if the frame property has non-default
constructors.

#### Removing Frames
Frames can be removed by calling ``removeFrame()``:
```
EnvireGraph g;
const FrameId frame = "frame_a";
g.addFrame(frame);
g.disconnectFrame(frame);
g.removeFrame(frame);
```

``disconnectFrame()`` removes all transforms that are connected to the given frame.
Frames can only be removed, if they are not connected to the graph. I.e. if no
edges are connected to the frame. An exception will be thrown, if the frame is
still connected. This is an artificial restriction, technically it would be
possible to remove frames while they are still connected. The intention of this
restriction is, to make the user aware of the consequences that removing a frame
might have for the graph structure as a whole.


#### Loading Envire Plugins
Envire Plugins provide ...
TODO
TODO
TODO
TODO
#### Creating Items
Before an item can be added to a frame, it has to be loaded using the ``ClassLoader``.
```
#include <envire_core/plugin/ClassLoader.hpp>
#include <envire_core/items/Item.hpp>
#include <octomap/AbstractOcTree.h>
```
```
envire::core::Item<boost::shared_ptr<octomap::AbstractOcTree>>::Ptr octree;
ClassLoader* loader = ClassLoader::getInstance();
if(!loader->createEnvireItem("envire::core::Item<boost::shared_ptr<octomap::AbstractOcTree>>", octree))
{
	std::cerr << "Unabled to load envire::octomap::OcTree" << std::endl;
	return -1;
}
```

It is also possible to instantiate items directly, however this is only
recommended for testing because visualization and serialization only work if
the ``ClassLoader`` was used to load the item.

#### Adding Item
Once the item is loaded, there are two ways to add it to the graph.
The common way is to add it using ``addItemToFrame()``:
```
g.addItemToFrame(frame, octree);
```
The item will remember the frame that it was added to. I.e. an item cannot be part of two frames at the same time.

It is also possible to set the frame id beforehand and add the item using
``addItem()``.
```
octree->setFrame(frame);
g.addItem(octree);
```

The item type can be a ``boost::shared_ptr`` to any subclass of ``ItemBase``.
Item contains a typedef ``Ptr`` to make working with the pointer more convenient.
```
envire::core::Item<...>::Ptr p;
```


#### Accessing Items
When working with items, the user needs to know the item type. The type can
either be provided at compile time using template parameters or at runtime using
``std::type_index``.

#### Checking Whether a Frame Contains Items of a Specific Type
``containsItems()`` is used to check for the existence of items of a given type
in a given frame.
```
const bool contains = g.containsItems<envire::core::Item<boost::shared_ptr<octomap::AbstractOcTree>>>(frame);
```

If the type is not known at compile time, there is also an overload that
accepts ``std::type_index``. You can get the type index by calling
``getTypeIndex()`` on any ``Item``.

```
const std::type_index index(octree->getTypeIndex());
const bool contains2 = g.containsItems(frame, index);
```


#### Accessing Items with Iterators

The ``ItemIterator`` can be used to iterate over all items of a specific type
in a frame. The iterator internally takes care of the necessary type casting
and type checks.
```
using OcTreeItem = envire::core::Item<boost::shared_ptr<octomap::AbstractOcTree>>;  
using OcTreeItemIt = EnvireGraph::ItemIterator<envire::core::Item<boost::shared_ptr<octomap::AbstractOcTree>>>;
OcTreeItemIt it, end;
std::tie(it, end) = g.getItems<envire::core::Item<boost::shared_ptr<octomap::AbstractOcTree>>>(frame);
for(; it != end; ++it)
{
	std::cout << "Item uuid: " << it->getIDString() << std::endl;
}
```

A convenience method exist to get an ``ItemIterator`` of the i'th item:
```
OcTreeItemIt itemIt = g.getItem<OcTreeItem>(frame, 42);
```

#### Accessing Items without Iterators
If type information is not available at compile time, ``getItems()`` can also
be used with ``std::type_index``:
```const std::type_index index2(octree->getTypeIndex());
const Frame::ItemList& items = g.getItems(frame, index2);
```
However without compile time type information automatic type casting is not
available, thus in this case ``getItems`` returns a list of ``ItemBase::Ptr``.
The list is returned as reference and points to graph internal memory.


#### Removing Items

Items can be removed by calling ``removeItemFromFrame()``. Removing items invalidates
all iterators of the same type. To be able to iteratively remove items, the
method returns a new pair of iterators.
```
OcTreeItemIt i, endI;
std::tie(i, endI) = g.getItems<OcTreeItem>(frame);
for(; i != endI;)
{
		std::tie(i, endI) = g.removeItemFromFrame(frame, i);
}
```

All items can be removed at once using ``clearFrame()``.
```
g.clearFrame(frame);
```

#### Adding Transformations
```
EnvireGraph g;
const FrameId a = "frame_a";
const FrameId b = "frame_b";
Transform ab;
/** initialize Transform */
g.addTransform(a, b, ab);
```
If a transformation is added, the inverse will be added automatically.
If one or both of the frames are not part of the graph, they will be added.

#### Removing Transformations
```
g.removeTransform(a, b);
```
The inverse will be removed as well.

#### Modifying Transformations
Transformations can be replaced using ``updateTransform``.
The inverse will be updated automatically.
```
Transform tf;
tf.transform.translation << 84, 21, 42;
g.updateTransform(a, b, tf);
```


#### Calculating Transformations
``getTransform()`` can be used to calculate the transformation between two
frames if a path connecting the two exists in the graph. Breadth first search is
used to find the path connecting the two frames.
```
const Transform tf2 = g.getTransform(a, b);
```

Calculating the transformation between two frames might be expensive depending
on the complexity of the graph structure. A ``TreeView`` can be used to speed
up the calculation:
```
TreeView view = g.getTree(g.getVertex(a));
const Transform tf3 = g.getTransform(a, b, view);
```

Since creating the ``TreeView`` walks the whole graph once, using this methods
only makes sense when multiple transformations need to be calculated.

If you need to calculate the same transformation multiple times, you can
use ``getPath()`` to retrieve a list of all frames that need to be traversed
to calculate the transformation. The path can be used to speed up the calculation
of the transform even further.
```
envire::core::Path::Ptr path = g.getPath(a, b, false);
const Transform tf4 = g.getTransform(path);
```


#### Disconnecting a Frame from the Graph
``disconnectFrame()`` can be used to remove all transformations coming from
or leading to a certain frame.

#### TreeViews

``TreeViews`` provide a tree view of the graph structure. I.e. when viewed
through a ``TreeView`` the graph turns into a tree with a specific root node.

TreeViews use vertex_descriptors instead of FrameIds to reference frames because
vertex_descriptors can be hashed in constant time (they are just pointers).

#### Creating Tree Views
TreeViews can be created by calling ``getTree()`` and providing a root node.
```
EnvireGraph g;
const FrameId root("root");
TreeView view = g.getTree(root);
```

Note that the view will most likely be copied on return. If the tree is large
you might want to avoid that copy and pass an empty view as out-parameter instead:
```
TreeView view2;
g.getTree(root, &view2);
```

#### Updating Tree Views

By default, a tree view shows a snapshot of the graph. I.e. if the graph changes,
the changes will not be visible in the view. The view or parts of it might
become invalid when vertices or edges are removed from the graph.
To avoid this, you can request a self-updating tree view:
```
g.getTree(root, true, &view);
```

The view has three signals ``crossEdgeAdded``, ``edgeAdded`` and ``edgeRemoved``
that will be emitted whenever the tree view changes.


Maintenance and development
--------------------
DFKI GmbH - Robotics Innovation Center

![alt tag](https://github.com/envire/envire.github.io/raw/master/images/dfki_logo.jpg)
