
This specification establishes a convention for creating a style of
reusable JavaScript modules and systems that will load and link those
modules.

*   Modules are singletons.
*   Modules have an implicitly isolated lexical scope.
*   Loading may be eager in asynchronous loaders.
*   Execution must be lazy.
*   With some care, modules may have cyclic dependencies.
*   Systems of modules may be isolated but still share the same context.
*   Modules may be used both in browsers and servers.


Guarantees Made by Module Writers
=================================

No requirement in this section implies that a module system must enforce
that requirement.  These are merely requirements that a module must
satisfy in order to function in any module system.

1.  A module must be encoded in UTF-8.
1.  A module must only read and write variables in its lexical scope.
1.  A module must only assume that its lexical scope contains the free
    variables ``require``, ``module``, ``exports``, and ``define``
    beyond those defined by JavaScript.  *What constitutes JavaScript is
    beyond the scope of this specification and may vary with practical
    limits on a module's portability.*
1.  A module must only depend on specified behavior of the values
    provided to its lexical scope, or attempt to upgrade existing values
    to specified behavior, accounting for the possibility that such may
    not be possible if said values are immutable.
1.  All calls to ``require`` must be by name from lexical scope.
1.  All calls to ``require`` must be given a single string literal as
    the argument, or the value of ``require.main`` if it is defined.
    *This rule exists to help asynchronous loader implementations
    guarantee that all modules are loaded before they're required.
    String literals are discoverable before execution by scanning the
    text.
    The module identified by ``require.main`` is provably already
    loaded, so it is a reasonable exception.  The scanning for
    ``require`` calls can occur at different times for production and
    development to meet the performance and the page-refresh-only
    requirements of the respective modes.
    In production, the ``require`` calls can occur in a build step and
    be used to populate the here unspecified arguments of ``define``
    with a dependencies array, whereas the ``require`` calls may be
    scanned by calling ``toString`` on the ``define`` ``callback`` by a
    client-side loader in development.*
1.  For interoperability with loaders that support the following cases,
    modules must call ``define`` to receive ``require``, ``exports``,
    ``module``, or establish their exports.
    *   When modules must be **debugged when deployed** on a different
        domain than the origin page (perhaps a CDN) in browsers that do
        not support cross-origin HTTP request headers (CORS) or the
        developer cannot configure the deployment server to provide CORS
        headers.  JavaScript that is deployed in a debuggable form will
        suffer performance penalties since it would preclude
        minification and bundling.
    *   modules must be debugged in browsers that do not support
        ``//@sourceURL`` comments **and** (a module build step **or**
        module server that add ``define`` calls are not acceptable).
1.  ``define`` may be called with either an object or a function
    ("callback") as its final argument.
    1.  Within a ``define`` callback, the variables ``require``,
        ``exports``, and ``module`` must use the corresponding values
        provided as respective arguments to the callback instead of
        those received from lexical scope.
    1.  The first argument, ``require``, to a ``define`` callback must
        be named ``require``.
        *Chosing an alternate name would make it impossible for
        debugging client-side browser module loader to discover a
        module's dependencies in the context of this specification.*
1.  In the lexical scope of a module, the name ``define`` is reserved
    for the module system.
    *This permits tools that optimize modules for deployment to find the
    ``define`` call and inject additional initial arguments to relieve
    responsibilities from the client-side browser module loader.*
    *Presently, for RequireJS ``define`` must be called with a function
    expression as its argument, but I hope this will be relaxed.*
    1.  A call of ``define`` must only exist once in a module's text.
    1.  ``define`` must only be called once while executing the module.
    1.  The value of ``define`` when called must be the same value as
        provided in the lexical scope of the module.


Guarantees Made by Module Interpreters
======================================

1.  The top scope of a module must not be shared with any other module.
1.  A ``require`` function must exist in a function's lexical scope.
    1.  The ``require`` function must accept a module identifier as its
        first and only argument.
    1.  ``require`` must return the same value as ``module.exports`` in
        the identified module, or must throw an exception while trying.
    1.  ``require`` may have a ``main`` property.
        1.  The ``require.main`` property may be read-only and
            non-configurable.
        1.  The ``require.main`` property must be the same value as
            ``module`` in the lexical scope of the first module that
            began executing in the current system of modules.
        1.  The ``require.main`` property must be the same value in
            every module.
    1.  ``require`` must have a ``resolve`` function.
        1.  ``resolve`` accepts a module identifier as its first and only
            argument
        1.  ``resolve`` must return the resolved module identifier
            corresponding to the given module identifier relative to
            this module's resolved module identifier.
    1.  ``require`` must have an ``async`` function.
        1.  ``async`` must accept an identifier or an array of
            identifiers as its first argument.
            1.  A single identifier is equivalent to an array with a
                single identifier.
        1.  ``async`` accepts an optional ``callback`` function as its
            second argument.
        1.  ``async`` accepts an optional ``errback`` function as its
            third argument.
        1.  ``async`` must call ``require`` for each module identifier
            in order
        1.  ``async`` may wait for each module's transitively required
            modules to asynchronously load before calling ``require``.
        1.  If any ``require`` call throws an exception, or if any
            module cannot be loaded before it is required, ``errback``
            must be called with the ``Error`` as its argument.
        1.  If every ``require`` call returns, ``async`` must call
            ``callback`` with the respective exports for each module
            identifier as its arguments.
1.  An ``exports`` object must exist in a function's lexical scope.
    1.  the ``exports`` object must initially be the same value as
        ``module.exports``.
1.  A ``module`` object must exist in a function's lexical scope.
    1.  The ``module`` object must have an ``id`` property.
        1.  The ``module.id`` property must be a module identifier such
            that ``require(module.id)`` must return the
            ``module.exports`` value when called in this or any module
            in the same system of modules.
        1.  The ``module.id`` property may be read-only and
            non-configurable.
    1.  The ``module`` object must have an ``exports`` property.
        1.  The ``module.exports`` property must initially be an empty
            object.
        1.  The ``module.exports`` property must be writable and
            configurable.
    1.  The ``module`` object may have a ``path`` URL relative to
        ``file:///``
    1.  The ``module`` object may have a ``directory`` URL relative to
        ``file:///``
        1.  The directory must be the directory containing the ``path``.
1.  A ``define`` function must exist in a function's lexical scope.
    1.  ``define`` accepts a function ("callback") or object ("exports")
        as its last argument.
    1.  ``callback`` accepts ``require``, ``exports``, and ``module``.
    1.  ``callback`` must be called with the corresponding values from
        the module's lexical scope.
    1.  If ``callback`` returns a value other than ``undefined``, the
        return value must be assigned to ``module.exports``.
    1.  If ``define`` is called with an "exports" object, the value must
        be assigned to ``module.exports``.


Module Identifiers
==================

1.  A module identifier is a string of "terms" delimited by forward
    slashes.
1.  A term is either:
    1.  any combination of lower-case letters, numbers, and hyphens,
    1.  ``.``, or
    1.  ``..``
1.  Module identifiers should not have file-name extensions like
    ``.js``.
1.  Module identifiers may be "relative" or "resolved".  A module
    identifier is "relative" if the first term is ``.`` or ``..``.
1.  Top-level identifiers are resolved relative to ``""``.
1.  The ``require`` function in each module resolves relative
    identifiers from the corresponding ``module.id``.
1.  To resolve any path of module identifiers,
    1.  An array of terms must be initialized to an empty array.
    1.  For each module identifier in the path of identifiers,
        1.  Pop off the last term in the array, provided one exists.
        1.  For each term in a module identifier in order,
            1.  Take no action if the term is ``"."``.
            1.  Pop a term off the end of the array if the term is
                ``".."``.
            1.  Push the term on the end of the array otherwise.
    1.  The array of terms must be joined with forward slashes, ``"/"``
        to construct the resulting "resolved" identifier.


Unspecified
===========

This specification leaves the following important points of
interoperability unspecified:

1.  Whether modules are stored with a database, file system, or factory
    functions, or are interchangeable with link libraries.
1.  Whether a path is supported by the module loader for resolving
    module identifiers.
1.  Whether other arguments may be provided to ``define`` and how they
    are interpreted.

