***************************************************************************
Django Forms Deep Dive - Nathan R. Yergler from EventBrite
***************************************************************************

**Presenter**: `Nathan Yergler
<https://us.pycon.org/2012/speaker/profile/362/>`_ (http://yergler.net/)
(`@nyergler <https://twitter.com/#!/nyergler>`_) from `Eventbrite
<http://www.eventbrite.com/>`_

**PyCon 2012 presentation page**: https://us.pycon.org/2012/schedule/presentation/420/

**Slides**: http://yergler.net/2012/pycon-forms/

**Video**: http://pyvideo.org/video/698/django-form-processing-deep-dive


Form Basics
===========

Forms in context
----------------

==========      ===============================================================

----------      ---------------------------------------------------------------
**Views**       Convert request to response
**Forms**       Convert input to Python objects (input is not necessarily HTML)
**Models**      Data and business logic
==========      ===============================================================


Defining Forms
--------------

Forms are composed of fields, which have a widget.

.. code-block:: python

    from django.utils.translation import gettext_lazy as _
    from django import forms

    class ContactForm(forms.Form):

        name = forms.CharField(label=_("Your Name"),
            max_length=255,
            widget=forms.TextInput,
        )

        email = forms.EmailField(label=_("Email address"))

Form API: https://docs.djangoproject.com/en/dev/ref/forms/api/

Instantiating a Form: unbound forms vs. bound forms
---------------------------------------------------

Unbound forms don't have data associated with them, but they can be rendered:

.. code-block:: python

    form = ContactForm()

Bound forms have specific data associated, which can be validated:

.. code-block:: python

    form = ContactForm(data=request.POST, files=request.FILES)


Accessing fields
----------------

Two ways:

* ``form.fields['name']`` returns the ``Field`` object
* ``form['name']`` returns a ``BoundField``


Initial data
------------

.. code-block:: python

    form = ContactForm(
        initial={
            'name': 'First and Last Name',
        }
    )

    >>> form['name'].value()
    'First and Last Name'


Validation
==========

Validating the form
-------------------

* Only bound forms can be validated
* Calling ``form.is_valid()`` triggers validation if needed
* Validated, cleaned data is stored in ``form.cleaned_data``
* Calling ``form.full_clean()`` performs the full cycle

Field validation
----------------

* Three phases for fields: To Python, Validation, and Cleaning
* If validation raises an Error, cleanning is skipped
* Validators are callables that can raise a ``ValidationError``
* Django includes generic ones for some common tasks
* Examples: URL, Min/Max Value, Min/Max Length, Regex, Email, etc.

Field cleaning
--------------

* ``.clean_fieldname()`` method is called after validators
* Input has already been converted to Python objects
* Methods can still raise ``ValidationError``
* Methods *must* return the cleaned value

.clean_email() example
----------------------

.. code-block:: python

    class ContactForm(forms.Form):
        name = forms.CharField(
            label=_("Name"),
            max_length=255,
        )

        email = forms.EmailField(
            label=_("Email address"),
        )

        def clean_email(self):
            if (self.cleaned_data.get('email', '').endswith('hotmail.com')):
                raise ValidationError("Invalid email address.")

            return self.cleaned_data.get('email', '')

Form validation
---------------

* ``.clean()`` performs cross-field validation
  - Example: Check that ``email`` and ``confirm_email`` fields match
* Called even if errors were raised by Fields
* *Must* return the cleaned data dictionary
* ``ValidationError``s raised by ``.clean()`` will be grouped in ``form.non_field_errors()`` by default

.clean() example
----------------

Example of cross-field validation: Check that ``email`` and ``confirm_email`` fields match:

.. code-block:: python

    class ContactForm(forms.Form):
        name = forms.CharField(
            label=_("Name"),
            max_length=255,
        )

        email = forms.EmailField(label=_("Email address"))
        confirm_email = forms.EmailField(label=_("Confirm"))

        def clean(self):
            if (self.cleaned_data.get('email') !=
                self.cleaned_data.get('confirm_email')):

                raise ValidationError("Email addresses do not match.")

            return self.cleaned_data

Initial != Default Data
-----------------------

* Initial data is used as a starting point
* It does not automatically propagate to ``cleaned_data``
* Defaults for non-required fields should be specified when accessing the dict:

.. code-block:: python

    self.cleaned_data.get('name', 'default')

Tracking changes
----------------

* Forms use initial data to track change fields
* ``form.has_changed()``
* ``form.changed_fields``
* Fields can render a hidden input with the initial value, as well:

.. code-block:: python

    >>> changed_date = forms.DateField(show_hidden_initial=True)
    >>> print form['changed_date']
    '<input type="text" name="changed_date" id="id_changed_date" /><input type="hidden" name="initial-changed_date" id="initial-id_changed_date" />'


Testing
=======

Not clear whether they're unit tests or functional tests, etc. but nonetheless useful

Testing forms
-------------

* Remember what Forms are for
* Testing strategies

  - Initial states
  - Field validation
  - Final state of ``cleaned_data``

Unit tests
----------

.. code-block:: python

    import unittest

    class FormTests(unittest.TestCase):
        def test_validation(self):
            form_data = {
                'name': 'X' * 300,
            }

            form = ContactForm(data=form_data)
            self.assertFalse(form.is_valid())

Test Data
---------

Eventbrite just released a library on GitHub called rebar -- https://github.com/eventbrite/rebar

.. code-block:: python

    from rebar.testing import flatten_to_dict

    form_data = flatten_to_dict(ContactForm())
    form_data.update({
            'name': 'X' * 300,
        })
    form = ContactForm(data=form_data)
    assert(not form.is_valid())


Rendering Forms
===============

Idiomatic form usage
--------------------

(Plug for class-based views....)

.. code-block:: python

    from django.views.generic.edit import FormMixin, ProcessFormView

    class ContactView(FormMixin, ProcessFormView):
        form_class = ContactForm
        success_url = '/contact/sent'

        def get_form_kwargs(self):
            return super(ContactView, self).get_form_kwargs()

Form Output
-----------

Three primary "whole-form" output modes:

* ``form.as_p()``
* ``form.as_ul()``
* ``form.as_table()``

.. code-block:: html

    <tr><th><label for="id_name">Name:</label></th>
      <td><input id="id_name" type="text" name="name" maxlength="255" /></td></tr>
    <tr><th><label for="id_email">Email:</label></th>
      <td><input id="id_email" type="text" name="email" maxlength="Email address" /></td></tr>
    <tr><th><label for="id_confirm_email">Confirm email:</label></th>
      <td><input id="id_confirm_email" type="text" name="confirm_email" maxlength="Confirm" /></td></tr>

Controlling form output
-----------------------

.. code-block:: django

    {% for field in form %}
    {{ field.label_tag }}: {{ field }}
    {{ field.errors }}
    {% endfor %}
    {{ field.non_form_errors }}

Additional rendering properties:

* ``field.label``
* ``field.label_tag``
* ``field.html_id``
* ``field.help_text``

Customizing rendering
---------------------

http://yergler.net/2012/pycon-forms/slides/forms/#26

Libraries that make some of this stuff easier:

* `django-crispy-forms <http://django-crispy-forms.readthedocs.org/>`_
* `django-form-utils <http://pypi.python.org/pypi/django-form-utils>`_

You can specify additional attributes for widgets as part of the form definition.

.. code-block:: python

    class ContactForm(forms.Form):
        name = forms.CharField(
            max_length=255,
            widget=forms.Textarea(
                attrs={'class': 'custom'},
            ),
        )

You can also specify form-wide CSS classes to add for error and required states.

.. code-block:: python

    class ContactForm(forms.Form):
        error_css_class = 'error'
        required_css_class = 'required'
        ...

Customizing error messages
--------------------------

Built-in validators have default error messages

.. code-block:: python

    >>> generic = forms.CharField()
    >>> generic.clean('')
    Traceback (most recent call last):
      ...
    ValidationError: [u'This field is required.']

``error_messages`` lets you customize those messages

.. code-block:: python

    >>> name = forms.CharField(
    ...   error_messages={'required': 'Please enter your name'})
    >>> name.clean('')
    Traceback (most recent call last):
      ...
    ValidationError: [u'Please enter your name']

Error Class
-----------

* ``ValidationError`` exceptions raised are wrapped in a class
* This class controls HTML formatting
* By default, ``ErrorList`` is used; outputs as ``<ul>``
* Specify the ``error_class`` kwarg when constructing the form to override

Error Class
-----------

.. code-block:: python

    from django.forms.util import ErrorList

    class ParagraphErrorList(ErrorList):
        def __unicode__(self):
            return self.as_paragraphs()

        def as_paragraphs(self):
            return "<p>%s</p>" % (
                ",".join(e for e in self.errors)
            )

    form = ContactForm(data=form_data, error_class=ParagraphErrorList)

Multiple Forms
--------------

Avoid potential name collisions with prefix:

.. code-block:: python

    contact_form = ContactForm(prefix='contact')

Adds the prefix to HTML name and ID:

.. code-block:: html

    <tr><th><label for="id_contact-name">Name:</label></th>
      <td><input id="id_contact-name" type="text" name="contact-name"
           maxlength="255" /></td></tr>
    <tr><th><label for="id_contact-email">Email:</label></th>
      <td><input id="id_contact-email" type="text" name="contact-email"
           maxlength="Email address" /></td></tr>
    <tr><th><label for="id_contact-confirm_email">Confirm
         email:</label></th>
      <td><input id="id_contact-confirm_email" type="text"
           name="contact-confirm_email" maxlength="Confirm" /></td></tr>


Forms for Models
================

Model Forms
-----------

* ModelForms map a Model to a Form
* Validation includes Model validators by default
* Supports creating and editing instances
* Key differences from forms

  - A field for the primary key (usually ``id``)
  - ``save()`` method
  - ``.instance`` property

Model Form Example
------------------

.. code-block:: python

    from django.db import models
    from django import forms

    class Contact(models.Model):
        name = models.CharField(max_length=100)
        email = models.EmailField()
        notes = models.TextField()

    class ContactForm(forms.ModelForm):
        class Meta:
            model = Contact
            ...

Limiting Fields
---------------

* You don't need to expose all the fields in your form
* You can either specify fields to expose, or fields to exclude

.. code-block:: python

    class ContactForm(forms.ModelForm):
        class Meta:
            model = Contact
            fields = ('name', 'email',)

    class ContactForm(forms.ModelForm):
        class Meta:
            model = Contact
            exclude = ('notes',)

Overriding Fields
-----------------

* Django will generate fields and widgets based on the model
* These can be overridden as well

.. code-block:: python

    class ContactForm(forms.ModelForm):
        name = forms.CharField(widget=forms.TextInput)

        class Meta:
            model = Contact

Instantiating Model Forms
-------------------------

.. code-block:: python

    model_form = ContactForm()

    model_form = ContactForm(
        instance=Contact.objects.get(id=2)
        )

``ModelForm.is_valid()``
------------------------

* ModelForms have an additional method, ``_post_clean()``
* Sets cleaned fields on the Model instance
* Called *regardless* of whether the form is valid

Testing
-------

.. code-block:: python

    class ModelFormTests(unittest.TestCase):
        def test_validation(self):
            form_data = {
                'name': 'Test Name',
            }

            form = ContactForm(data=form_data)
            self.assert_(form.is_valid())
            self.assertEqual(form.instance.name, 'Test Name')

            form.save()

            self.assertEqual(
                Contact.objects.get(id=form.instance.id).name,
                'Test Name'
            )


Form sets
=========

Form sets
---------

* Handles multiple copies of the same form
* Adds a unique prefix to each form:

.. code-block:: python

    form-1-name

* Support for insertion, deletion, and reordering

Defining form sets
------------------

.. code-block:: python

    from django.forms import formsets

    ContactFormSet = formsets.formset_factory(
        ContactForm,
    )

    formset = ContactFormSet(data=request.POST)

Factory kwargs:

* ``can_delete``
* ``extra``
* ``max_num``

Using form sets
---------------

.. code-block:: django

    <form action="" method="POST">
    {% formset %}
    </form>

or more control over output:

.. code-block:: django

    <form action="." method="POST">
    {% formset.management_form %}
    {% for form in formset %}
       {% form %}
    {% endfor %}
    </form>

Management form
---------------

* ``formset.management_form`` provides fields for tracking the member forms

  - ``TOTAL_FORMS``
  - ``INITIAL_FORMS``
  - ``MAX_NUM_FORMS``

* Management form data **must** be present to validate a Form Set

``formset.is_valid()``
----------------------

* Performs validation on each member form
* Calls ``.clean()`` method on the FormSet
* ``formset.clean()`` can be overridden to validate across Forms
* Errors raised are collected in ``formset.non_form_errors``

FormSet.clean()
---------------

.. code-block:: python

    from django.forms import formsets

    class BaseContactFormSet(formsets.BaseFormSet):
        def clean(self):
            names = []
            for form in self.forms:
                if form.cleaned_data.get('name') in names:
                    raise ValidationError()
                names.append(form.cleaned_data.get('name'))

    ContactFormSet = formsets.formset_factory(
        ContactForm,
        formset=BaseContactFormSet
    )

Testing
-------

* FormSets can be tested in the same ways as Forms
* Helpers to generate test form data:

  - ``flatten_to_dict`` works with FormSets just like Forms
  - ``empty_form_data`` takes a FormSet and index, returns a dict of data for an empty form

.. code-block:: python

    from rebar.testing import flatten_to_dict, empty_form_data

    formset = ContactFormSet()
    form_data = flatten_to_dict(formset)
    form_data.update(
        empty_form_data(formset, len(formset))
    )

Model Formsets
--------------

* ModelFormSets:FormSets :: ModelForms:Forms
* ``queryset`` argument specifies intial set of objects
* ``.save()`` returns the list of saved instances
* If ``can_delete`` is ``True``, ``.save()`` also deletes the models flagged for deletion

Advanced & Miscellaneous Detritus
=================================

* Django's i18n/l10n framework supports localized input formats
* For example: 10,00 vs. 10.00

Enable in ``settings.py``:

.. code-block:: python

    USE_L10N = True
    USE_THOUSAND_SEPARATOR = True # optional

Localizing fields example
-------------------------

And then use the ``localize`` kwarg

.. code-block:: python

    >>> from django import forms
    >>> class DateForm(forms.Form):
    ...     pycon_ends = forms.DateField(localize=True)

    >>> DateForm({'pycon_ends': '3/15/2012'}).is_valid()
    True
    >>> DateForm({'pycon_ends': '15/3/2012'}).is_valid()
    False

    >>> from django.utils import translation
    >>> translation.activate('en_GB')
    >>> DateForm({'pycon_ends':'15/3/2012'}).is_valid()
    True

Dynamic forms
-------------

* Declarative syntax is just sugar
* Forms use a metaclass to populate ``form.fields``
* After ``__init__`` finishes, you can manipulate ``form.fields`` without
  impacting other instances

State validators
----------------

* Validation isn't necessarily all or nothing

* State Validators define validation for specific states, on top of basic
  validation

* Your application can take action based on whether the form is valid, or valid for a particular state

State validators example
------------------------

.. code-block:: python

    from django import forms
    from rebar.validators import StateValidator, StateValidatorFormMixin

    class PublishValidator(StateValidator):
        validators = {
            'title': lambda x: bool(x),
         }

    class EventForm(StateValidatorFormMixin, forms.Form):
        state_validators = {
            'publish': PublishValidator,
        }
        title = forms.CharField(required=False)

Using it:

.. code-block:: python

    >>> form = EventForm(data={})
    >>> form.is_valid()
    True
    >>> form.is_valid('publish')
    False
    >>> form.errors('publish')
    {'title': 'This field is required'}

The End
-------

http://yergler.net/2012/pycon-forms
