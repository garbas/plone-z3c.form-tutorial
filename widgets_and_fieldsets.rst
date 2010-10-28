Widgets and Fieldsets
=====================

Custom widget for field
-----------------------

    * `plone.directives.form`_ way:

        .. code-block:: python

            class IPerson(form.Schema):
                ...

                form.widget(address=AddressWidget)
                address = schema.Text(
                    title=u"Address")


    * 'plain' `z3c.form`_ way:

        .. code-block:: python

            class IPerson(Interface):
                ...

            class RegisterForm(z3c.form.form.Form):
                fields = fields.Fields(IPerson)
                fields['address'].widgetFactory = AddressWidget


Form widget hints
-----------------

.. code-block:: python 

    from z3c.form.interfaces import IEditForm
    from plone.directives import form
    from plone.app.z3cform.wysiwyg import WysiwygFieldWidget

    class IMySchema(form.Schema):

        form.fieldset('extra',
                label=u"Extra info",
                fields=['footer', 'dummy']
            )
    
        title = schema.TextLine(
                title=u"Title"
            )

        summary = schema.Text(
                title=u"Summary",
                description=u"Summary of the body",
                readonly=True
            )

        form.widget(body='plone.app.z3cform.wysiwyg.WysiwygFieldWidget')
        form.primary('body')
        body = schema.Text(
                title=u"Body text",
                required=False,
                default=u"Body text goes here"
            )


        form.widget(footer=WysiwygFieldWidget)
        footer = schema.Text(
                title=u"Footer text",
                required=False
            )

        form.omitted('dummy')
        dummy = schema.Text(
                title=u"Dummy"
            )

        form.omitted('edit_only')
        form.no_omit(IEditForm, 'edit_only')
        edit_only = schema.TextLine(
                title = u'Only included on edit forms',
            )

        form.mode(secret='hidden')
        form.mode(IEditForm, secret='input')
        secret = schema.TextLine(
                title=u"Secret",
                default=u"Secret stuff (except on edit forms)"
            )

        form.order_before(not_last='summary')
        not_last = schema.TextLine(
                title=u"Not last",
            )



Update widget settings
----------------------

.. code-block:: python

    class RegisterForm(form.SchemaForm):
        ...

        def updateWidgets(self):
            super(RegisterForm, self).updateWidgets()
            self.widgets['firstname'].size = 15
            self.widgets['lastname'].size = 35


Fieldsets
---------

.. code-block:: python

    class IPerson(form.Schema):
        ...

        form.fieldset(
            'biography',
            u"Biography",
            fields=['bio'],
        )
        
        form.widget(bio=WysiwygFieldWidget)
        bio = schema.Text(
            title=u"Biography")


    

New widget
----------


Widget
^^^^^^

.. code-block:: xml

    <!-- Date widget -->
    <class class=".widget_date.DateWidget">
        <require permission="zope.Public"
                 interface=".interfaces.IDateWidget" />
    </class>

    <!-- use Date by default -->
    <class class="zope.schema._field.Date">
        <implements interface=".interfaces.IDateField" />
    </class>

    <adapter
        factory=".widget_date.DateFieldWidget"
        for=".interfaces.IDateField
             z3c.form.interfaces.IFormLayer" />


.. code-block:: python

    import z3c.form
    import zope.i18n
    import zope.schema
    import zope.interface
    improt zope.component

    class DateWidget(z3c.form.browser.widget.HTMLTextInputWidget,
                     z3c.form.widget.Widget):
        """ Date widget. """

        zope.interface.implementsOnly(IDateWidget)

        ...

        def extract(self, default=z3c.form.interfaces.NOVALUE):
            # get normal input fields
            day = self.request.get(self.name + '-day', default)
            month = self.request.get(self.name + '-month', default)
            year = self.request.get(self.name + '-year', default)

            if not default in (year, month, day):
                return (year, month, day)

            # get a hidden value
            formatter = self.request.locale.dates.getFormatter("date", "short")
            hidden_date = self.request.get(self.name, '')
            try:
                dateobj = formatter.parse(hidden_date)
                return (str(dateobj.year),
                        str(dateobj.month),
                        str(dateobj.day))
            except zope.i18n.format.DateTimeParseError:
                pass

        return default


    @zope.component.adapter(zope.schema.interfaces.IField, z3c.form.interfaces.IFormLayer)
    @zope.interface.implementer(z3c.form.interfaces.IFieldWidget)
    def DateFieldWidget(field, request):
        """IFieldWidget factory for DateWidget."""
        return z3c.form.widget.FieldWidget(field, DateWidget(request))
        

Converters
^^^^^^^^^^

.. code-block:: xml

        <adapter
            factory=".converter.DateDataConverter"
            for="zope.schema.interfaces.IDate
                 .interfaces.IDateWidget" />

.. code-block:: python

    from z3c.form.converter import BaseDataConverter
    from collective.z3cform.datetimewidget.interfaces import DateValidationError

    class DateDataConverter(BaseDataConverter):
    
        def toWidgetValue(self, value):
            if value is self.field.missing_value:
                return ('', '', '')
            return (value.year, value.month, value.day)

        def toFieldValue(self, value):
            for val in value:
                if not val:
                    return self.field.missing_value

            try:
                value = map(int, value)
            except ValueError:
                raise DateValidationError
            try:
                return date(*value)
            except ValueError:
                raise DateValidationError


Template
^^^^^^^^

.. code-block:: xml

        <z3c:widgetTemplate
            mode="input"
            widget=".interfaces.IDateWidget"
            layer="z3c.form.interfaces.IFormLayer"
            template="templates/date_input.pt" />


.. _`plone.directives.form`: http://pypi.python.org/pypi/plone.directives.form
.. _`z3c.form`: http://pypi.python.org/pypi/z3c.form
