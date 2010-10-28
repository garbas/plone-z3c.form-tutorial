Vocabularies
============


    * ``term``: entry in a vocabulary
    * ``token``: ASCII string which is passed when form is submited
    * ``value``: actuall value which would be stored in object
    * ``title``: representational string (unicode)


Static vocabularies
-------------------

.. code-block:: python

    class IPerson(form.Schema):
        ...

        favorite_colors = schema.Set(
            title=u"Favorite colors",
            value_type=schema.Choice(values=['red', 'green', 'blue', 'yellow'])

        country = schema.Choice(
            title=u"Country",
            values=['Slovenia', 'Spain', 'Portugal', 'France'])


Dynamic vocabularies
--------------------

.. code-block:: python

    from zope.schema.interfaces import IContextSourceBinder
    from zope.schema.vocabulary import SimpleVocabulary


    grok.provider(IContextSourceBinder)
    def countries_list(context):
        terms = []
        for term in ['Slovenia', 'Spain', 'Portugal', 'France']:
            terms.append(SimpleVocabulary.createTerm(term, term, term))
        return SimpleVocabulary(terms)


    class IPerson(form.Schema):
        ...

        country = schema.Choice(
            title=u"Country",
            source=countries_list)

... or ...

.. code-block:: python

    class CountriesList(object):
        grok.implements(IContextSourceBinder)

        def __init__(self, continent):
            self.continent = continent

        def __call__(self, context):
            countries = dict(
                europe=['Slovenia', 'Spain', 'Portugal', 'France'],
                south_america=['Argentina', 'Brasil', 'Chile'], )

            terms = []
            for term in countries[self.continent]:
                terms.append(SimpleVocabulary.createTerm(term, term, term))

            return SimpleVocabulary(terms)


    class IPerson(form.Schema):
        ...

        country = schema.Choice(
            title=u"Country",
            source=CountriesList('south_america'))


Named vocabularies
------------------

.. code-block:: python

    from zope.schema.interfaces import IVocabularyFactory


    class CountriesList(object)
        grok.implements(IVocabularyFactory)

        def __call__(self, context):
            terms = []
            for term in ['Slovenia', 'Spain', 'Portugal', 'France']:
                terms.append(SimpleVocabulary.createTerm(term, term, term))
            return SimpleVocabulary(terms)

    grok.global_utility(CountriesList, name=u"countries_list")

    class IPerson(form.Schema):
        ...

        country = schema.Choice(
            title=u"Country",
            vocabulary=u"countries_list")

VDEX vocabulary
---------------

Imagine you have big vocabularies with a lot of relations. I'm talking +10.000 
terms with +30.000 relations. So this would be perfect use case to use
`collective.vdexvocabulary`_. Also there are other stuff which I didn't found
in other vocabulary packages for Plone/Zope: 

    * i18n support (as it is defined in IMS VDEX)
    * proper order also with unicode charecters (if zope.ucol is installed)
    * easy registration using zcml
    * relations as it specified in IMS VDEX standard


Installation
^^^^^^^^^^^^

In your configure.zcml add

.. code-block:: xml

    <configure
        ...
        xmlns:vdex="http://namespaces.zope.org/vdex"
        ...>

      <include package="collective.vdexvocabulary" file="meta.zcml" />
      <include package="collective.vdexvocabulary" />

And to register a vdex vocabulary simply add line bellow pointing to file
containing vdex vocabulary

.. code-block:: xml

    <configure
        ...
        xmlns:vdex="http://namespaces.zope.org/vdex"
        ...>

      <vdex:vocabulary file="path-to/very-interesting.xml" />

To make registration of vocabularies even easier you can also register 
several vocabularies and just point to directory

.. code-block:: xml

    <configure
        ...
        xmlns:vdex="http://namespaces.zope.org/vdex"
        ...>

      <vdex:vocabulary directory="path-to/my-vdex-vocabularies" />

vdex files in ``path-to/my-vdex-vocabularies`` directory should have ending
``.vdex`` to be recognized by ``vdex:vocabulary`` ZCML directive.


Example VDEX file
^^^^^^^^^^^^^^^^^

Example of car manufacturers list (``car_manufacturers.vdex``).

.. code-block:: xml

    <?xml version="1.0" encoding="utf-8"?>
    <vdex xmlns="http://www.imsglobal.org/xsd/imsvdex_v1p0"
          orderSignificant="false" language="en">
        <vocabIdentifier>your.package.car_manufacturers</vocabIdentifier>
        <term>
            <termIdentifier>ford</termIdentifier>
            <caption>
                <langstring language="en">Ford</langstring>
                <langstring language="es">Una miedra de coche</langstring>
            </caption>
        </term>
        <term>
            <termIdentifier>bmw</termIdentifier>
            <caption>
                <langstring language="en">BMW</langstring>
                <langstring language="es">Be-eMe-uWe, mierda</langstring>
            </caption>
        </term>

        <relationship>
            <sourceTerm>bmw</sourceTerm>
            <targetTerm vocabIdentifier="your.package.car_models">very-special-bmw-model</targetTerm>
            <relationshipType source="http://www.imsglobal.org/vocabularies/iso2788_relations.xml">NT</relationshipType>
        </relationship>

        ...

    </vdex>

List of car models (``car_models.vdex``).

.. code-block:: xml

    <?xml version="1.0" encoding="utf-8"?>
    <vdex xmlns="http://www.imsglobal.org/xsd/imsvdex_v1p0"
          orderSignificant="false" language="en">
        <vocabIdentifier>your.package.car_models</vocabIdentifier>

        <term>
            <termIdentifier>very-special-bmw-model</termIdentifier>
            <caption>
                <langstring language="en">Very special BMW model</langstring>
                <langstring language="es">Un modelo de Be-eMe-uWe</langstring>
            </caption>
        </term>

        <relationship>
            <sourceTerm>very-special-bmw-model</sourceTerm>
            <targetTerm vocabIdentifier="your.package.car_manufacturers">bmw</targetTerm>
            <relationshipType source="http://www.imsglobal.org/vocabularies/iso2788_relations.xml">BT</relationshipType>
        </relationship>

    ...

    </vdex>

How to access relations (from code)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Relations are defined by `ISO2788`_.

To get listing of BMW car models from above VDEX example you have to

.. code-block:: python

    from zope.schema.vocabulary import getVocabularyRegistry

    vr = getVocabularyRegistry()
    car_manufacturers = vr.get(self.context, 'your.package.car_manufacturers')
    car_models = vr.get(self.context, 'your.package.car_models')

    bmw = car_manufacturers.getTerm('bmw')
    bmw_car_models = bmw.related.get('NT', [])


Elephant vocabularies
---------------------

Like elephants don't forget anything, so does not
``collective.elephantvocabulary``. It provides a wrapper around for existing
`zope.schema`_ vocabularies and make them not forget anything.

Examples:

..  code-block:: python

    from collective.elephantvocabulary import wrap_vocabulary
    
    class IPerson(form.Schema):
        ...

        # countries_list -> ['Slovenia', 'Spain', 'Portugal', 'France']

        country1 = schema.Choice(
            title=u"Country",
            vocabulary=wrap_vocabulary(countries_list, hidden_terms=['Slovenia']))
            # [Spain', 'Portugal', 'France']

        country2 = schema.Choice(
            title=u"Country",
            vocabulary=wrap_vocabulary(countries_list,
                                visible_terms=['Slovenia', 'Portugal']))
            # ['Slovenia', 'Portugal']

        country3 = schema.Choice(
            title=u"Country",
            vocabulary=wrap_vocabulary(countries_list, hidden_terms=['Slovenia'],
                                visible_terms=['Slovenia', 'Spain', 'Portugal']))
            # ['Spain', 'Portugal']


        country4 = schema.Choice(
            title=u"Country",
            vocabulary=wrap_vocabulary(u'countries_list',
                            hidden_terms=['Slovenia', 'Spain']))
            # ['Portugal', 'France']
        
        country5 = schema.Choice(
            title=u"Country",
            vocabulary=wrap_vocabulary(u'countries_list',
                hidden_terms_from_registry=u"example.hidden_countries"))


.. _`collective.vdexvocabulary`: http://pypi.python.org/pypi/collective.vdexvocabulary
.. _`zope.schema`: http://pypi.python.org/pypi/zope.schema
.. _`ISO2788`: http://www.imsglobal.org/vocabularies/iso2788_relations.xml
.. _`IMS VDEX`: http://en.wikipedia.org/wiki/IMS_VDEX
