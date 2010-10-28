In detail: ``plone.autoform``
=============================

As already said `plone.autoform`_ package provides some more functionality
to be passed with interface. Its using ``tag values`` to store hints.

.. code-block:: python

    from plone.autoform.interfaces import OMITTED_KEY
    from plone.autoform.interfaces import WIDGETS_KEY
    from plone.autoform.interfaces import MODES_KEY
    from plone.autoform.interfaces import ORDER_KEY
    from plone.autoform.interfaces import READ_PERMISSIONS_KEY
    from plone.autoform.interfaces import WRITE_PERMISSIONS_KEY
    from plone.autoform.interfaces import FIELDSETS_KEY


In short
--------

    * ``OMITTED_KEY``: list of (interface, fieldName, boolean).

        If ``boolean`` is true then field with ``fieldName`` for form which
        implements ``interface`` will be omitted.

    * ``WIDGETS_KEY``: list of (interface, fieldName, mode).

        Widget for field ``fieldName`` will be rendered in ``mode`` ('hidden',
        'input', 'display') for form which provides ``interface``.

    * ``ORDER_KEY``: list of (fieldName, direction, relativeTo)

        Field ``fieldName`` will be places in ``direction`` ('after',
        'before') relative to field ``relativeTo``.

    * ``READ_PERMISSIONS_KEY``: dict of fieldName -> permission mapping.

        When form is in 'display' mode field ``fieldName`` will be omited
        unless user has ``permission`` for form's context.

    * ``WRITE_PERMISSIONS_KEY``: dict of fieldName -> permission mapping.

        When form is in 'input' mode field ``fieldName`` will be omited
        unless user has ``permission`` for form's context.

    * ``FIELDSETS_KEY``:  list of ``plone.supermodel.model.Fieldset``
      instances.


Example of ``PersonForm`` with ``plone.autoform``
-------------------------------------------------


.. code-block:: python

    from plone.autoform.form import AutoextendibleForm


    IPerson.setTaggedValue(MODES_KEY, [(Interface, 'age', 'hidden')])

    class PersonForm(AutoextendibleForm, form.Form)
        schema = IPerson
        ...

.. _`plone.autoform`: http://pypi.python.org/pypi/plone.autoform

