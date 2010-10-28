Actions
=======


.. code-block:: python

    import datetime


    @button.buttonAndHandler(u"Register", accessKey=u"r")
    def handleRegister(self, action):
        ...

    @button.buttonAndHandler(u"Special first in month registration",
            condition=lambda form: datetime.datetime.now().day == 1)
    def handleRegisterSpecial(self, action):
        ...

Updating button properties
--------------------------

.. code-block:: python

    class RegisterForm(form.SchemaForm):
        ...

        def updateActions(self):
            super(RegisterForm, self).updateActions()
            self.actions['register'].addClass('context')

