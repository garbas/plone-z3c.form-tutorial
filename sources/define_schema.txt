Define schema and create simple form
====================================

.. code-block:: python

    from zope import schema
    from plone.directives import form


    class IPerson(form.Schema):

        firstname = schema.TextLine(
            title=u"Firstname",
            required=True)

        lastname = schema.TextLine(
            title=u"Lastname",
            required=True)

        email = schema.TextLine(
            title=u"E-mail")

        address = schema.Text(
            title=u"Address")


.. code-block:: python

    from five import grok
    from z3c.form import button

    from Products.CMFCore.interfaces import ISiteRoot
    from Products.statusmessages.interfaces import IStatusMessage


    class RegisterForm(form.SchemaForm):
        grok.name('register')
        grok.require('zope2.View')
        grok.context(ISiteRoot)

        schema = IPerson
        ignoreContext = True

        label = u"Register"
        description = u"After you will receive email to confirm registrations."

        def update(self):
            self.request.ser('disable_border', True)
            super(RegisterForm, self).update()

        @button.buttonAndHandler(u"Register")
        def handleRegister(self, action):
            data, errors = self.extractData()
            if errors:
                self.status = self.formErorrsMessage
                return

            self.context['Members'].invokeFactory('Person', **data)

            IStatusMessage(self.request).addStatusMessage(
                "Registered! Soon you will receive email.",
                'info')

            self.request.response.redirect(self.context.absolute_url())
