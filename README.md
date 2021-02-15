# GOV.UK Frontend WTForms Widgets

[![PyPI version](https://badge.fury.io/py/govuk-frontend-wtf.svg)](https://pypi.org/project/govuk-frontend-wtf/)
![govuk-frontend 3.11.0](https://img.shields.io/badge/govuk--frontend%20version-3.11.0-005EA5?logo=gov.uk&style=flat)
![Build](https://github.com/LandRegistry/govuk-frontend-wtf/workflows/Build/badge.svg)

> [Flask-WTF](https://flask-wtf.readthedocs.io/) is used for building HTML forms, making things like form validation and rendering of errors much easier than having to build this yourself. The [Flask-WTF documentation](https://flask-wtf.readthedocs.io/) covers the standard use cases and you should refer to this.
> 
> In addition to the standard use case however, flask-skeleton-ui includes custom widgets that can be used to render Flask-WTF forms in the GOV.​UK style. These widgets automatically render error messages in the appropriate places as well as showing an error summary at the top of the page in a fully GOV.​UK compliant manner.

- Package: [https://pypi.org/project/govuk-frontend-wtf/](https://pypi.org/project/govuk-frontend-wtf/)
- Demo app: [https://github.com/LandRegistry/govuk-frontend-wtf-demo](https://github.com/LandRegistry/govuk-frontend-wtf-demo)
- Live demo: [https://govuk-frontend-wtf.herokuapp.com/](https://govuk-frontend-wtf.herokuapp.com/)

## How to use

For more detailed examples please refer to the [demo app source code](https://github.com/LandRegistry/govuk-frontend-wtf-demo).

After running `pip install govuk-frontend-wtf`, ensure that you tell Jinja where to load the templates from using the `PackageLoader` and register `WTFormsHelpers` as follows:

```python
from flask import Flask
from govuk_frontend_wtf.main import WTFormsHelpers
from jinja2 import ChoiceLoader, PackageLoader, PrefixLoader

app = Flask(__name__)

app.jinja_loader = ChoiceLoader(
    [
        PackageLoader("app"),
        PrefixLoader(
            {
                "govuk_frontend_jinja": PackageLoader("govuk_frontend_jinja"),
                "govuk_frontend_wtf": PackageLoader("govuk_frontend_wtf"),
            }
        ),
    ]
)

WTFormsHelpers(app)
```

Import and include the relevant widget on each field in your form class (see [table below](#widgets)). Note that `widget=GovTextInput()` is the only difference relative to a standard Flask-WTF form definition.

```python
from flask_wtf import FlaskForm
from govuk_frontend_wtf.wtforms_widgets import GovSubmitInput, GovTextInput
from wtforms import StringField, SubmitField
from wtforms.validators import Email, InputRequired, Length


class ExampleForm(FlaskForm):
    email_address = StringField(
        "Email address",
        widget=GovTextInput(),
        validators=[
            InputRequired(message="Enter an email address"),
            Length(max=256, message="Email address must be 256 characters or fewer"),
            Email(message="Enter an email address in the correct format, like name@example.com"),
        ],
        description="We’ll only use this to send you a receipt",
    )

    submit = SubmitField("Continue", widget=GovSubmitInput())
```

Create a route to serve your form and template.

```python
from flask import redirect, render_template, url_for

from app import app
from app.forms import ExampleForm

@app.route("/")
def index():
    return render_template("index.html")


@app.route("/example-form", methods=["GET", "POST"])
def example():
    form = ExampleForm()
    if form.validate_on_submit():
        return redirect(url_for("index"))
    return render_template("example_form.html", form=form)
```

Finally, in your template make sure to set the page title appropriately if there are any form validation errors. Also include the `govukErrorSummary()` component at the start of the `content` block. Pass parameters in a dictionary to your form field as per the associated [component macro options](https://design-system.service.gov.uk/components/).

```html
{% extends "base.html" %}

{%- from 'govuk_frontend_jinja/components/back-link/macro.html' import govukBackLink -%}
{%- from 'govuk_frontend_jinja/components/error-summary/macro.html' import govukErrorSummary -%}

{% block pageTitle %}{%- if form and form.errors %}Error: {% endif -%}Example form – GOV.UK Frontend WTForms Demo{% endblock %}

{% block beforeContent %}
  {{ govukBackLink({
    'text': "Back",
    'href': url_for('index')
  }) }}
{% endblock %}

{% block content %}
{% if form.errors %}
    {{ govukErrorSummary(wtforms_errors(form)) }}
{% endif %}

<h1 class="govuk-heading-l">Example form</h1>
<div class="govuk-grid-row">
    <div class="govuk-grid-column-two-thirds">
        <form action="" method="post" novalidate>
            {{ form.csrf_token }}
            
            {{ form.email_address(params={
              'hint': {
                'text': form.email_address.description
              },
              'type': 'email',
              'autocomplete': 'email',
              'spellcheck': false
            }) }}
            
            {{ form.submit }}
        </form>
    </div>
</div>
{% endblock %}
```

## Widgets

The available widgets and their corresponding Flask-WTF field types are as follows:

| WTForms Field                                                                                             | GOV.​UK Widget               | Notes |
| --------------------------------------------------------------------------------------------------------- | --------------------------- | ----- |
| [BooleanField](https://wtforms.readthedocs.io/en/2.3.x/fields/#wtforms.fields.BooleanField)               | GovCheckboxInput            |       |
| [DecimalField](https://wtforms.readthedocs.io/en/2.3.x/fields/#wtforms.fields.DecimalField)               | GovTextInput                |       |
| [FileField](https://wtforms.readthedocs.io/en/2.3.x/fields/#wtforms.fields.FileField)                     | GovFileInput                |       |
| [MultipleFileField](https://wtforms.readthedocs.io/en/2.3.x/fields/#wtforms.fields.MultipleFileField)     | GovFileInput(multiple=True) | Note that you need to specify `multiple=True` when invoking the widget in your form class. _Not_ when you render it in the Jinja template. |
| [FloatField](https://wtforms.readthedocs.io/en/2.3.x/fields/#wtforms.fields.FloatField)                   | GovTextInput                |       |
| [IntegerField](https://wtforms.readthedocs.io/en/2.3.x/fields/#wtforms.fields.IntegerField)               | GovTextInput                | Use `params` to specify a `type` if you need to use HTML5 number elements. This will not happen automatically. |
| [PasswordField](https://wtforms.readthedocs.io/en/2.3.x/fields/#wtforms.fields.PasswordField)             | GovPasswordInput            |       |
| [RadioField](https://wtforms.readthedocs.io/en/2.3.x/fields/#wtforms.fields.RadioField)                   | GovRadioInput               |       |
| [SelectField](https://wtforms.readthedocs.io/en/2.3.x/fields/#wtforms.fields.SelectField)                 | GovSelect                   |       |
| [SelectMultipleField](https://wtforms.readthedocs.io/en/2.3.x/fields/#wtforms.fields.SelectMultipleField) | GovCheckboxesInput          | Note that this renders checkboxes as `<select multiple>` elements are frowned upon. |
| [SubmitField](https://wtforms.readthedocs.io/en/2.3.x/fields/#wtforms.fields.SubmitField)                 | GovSubmitInput              |       |
| [StringField](https://wtforms.readthedocs.io/en/2.3.x/fields/#wtforms.fields.StringField)                 | GovTextInput                |       |
| [TextAreaField](https://wtforms.readthedocs.io/en/2.3.x/fields/#wtforms.fields.TextAreaField)             | GovTextArea                 |       |

In order to generate things like email fields using `GovTextInput` you will need to pass additional params through when rendering it as follows:

```html
{{ form.email_address(params={'type': 'email'}) }}
```

## Running the tests

[TODO]

## Versioning

We use [SemVer](http://semver.org/) for versioning. For the versions available, see the [tags on this repository](https://github.com/LandRegistry/govuk-frontend-wtf/tags).

## Contributors

- [Matt Shaw](https://github.com/matthew-shaw) (Primary maintainer)
- [Andy Mantell](https://github.com/andymantell) (Original author)
- [Hugo Baldwin](https://github.com/byzantime)
