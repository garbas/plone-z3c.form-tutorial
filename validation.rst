Validation in all its glory
===========================

Field level validation
----------------------

    * Fields already provide their own valiation. eg: zope.schema.Int field
      should be integer, ...

    * ``constrains`` if you need a little more control.

        .. code-block:: python

            from zope.interface import Invalid


            def over_21(value):
                if value >= 21:
                    return True
                raise Invalid("You need to be over 21, to register.")

            class IPerson(form.Schema):
                ... 

                age = schema.Int(
                    title=u"Age",
                    constraint=over_21,
                    required=True)

Widget level validation
-----------------------

    * Validate widget using ``@form.validator``

        .. code-block:: python
            
            @form.validator(field=IPerson['age'])
            def over_21(value):
                if value >= 21:
                    return True
                raise Invalid("You need to be over 21, to register.")
            
        Possible to make validator more specific or generic using 'context',
        'request', 'view', 'field', 'widget' parameter of ``@form.validator``
        decorator.

     * more advanced

        .. code-block:: python

            from z3c.form import validator


            class Over21(validator.SimpleFieldValidator):

                def validate(self, value):
                    # here we could skipt default field validation
                    super(Over21, self).validate(value)

                    if value >= 21:
                        return True
                    raise Invalid("You need to be over 21, to register.")


            validation.WidgetValidatorDiscriminators(Over21,
                        field=IPerson['age'], view=RegisterForm)
            grok.global_adapter(Over21)

        This we use in case:

            - want to skip default validation of field properties ('required',
              'min', ..).

            - need to access context, request, form, field, widget to validate.


Form level validation
---------------------

    * ``invariants``

        .. code-block:: python

            from zope.interface import invariants


            class IPerson(form.Schema):
                ...

                @invariants
                def legal_age(data):
                    if data.city == 'Sevilla' and \
                       data.age >= 18:
                       return True
                    elif data.age >= 21:
                        return True
                    raise Invalid("You need to be over 21, to register.")

    * validation can also happen inside action handler

        .. code-block:: python

            from z3c.form.interfaces import ActionExecutionError
            from z3c.form.interfaces import WidgetActionExecutionError


            class RegisterForm(form.SchemaForm):
                ...

                def handleRegister(self, action):
                    data, errors = self.extractData()

                    if data['age'] > 150:
                        raise WidgetActionExecutionError(
                                    Invalid(u"nobody should register if its older then 100"))
                    
                    try:
                        registration = getToolByName(self.context, 'portal_registration')
                        registration.addMember(data['email'], '12345', properties=data)
                    except ValueError, e:
                        raise ActionExecutionError(
                                Invalid(u"User with this email already exists."))
                    
                    if errors:
                        self.status = self.formErorrsMessage
                        return
                    ...

Customizing error messages
--------------------------

    * replace only text in error message

        .. code-block:: python

            from zope.schema.interfaces import ValidationError

            @form.error_message(field=IPerson['age'], error=ValidationError)
            def to_young_message(value):
                return "By the law we are not allowed to let you in. Sorry." \
                       "See you in %s years." % (21 - value)

        allowed arguments for error_message are:
        
            error, request, widget, field, form, contet
        
        standard errors are (all listed in ``zope.schema.interfaces``):

            RequiredMissing, WrongType, TooBig, TooSmall, TooLong, TooShort,
            InvalidValue, ConstraintNotSatisfied, WrongContainedType,
            NotUnique, InvalidURI, InvalidId, InvalidDottedName

    * output HTML

        .. code-block:: python

            import zope.component
            from z3c.form import error
            from zope.schema.interfaces import TooSmall
        

            TooYoung = error.ErrorViewMessage(
                 u'Too young', error=TooSmall, field=IPerson['age'])

            zope.component.provideAdapter(TooYoung, name='message')

        
