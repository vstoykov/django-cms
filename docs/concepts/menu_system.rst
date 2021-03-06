#########################
How the menu system works
#########################

**************
Basic concepts
**************

Registration
============

The menu system isn't monolithic. Rather, it is composed of numerous active parts, many of which can operate independently of each other.

What they operate on is a list of menu nodes, that gets passed around the menu system, until it emerges at the other end.

The main active parts of the menu system are menu *generators* and *modifiers*.

Some of these parts are supplied with the menus application. Some come from other applications (from the cms application in django CMS, for example, or some other application entirely).

All these active parts need to be registered within the menu system.

Then, when the time comes to build a menu, the system will ask all the registered menu generators and modifiers to get to work on it.

Generators and Modifiers
======================== 

Menu generators and modifiers are classes.

Generators
----------

To add nodes to a menu a generator is required. 

There is one in cms for example, which examines the Pages in the database and adds them as nodes.

These classses are subclasses of :py:class:`menus.base.Menu`. The one in cms is :py:class:`cms.menu.CMSMenu`.

In order to use a generator, its :py:meth:`get_nodes` method must be called.

Modifiers
---------

A modifier examines the nodes that have been assembled, and modifies them according to its requirements (adding or removing them, or manipulating their attributes, as it sees fit).

An important one in cms (:py:class:`cms.menu.SoftRootCutter`) removes the nodes that are no longer required when a soft root is encountered.

These classes are subclasses of :py:class:`menus.base.Modifier`. Examples are :py:class:`cms.menu.NavExtender` and :py:class:`cms.menu.SoftRootCutter`.

In order to use a modifier, its :py:meth:`modify()` method must be called.

Note that each Modifier's :py:meth:`modify()` method can be called *twice*, before and after the menu has been trimmed.

For example when using the {% show_menu %} templatetag, it's called: 

* first, by :py:meth:`menus.menu_pool.MenuPool.get_nodes()`, with the argument post_cut = False
* later, by the templatetag, with the argument post_cut = True

This corresponds to the state of the nodes list before and after :py:meth:`menus.templatetags.menu_tags.cut_levels()`, which removes nodes from the menu according to the arguments provided by the templatetag.

This is because some modification might be required on *all* nodes, and some might only be required on the subset of nodes left after cutting.

Nodes
=====

Nodes are assembled in a tree. Each node is an instance of the :py:class:`menus.base.NavigationNode` class.

A NavigationNode has attributes such as URL, title, parent and children - as one would expect in a navigation tree.

.. warning::
    You can't assume that a :py:class:`menus.base.NavigationNode` represents a django CMS Page. Firstly, some nodes may
    represent objects from other applications. Secondly, you can't expect to be able to access Page objects via
    NavigationNodes.

***********************
How does all this work?
***********************

Tracing the logic of the menu system
====================================

Let's look at an example using the {% show_menu %} templatetag. It will be different for other templatetags, and your applications might have their own menu classes. But this should help explain what's going on and what the menu system is doing.

One thing to understand is that the system passes around a list of ``nodes``, doing various things to it. 

Many of the methods below pass this list of nodes to the ones it calls, and return them to the ones that they were in turn called by.
                 
Don't forget that show_menu recurses - so it will do *all* of the below for *each level* in the menu.

* ``{% show_menu %}`` - the templatetag in the template
    * :py:meth:`menus.templatetags.menu_tags.ShowMenu.get_context()` 
        * :py:meth:`menus.menu_pool.MenuPool.get_nodes()`
            * :py:meth:`menus.menu_pool.MenuPool.discover_menus()` checks every application's menu.py, and registers:
 				* Menu classes, placing them in the self.menus dict
				* Modifier classes, placing them in the self.modifiers list
            * :py:meth:`menus.menu_pool.MenuPool._build_nodes()` 
                * checks the cache to see if it should return cached nodes
                * loops over the Menus in self.menus (note: by default the only generator is :py:class:`cms.menu.CMSMenu`); for each:
				    * call its :py:meth:`get_nodes()` - the menu generator
				    * :py:meth:`menus.menu_pool._build_nodes_inner_for_one_menu()`
				    * adds all nodes into a big list
            * :py:meth:`menus.menu_pool.MenuPool.apply_modifiers()` 
                * :py:meth:`menus.menu_pool.MenuPool._mark_selected()`
                * loops over each node, comparing its URL with the request.path_info, and marks the best match as ``selected``
                * loops over the Modifiers in self.modifiers calling each one's :py:meth:`modify(post_cut=False)`. The default Modifiers are:
                    * :py:class:`cms.menu.NavExtender`
                    * :py:class:`cms.menu.SoftRootCutter` removes all nodes below the appropriate soft root 
                    * :py:class:`menus.modifiers.Marker` loops over all nodes; finds selected, marks its ancestors, siblings and children
                    * :py:class:`menus.modifiers.AuthVisibility` removes nodes that require authorisation to see
                    * :py:class:`menus.modifiers.Level` loops over all nodes; for each one that is a root node (level = 0) passes it to:
                        * :py:meth:`menus.modifiers.Level.mark_levels()` recurses over a node's descendants marking their levels
        * we're now back in :py:meth:`menus.templatetags.menu_tags.ShowMenu.get_context()` again
        * if we have been provided a root_id, get rid of any nodes other than its descendants
        * :py:meth:`menus.templatetags.menu_tags.cut_levels()` removes nodes from the menu according to the arguments provided by the templatetag
        * :py:meth:`menu_pool.MenuPool.apply_modifiers(post_cut = True)` loops over all the Modifiers again
            * :py:class:`cms.menu.NavExtender`
            * :py:class:`cms.menu.SoftRootCutter` 
            * :py:class:`menus.modifiers.Marker`
            * :py:class:`menus.modifiers.AuthVisibility` 
            * :py:class:`menus.modifiers.Level`:
                * :py:meth:`menus.modifiers.Level.mark_levels()` 
        * return the nodes to the context in the variable ``children``
