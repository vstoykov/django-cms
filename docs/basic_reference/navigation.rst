##########
Navigation
##########

.. highlight:: html+django

There are four template tags for use in the templates that are connected to the
menu:

* :ttag:`show_menu`
* :ttag:`show_menu_below_id`
* :ttag:`show_sub_menu`
* :ttag:`show_breadcrumb`

To use any of these templatetags, you need to have ``{% load menu_tags %}`` in
your template before the line on which you call the templatetag.

.. note::

    Please note that menus live in the :mod:`menus` application, which though
    tightly coupled to the :mod:`cms` application exists independently of it.
    Menus are usable by any application, not just by django CMS.

*********
show_menu
*********

:ttag:`{% show_menu %} <show_menu>` renders the navigation of the current page.
You can overwrite the appearance and the HTML if you add a ``menu/menu.html``
template to your project or edit the one provided with django CMS.
``show_menu`` takes four optional parameters: ``start_level``, ``end_level``,
``extra_inactive``, and ``extra_active``.

The first two parameters, ``start_level`` (default=0) and ``end_level``
(default=100) specify from which level the navigation should be rendered and at
which level it should stop. If you have home as a root node (i.e. level 0) and
don't want to display the root node(s), set ``start_level`` to 1.

The third parameter, ``extra_inactive`` (default=0), specifies how many levels
of navigation should be displayed if a node is not a direct ancestor or
descendant of the current active node.

The fourth parameter, ``extra_active`` (default=100), specifies how
many levels of descendants of the currently active node should be displayed.

You can supply a ``template`` parameter to the tag.

Some Examples
=============

Complete navigation (as a nested list)::

    {% load menu_tags %}
    <ul>
        {% show_menu 0 100 100 100 %}
    </ul>

Navigation with active tree (as a nested list)::

    <ul>
        {% show_menu 0 100 0 100 %}
    </ul>

Navigation with only one active extra level::

    <ul>
        {% show_menu 0 100 0 1 %}
    </ul>

Level 1 navigation (as a nested list)::

    <ul>
        {% show_menu 1 %}
    </ul>

Navigation with a custom template::

    {% show_menu 0 100 100 100 "myapp/menu.html" %}


******************
show_menu_below_id
******************

If you have set an id in the advanced settings of a page, you can display the
submenu of this page with a template tag. For example, we have a page called
meta that is not displayed in the navigation and that has the id "meta"::

    <ul>
        {% show_menu_below_id "meta" %}
    </ul>

You can give it the same optional parameters as :ttag:`show_menu`::

    <ul>
        {% show_menu_below_id "meta" 0 100 100 100 "myapp/menu.html" %}
    </ul>

Unlike :ttag:`show_menu`, however, soft roots will not affect the menu when 
using :ttag:`show_menu_below_id`.

*************
show_sub_menu
*************

Displays the sub menu of the current page (as a nested list).

The first argument, ``levels`` (default=100), specifies how many levels deep
the sub menu should be displayed.

The second argument, ``root_level`` (default=None), specifies at what level, if
any, the menu should have its root. For example, if root_level is 0 the menu
will start at that level regardless of what level the current page is on.

The third argument, ``nephews`` (default=100), specifies how many levels of
nephews (children of siblings) are shown.

Fourth argument, ``template`` (default=menu/sub_menu.html), is the template
used by the tag; if you want to use a different template you **must** supply
default values for ``root_level`` and ``nephews``.

Examples::

    <ul>
        {% show_sub_menu 1 %}
    </ul>

Rooted at level 0::

    <ul>
        {% show_sub_menu 1 0 %}
    </ul>

Or with a custom template::

    <ul>
        {% show_sub_menu 1 None 100 "myapp/submenu.html" %}
    </ul>


***************
show_breadcrumb
***************

Show the breadcrumb navigation of the current page. The template for the HTML
can be found at ``menu/breadcrumb.html``.::

    {% show_breadcrumb %}

Or with a custom template and only display level 2 or higher::

    {% show_breadcrumb 2 "myapp/breadcrumb.html" %}

If the current URL is not handled by the CMS or you are working in a navigation
extender, you may need to provide your own breadcrumb via the template.
This is mostly needed for pages like login, logout and third-party apps.


.. _extending_the_menu:


*******************************************
Properties of Navigation Nodes in templates
*******************************************
::

    {{ node.is_leaf_node }}

Is it the last in the tree? If true it doesn't have any children.
(This normally comes from mptt.)
::

    {{ node.level }}

The level of the node. Starts at 0.
::

    {{ node.menu_level }}

The level of the node from the root node of the menu. Starts at 0.
If your menu starts at level 1 or you have a "soft root" (described
in the next section) the first node would still have 0 as its ``menu_level``.
::

    {{ node.get_absolute_url }}

The absolute URL of the node, without any protocol, domain or port.
::

    {{ node.title }}

The title in the current language of the node.
::

    {{ node.selected }}

If true this node is the current one selected/active at this URL.
::

    {{ node.ancestor }}

If true this node is an ancestor of the current selected node.
::

    {{ node.sibling }}

If true this node is a sibling of the current selected node.
::

    {{ node.descendant }}

If true this node is a descendant of the current selected node.
::

    {{ node.soft_root }}

If true this node is a "soft root".

**********
Soft Roots
**********

What Soft Roots do
==================

A *soft root* is a page that acts as the root for a menu
navigation tree.

Typically, this will be a page that is the root of a significant
new section on your site.

When the *soft root* feature is enabled, the navigation menu
for any page will start at the nearest *soft root*, rather than
at the real root of the site's page hierarchy.

This feature is useful when your site has deep page hierarchies
(and therefore multiple levels in its navigation trees). In such
a case, you usually don't want to present site visitors with deep
menus of nested items.

For example, you're on the page "Introduction to Bleeding", so the menu might look like this:

* School of Medicine
    * Medical Education
    * Departments
        * Department of Lorem Ipsum
        * Department of Donec Imperdiet
        * Department of Cras Eros
        * Department of Mediaeval Surgery
            * Theory
            * Cures
                * Bleeding
                    * Introduction to Bleeding <this is the current page>
                    * Bleeding - the scientific evidence
                    * Cleaning up the mess
                * Cupping
                * Leaches
                * Maggots
            * Techniques
            * Instruments
        * Department of Curabitur a Purus
        * Department of Sed Accumsan
        * Department of Etiam
    * Research
    * Administration
    * Contact us
    * Impressum

which is frankly overwhelming.

By making "Department of Mediaeval Surgery" a *soft root*, the
menu becomes much more manageable:

* Department of Mediaeval Surgery
    * Theory
    * Cures
        * Bleeding
            * Introduction to Bleeding <current page>
            * Bleeding - the scientific evidence
            * Cleaning up the mess
        * Cupping
        * Leaches
        * Maggots
    * Techniques
    * Instruments

Using Soft Roots
================

Mark a page as *soft root* in the 'Advanced Settings' window of a page.

******************************
Modifying & Extending the menu
******************************

Please refer to the :doc:`../../extending_cms/app_integration` documentation
