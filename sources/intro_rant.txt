Intro rant
==========

    * How "important" are forms for our website?

        ..
            - They make your site interactive.
            - Good forms are the KEY to a good website.
            - Broken form means broken website.

    * Form frameworks? What do they solve?

        ..
            - Nobody will stop you from writing forms in "pure" HTML.
            - Sometimes this is even better idea then using some fancy big
              form framework.
            - Form frameworks are here to make our job easier at creating them,
              keeping forms consistent over the website, applying changes...

    * History of forms in Plone:

        - `CMFFormController`_: not really a form framework. or at least
            does't do more then half of the stuff you would expect from form
            framework today. depends on having "controller script" which relays
            to different (Zope) "Python Scripts". You could create form TTW.
            Amazigly but it still used in some places in Plone 4.

        - `zope.formlib`_: Library which ships with Zope. Also used a lot in
          Plone's controlpanel (eg. portlets are depending massivly on it).
          It is based on the idea that you create schema (interface) which
          lists fields, constrains, invariants, ... Then special view are used
          to render form properly, widgets, etc.

            ``However, it can be cumbersome to use, especially when it comes
            to creating custom widgets or more dynamic forms.``
        
                *(Martin Aspeli, http://plone/org, Shema-driven forms tutorial)*

        - `z3c.form`_: not so new anymore (initial release: 2007-05-24).
          Inspired by `zope.formlib`_, but allowing more flexibility. The
          thing we are going to talk about today.


.. _`CMFFormController`: http://pypi.python.org/pypi/Products.CMFFormController
.. _`zope.formlib`: http://pypi.python.org/pypi/zope.formlib
.. _`z3c.form`: http://pypi.python.org/pypi/z3c.form
