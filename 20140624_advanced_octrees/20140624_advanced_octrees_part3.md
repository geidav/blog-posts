# Advanced Octrees 3: non-static Octrees

Typically, Octrees are constructed once for geometry which is known a priori and doesn't change anymore (e.g. the level in a computer game). However, there are applications where objects are moving through the world, or objects located outside the Octree's root cell should be inserted after the Octree has been constructed already. One approach would be simply reconstructing the entire Octree from scratch each time it's modified. While working, obviously, this turns very inefficient as soon as the Octree contains more than just a handful objects. In this post I present a way to relocate moving objects in already constructed Octrees, as well as a way to expand/shrink Octrees.

Duplicating objects straddling cell boundaries works great for static scenes. However, in dynamic scenes keeping track of multiple references to the same object contained in multiple different nodes adds unnecessary complexity. Therefore, it's better to place objects in the lowest Octree cell which completely encloses the object (see [part one](http://geidav.wordpress.com/2014/07/18/advanced-octrees-1-preliminaries-insertion-strategies-and-max-tree-depth/)). That way a minimum amount of pointer information must be updated when a moving object transitions from one cell into another.

## Moving objects
An Octree containing moving objects must be updated whenever a moving object starts straddling its parent cell's bounding box. Usually, the number of moving objects is considerably smaller than the number of static objects. Thus, it can be advantageous to maintain two Octrees: one containing all static and one containing all moving objects. Instead of reconstructing the Octree from scratch, it's much faster to relocate only the objects that have moved out of their parent cell.

To do so, first, a list of all objects that have moved out of their parent cell is obtained. Each object in this list is pushed up the Octree until it ends up in a node which completely encloses it. As long as an object is still straddling its parent cell's bounding box the object must stay inside this cell. However, when the object keeps on moving into the same direction, there will be the moment it's again completely enclosed by one of its parent's child cells. Therefore, finally, all previously pushed up objects are tried to be moved down the Octree again, in order to place them in the smallest enclosing cell possible.

When relocating objects in an Octree it can happen that some objects move out of the root cell. In that case the Octree must be expanded as described in the following section. It can also happen that after pushing up an object leaving a node, this node and all its child nodes remain empty. In that case the node and its children should be removed from the tree.

## Expanding and shrinking
Sometimes, the extent of the world isn't known at the time the Octree is constructed. Consider a space game in which world entities can spawn at arbitrary locations, or where space ships can move around freely until they leave the Octree's root cell. To handle such situations an Octree must be expanded and shrunk dynamically as game entities spawn, disappear or move.

Octrees can be expanded by allocating a new Octree root node with seven new child nodes and the 8th child node being the old root node. It's crucial to expand the Octree into the direction of the outlying object. Therefore, the center of the new root node must be chosen in such a way, that the outlying object falls into on of them, or at least the distance between the outlying object and the new Octree root node decreases. This operation is repeated recursively until the outlying object finally falls into the Octree's root cell. As the Octree's extent grows exponentially (it doubles each tree level) any reasonably far away object will be enclosed after a few expansion steps.
A reverse operation can be applied when objects are removed from the Octree. If seven out of eight root node children are empty, all seven children can be deleted and the remaining child becomes the Octree's new root node.

Creating and deleting nodes at the top of hashed Octrees is very costly, because the locational code of all nodes below the new root node gets 3 bits longer and must be updated. Consequently, the hash map must be updated as well. If expanding/shrinking are rare operations it might be still worth using hashed Octrees. Though, usually, pointer-based implementations perform much better. For more information read [part two](http://geidav.wordpress.com/2014/08/18/advanced-octrees-2-node-representations/).