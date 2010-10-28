In detail: ``plone.app.z3cform``
================================


Inline KSS validation
---------------------

This only applies if `plone.app.kss`_ is installed.


WYSIWYG widget
--------------

.. code-block:: python

    from z3c.form import fields, form
    from z3c.form.interfaces import INPUT_MODE
    from plone.app.z3cform.wysiwyg.widget import WysiwygFieldWidget
    
    class PersonForm(form.Form):
        fields = field.Fields(IPerson)
        fields['bio'].widgetFactory[INPUT_MODE] = WysiwygFieldWidget


Query select widget
-------------------

.. code-block:: python

    from zoep.schema import Set, Choice
    from plone.app.z3cform.queryselect import ArchetypesContentSourceBinder

    class IRelated(Interface):

        related = Set(
            title=u"Related",
            description=u"Related content",
            value_type=schema.Choice(source=ArchetypesContentSourceBinder()))


.. _`plone.app.kss`: http://pypi.python.org/pypi/plone.app.kss
