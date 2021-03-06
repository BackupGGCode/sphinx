.. highlight:: rest

Extension API
=============

.. currentmodule:: sphinx.application

Each Sphinx extension is a Python module with at least a :func:`setup` function.
This function is called at initialization time with one argument, the
application object representing the Sphinx process.  This application object has
the following public API:

.. method:: Sphinx.add_builder(builder)

   Register a new builder.  *builder* must be a class that inherits from
   :class:`~sphinx.builder.Builder`.

.. method:: Sphinx.add_config_value(name, default, rebuild_env)

   Register a configuration value.  This is necessary for Sphinx to recognize
   new values and set default values accordingly.  The *name* should be prefixed
   with the extension name, to avoid clashes.  The *default* value can be any
   Python object.  The boolean value *rebuild_env* must be ``True`` if a change
   in the setting only takes effect when a document is parsed -- this means that
   the whole environment must be rebuilt.

   .. versionchanged:: 0.4
      If the *default* value is a callable, it will be called with the config
      object as its argument in order to get the default value.  This can be
      used to implement config values whose default depends on other values.

.. method:: Sphinx.add_event(name)

   Register an event called *name*.

.. method:: Sphinx.add_node(node, **kwds)

   Register a Docutils node class.  This is necessary for Docutils internals.
   It may also be used in the future to validate nodes in the parsed documents.

   Node visitor functions for the Sphinx HTML, LaTeX and text writers can be
   given as keyword arguments: the keyword must be one or more of ``'html'``,
   ``'latex'``, ``'text'``, the value a 2-tuple of ``(visit, depart)`` methods.
   ``depart`` can be ``None`` if the ``visit`` function raises
   :exc:`docutils.nodes.SkipNode`.  Example::

      class math(docutils.nodes.Element)
   
      def visit_math_html(self, node):
          self.body.append(self.starttag(node, 'math'))
      def depart_math_html(self, node):
          self.body.append('</math>')
   
      app.add_node(math, html=(visit_math_html, depart_math_html))

   Obviously, translators for which you don't specify visitor methods will choke
   on the node when encountered in a document to translate.

   .. versionchanged:: 0.5
      Added the support for keyword arguments giving visit functions.

.. method:: Sphinx.add_directive(name, cls, content, arguments, **options)

   Register a Docutils directive.  *name* must be the prospective directive
   name, *func* the directive function for details about the signature and
   return value.  *content*, *arguments* and *options* are set as attributes on
   the function and determine whether the directive has content, arguments and
   options, respectively.  For their exact meaning, please consult the Docutils
   documentation.

   .. XXX once we target docutils 0.5, update this

.. method:: Sphinx.add_role(name, role)

   Register a Docutils role.  *name* must be the role name that occurs in the
   source, *role* the role function (see the `Docutils documentation
   <http://docutils.sourceforge.net/docs/howto/rst-roles.html>`_ on details).

.. method:: Sphinx.add_description_unit(directivename, rolename, indextemplate='', parse_node=None, ref_nodeclass=None)

   This method is a very convenient way to add a new type of information that
   can be cross-referenced.  It will do this:

   * Create a new directive (called *directivename*) for a :term:`description
     unit`.  It will automatically add index entries if *indextemplate* is
     nonempty; if given, it must contain exactly one instance of ``%s``.  See
     the example below for how the template will be interpreted.
   * Create a new role (called *rolename*) to cross-reference to these
     description units.
   * If you provide *parse_node*, it must be a function that takes a string and
     a docutils node, and it must populate the node with children parsed from
     the string.  It must then return the name of the item to be used in
     cross-referencing and index entries.  See the :file:`ext.py` file in the
     source for this documentation for an example.

   For example, if you have this call in a custom Sphinx extension::

      app.add_description_unit('directive', 'dir', 'pair: %s; directive')

   you can use this markup in your documents::

      .. directive:: function

         Document a function.

      <...>

      See also the :dir:`function` directive.

   For the directive, an index entry will be generated as if you had prepended ::

      .. index:: pair: function; directive

   The reference node will be of class ``literal`` (so it will be rendered in a
   proportional font, as appropriate for code) unless you give the *ref_nodeclass*
   argument, which must be a docutils node class (most useful are
   ``docutils.nodes.emphasis`` or ``docutils.nodes.strong`` -- you can also use
   ``docutils.nodes.generated`` if you want no further text decoration).

   For the role content, you have the same options as for standard Sphinx roles
   (see :ref:`xref-syntax`).

.. method:: Sphinx.add_crossref_type(directivename, rolename, indextemplate='', ref_nodeclass=None)

   This method is very similar to :meth:`add_description_unit` except that the
   directive it generates must be empty, and will produce no output.

   That means that you can add semantic targets to your sources, and refer to
   them using custom roles instead of generic ones (like :role:`ref`).  Example
   call::

      app.add_crossref_type('topic', 'topic', 'single: %s', docutils.nodes.emphasis)

   Example usage::

      .. topic:: application API

      The application API
      -------------------

      <...>

      See also :topic:`this section <application API>`.

   (Of course, the element following the ``topic`` directive needn't be a
   section.)

.. method:: Sphinx.add_transform(transform)

   Add the standard docutils :class:`Transform` subclass *transform* to the list
   of transforms that are applied after Sphinx parses a reST document.

.. method:: Sphinx.add_javascript(filename)

   Add *filename* to the list of JavaScript files that the default HTML template
   will include.  The filename must be relative to the HTML static path, see
   :confval:`the docs for the config value <html_static_path>`.

   .. versionadded:: 0.5
   
.. method:: Sphinx.connect(event, callback)

   Register *callback* to be called when *event* is emitted.  For details on
   available core events and the arguments of callback functions, please see
   :ref:`events`.

   The method returns a "listener ID" that can be used as an argument to
   :meth:`disconnect`.

.. method:: Sphinx.disconnect(listener_id)

   Unregister callback *listener_id*.

.. method:: Sphinx.emit(event, *arguments)

   Emit *event* and pass *arguments* to the callback functions.  Return the
   return values of all callbacks as a list.  Do not emit core Sphinx events
   in extensions!

.. method:: Sphinx.emit_firstresult(event, *arguments)

   Emit *event* and pass *arguments* to the callback functions.  Return the
   result of the first callback that doesn't return ``None`` (and don't call
   the rest of the callbacks).

   .. versionadded:: 0.5


.. exception:: ExtensionError

   All these functions raise this exception if something went wrong with the
   extension API.

Examples of using the Sphinx extension API can be seen in the :mod:`sphinx.ext`
package.


.. _events:

Sphinx core events
------------------

These events are known to the core.  The arguments shown are given to the
registered event handlers.

.. event:: builder-inited (app)

   Emitted when the builder object has been created.  It is available as
   ``app.builder``.

.. event:: doctree-read (app, doctree)

   Emitted when a doctree has been parsed and read by the environment, and is
   about to be pickled.  The *doctree* can be modified in-place.

.. event:: missing-reference (app, env, node, contnode)

   Emitted when a cross-reference to a Python module or object cannot be
   resolved.  If the event handler can resolve the reference, it should return a
   new docutils node to be inserted in the document tree in place of the node
   *node*.  Usually this node is a :class:`reference` node containing *contnode*
   as a child.

   :param env: The build environment (``app.builder.env``).
   :param node: The :class:`pending_xref` node to be resolved.  Its attributes
      ``reftype``, ``reftarget``, ``modname`` and ``classname`` attributes
      determine the type and target of the reference.
   :param contnode: The node that carries the text and formatting inside the
      future reference and should be a child of the returned reference node.

   .. versionadded:: 0.5
   
.. event:: doctree-resolved (app, doctree, docname)

   Emitted when a doctree has been "resolved" by the environment, that is, all
   references have been resolved and TOCs have been inserted.

.. event:: env-updated (app, env)

   Emitted when the :meth:`update` method of the build environment has
   completed, that is, the environment and all doctrees are now up-to-date.

   .. versionadded:: 0.5
   
.. event:: page-context (app, pagename, templatename, context, doctree)

   Emitted when the HTML builder has created a context dictionary to render a
   template with -- this can be used to add custom elements to the context.

   The *pagename* argument is the canonical name of the page being rendered,
   that is, without ``.html`` suffix and using slashes as path separators.  The
   *templatename* is the name of the template to render, this will be
   ``'page.html'`` for all pages from reST documents.

   The *context* argument is a dictionary of values that are given to the
   template engine to render the page and can be modified to include custom
   values.  Keys must be strings.

   The *doctree* argument will be a doctree when the page is created from a reST
   documents; it will be ``None`` when the page is created from an HTML template
   alone.

   .. versionadded:: 0.4

.. event:: build-finished (app, exception)

   Emitted when a build has finished, before Sphinx exits, usually used for
   cleanup.  This event is emitted even when the build process raised an
   exception, given as the *exception* argument.  The exception is reraised in
   the application after the event handlers have run.  If the build process
   raised no exception, *exception* will be ``None``.  This allows to customize
   cleanup actions depending on the exception status.

   .. versionadded:: 0.5
   

.. _template-bridge:

The template bridge
-------------------

.. autoclass:: TemplateBridge
   :members:
