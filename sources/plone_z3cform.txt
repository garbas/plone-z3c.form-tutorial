In detail: ``plone.z3cform``
============================


Layout wrappers
---------------

    * define template on form class

        .. code-block:: python

            ...
            from Products.Five.browser.pagetemplatefile import ViewPageTemplateFile

            class PersonForm(form.Form):

                fields = field.Fields(IMyformSchema)
                template = ViewPageTemplateFile('mytemplate.pt')

                ...

    * via ``zope.pagetemplate.interfaces.IPageTemplate`` adapting ``(view, request)``

        by default we have this layout wrappers:

        .. code-block:: python

            import z3c.form.interfaces
            import plone.z3cform.interfaces

            from plone.z3cform.templates import ZopeTwoFormTemplateFactory
            from plone.z3cform.templates import FormTemplateFactory

            path = lambda p: os.path.join(os.path.dirname(plone.z3cform.__file__), 'templates', p)

            layout_factory = ZopeTwoFormTemplateFactory(path('layout.pt'),
                form=plone.z3cform.interfaces.IFormWrapper
            )

            wrapped_form_factory = FormTemplateFactory(path('wrappedform.pt'),
                form=plone.z3cform.interfaces.IWrappedForm,
            )

            standalone_form_factory = ZopeTwoFormTemplateFactory(path('form.pt'),
                form=z3c.form.interfaces.IForm
            )

            subform_factory = FormTemplateFactory(path('subform.pt'),
                form=z3c.form.interfaces.ISubForm
            )

        .. code-block:: xml

            <adapter factory=".templates.layout_factory" />
            <adapter factory=".templates.wrapped_form_factory" />
            <adapter factory=".templates.standalone_form_factory" />
            <adapter factory=".templates.subform_factory" />


``++widget++`` traverser
------------------------

Sometime we have to register a view on a widget, eg.:::

    http://plonesite.com/@@form/++widget++field/@@widget-view

When implementing such a widget (like: plone.formwidget.autocomplete) we need follow
example bellow (why? read more `here`_).

.. code-block:: python 

    from zope.interface import implementsOnly
    from Acquisition import Explicit
    from z3c.form.widget import Widget
    from z3c.form.interfaces import IWidget

    class MyCustomWidget(Widget, Explicit):
        implementsOnly(IWidget)
        ...


Fieldsets and form extenders
----------------------------

.. code-block:: python

    from zope.schema import Text
    from zope.component import adapts
    from plone.z3cform.fieldsets import extensible

    class PersonForm(extensible.ExtensibleForm, form.Form):
        fields = fields.Fields(IPerson)
        ...

    class IBiography(Interface):

        bio = Text(
            title=_(u"Biography"),
            required=True)

    class PersonBioForm(extensible.FormExtender):
        adapts(Interface, Interface, PersonForm) # context, request, form
        
        def __init__(self, context, request, form):
            self.context = context
            self.request = request
            self.form = form

        def update(self):
            self.add(fields.Fields(IBiography, prefix="bio"), group="Biography")
            self.remove('age')
            self.move('lastname', before='firstname')


CRUD (Create, Read, Update, Delete)
-----------------------------------


.. code-block:: python

    from plone.z3cform.crud import crud

    class PersonCRUDForm(crud.CrudForm):
        update_schema = IPerson

        def get_items(self):
            return self.context.objectIds()

        def add(self, data):
            id = self.context.invokeFactory('Person', **data)
            return self.context[id]

        def remove(self, (id, item)):
            self.context.manage_deleteItems([id,])
    
.. _`here`: http://pypi.python.org/pypi/plone.z3cform#a-caveat-about-security
