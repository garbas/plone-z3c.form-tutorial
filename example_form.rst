Example form
============


Shema-driven forms starts with ...
----------------------------------

.. code-block:: python

    from zope.interface import Interface
    from zope.schema import TextLine
    from my.package import MessageFactory as _

    class IPerson(Interface):

        firstname = TextLine(
            title=_(u"Firstname"),
            description=_(u"Please enter your firstname."),
            required=True)

        lastname = TextLine(
            title=_(u"Lastname"),
            description=_(u"Please enter your lastname."),
            required=True)

        age = TextLine(
            title=_(u"Age"),
            description=_(u"Please enter your age."))


Form
----

.. code-block:: python 

    import datetime
    from z3c.form import form, field
    from z3c.form import interfaces

    class PersonForm(form.Form):

        fields = field.Fields(IContact)

        def updateWidgets(self):
            super(ContactForm, self).updateWidgets()
            self.widgets['age'].mode = interfaces.HIDDEN_MODE
        
        @button.buttonAndHandler(u'Cancel')
        def handleCancel(self, action):
            print 'cancel'

        @button.buttonAndHandler(u'Save')
        def handleSave(self, action):
            data, errors = self.extractData()
            if errors:
                return False

            print 'Firstname: ' + data['firstname']
            print 'Lastname: ' + data['lastname']


Hook it up with familiar BrowserView machinery
----------------------------------------------


.. code-block:: python 

    from plone.z3cform.layer import wrap_form

    PersonFormView = wrap_form(PersonForm)


.. code-block:: xml

    <configure
        ...
        xmlns:browser="http://namespaces.zope.org/browser" />

        <browser:page
            name="person"
            for="*"
            class=".person.PersonFormView"
            permission="zope2.View"
            />
    </configure>


*This form can then be viewable via `++view++person` or `@@person`.*
