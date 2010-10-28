And last: ``plone.directives.form``
===================================


`plone.directives.form`_ is a product of effort behind `Dexterity`_, so it
also shares same philosophy.::

    Leave simple things simple and make impossible things possible.

Well I'm not sure if I made this up or I heard it, but if it's not like this
it should be.


Word on grokking
----------------

**Convention over configuration** they say. And they are right. Almost I
would say. In most usecases where you build / extend code you normally want
to write as little as code as possible and get the results quickly.


Then other time you are writing app / products / package / framework which
expects to be extended in some parts. For that I expose configuration in ZCML.


    Thumb rule I follow:

    ``Everything you want to be extendible put in configure.zcml``

While grokking might save you some time, it might give you big problems when
you will face the task of debugging it. But don't be scared ... its all
Python.


.. _`plone.directives.form`: http://pypi.python.org/pypi/plone.directives.form
.. _`Dexterity`: http://pypi.python.org/pypi/plone.app.dexterity

