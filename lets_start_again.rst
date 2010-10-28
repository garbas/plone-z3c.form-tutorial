Now lets start again
====================

``setup.py``:

.. code-block:: python

    setup(
        ...
        install_requires = [
            ...
            'plone.app.z3cform',
            'plone.directives.form',
            ...
        ]
    )

``configure.zcml``

.. code-block:: xml

    <include package="plone.directives.form" file="meta.zcml" />
    <include package="plone.directives.form" />
    <include package="plone.app.z3cform" />
    <grok:grok package="." />

    or

    <includeDependencies package="." />
    <grok:grok package="." />

    ...

``profiles/default/metadata.xml``

.. code-block:: xml

    <metadata>
        <version>1</version>
        <dependencies>
            <dependency>profile-plone.app.z3cform:default</dependency>
            ...
        </dependencies>
    </metadata>


``buildout.cfg``

.. code-block:: text

    [buildout]
    extends =
        ...
        http://good-py.appspot.com/release/plone.autoform/1.0b2
