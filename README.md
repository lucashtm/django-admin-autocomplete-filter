[![PyPI version](https://badge.fury.io/py/django-admin-autocomplete-filter.svg)](https://badge.fury.io/py/django-admin-autocomplete-filter)

Django Admin Autocomplete Filter
================================
A simple Django app to render list filters in django admin using autocomplete widget. This app is heavily inspired from [dal-admin-filters.](https://github.com/shamanu4/dal_admin_filters)

Overview:
---------

Django comes preshipped with an admin panel which is a great utility to create quick CRUD's.
The django 2.0 came with much needed [autocomplete_fields](https://docs.djangoproject.com/en/2.1/ref/contrib/admin/#django.contrib.admin.ModelAdmin.autocomplete_fields "autocomplete_fields") which uses select2 widget that comes with a search feature that loads the options asynchronously.
We can use this in django admin list filter.

    

Requirements:
-------------
Requires Django version >= 2.0

Installation:
-------------
You can install it via pip or to get the latest version clone this repo.

```shell script
pip install django-admin-autocomplete-filter
```

Add `admin_auto_filters` to your `INSTALLED_APPS` inside settings.py of your project.

Usage:
------
Let's say we have following models:
```python
from django.db import models

class Artist(models.Model):
    name = models.CharField(max_length=128)

class Album(models.Model):
    name = models.CharField(max_length=64)
    artist = models.ForeignKey(Artist, on_delete=models.CASCADE)
    cover = models.CharField(max_length=256, null=True, default=None)
```
And you would like to filter results in Album Admin on the basis of artist, then you can define `search fields` in Artist and then define filter as:

```python
from django.contrib import admin
from admin_auto_filters.filters import AutocompleteFilter


class ArtistFilter(AutocompleteFilter):
    title = 'Artist' # display title
    field_name = 'artist' # name of the foreign key field


class ArtistAdmin(admin.ModelAdmin):
    search_fields = ['name'] # this is required for django's autocomplete functionality
    # ...


class AlbumAdmin(admin.ModelAdmin):
    list_filter = [ArtistFilter]
    
    """
        defining this class is required for AutocompleteFilter
        it's a bug and I am working on it.
    """
    class Media:
        pass
    
    # ...
```

After following these steps you may see the filter as:

![](https://raw.githubusercontent.com/farhan0581/django-admin-autocomplete-filter/master/admin_auto_filters/media/screenshot1.png)

![](https://raw.githubusercontent.com/farhan0581/django-admin-autocomplete-filter/master/admin_auto_filters/media/screenshot2.png)

Functionality to provide custom view for search:
------------------------------------------------

Now you can also register your custom view instead of using django admin's search_results to control the results in the autocomplete. For this you will need to create your custom view and register the url in your admin class as shown below:

In your views.py:

```python
from admin_auto_filters.views import AutocompleteJsonView


class CustomSearchView(AutocompleteJsonView):
    def get_queryset(self):
        """
           your custom logic goes here.
        """
        queryset = Artist.objects.all().order_by('name')
        return queryset
```

After this, register this view in your admin class as:

```python
from django.contrib import admin
from django.urls import path


class AlbumAdmin(admin.ModelAdmin):
    list_filter = [ArtistFilter]

    class Media:
        pass
    
    def get_urls(self):
        urls = super().get_urls()
        custom_urls = [
            path('custom_search/', self.admin_site.admin_view(CustomSearchView.as_view(model_admin=self)),
                 name='custom_search'),
        ]
        return custom_urls + urls
```

Finally just tell the filter class to use this new view as:

```python
from django.shortcuts import reverse
from admin_auto_filters.filters import AutocompleteFilter


class ArtistFilter(AutocompleteFilter):
    title = 'Artist'
    field_name = 'artist'

    def get_autocomplete_url(self, request, model_admin):
        return reverse('admin:custom_search')
```

Shortcut for creating filters:
------------------------------

It is now possible to use the `AutocompleteFilterFactory` shortcut to create filters
on the fly, as shown below. Nested relations are supported, with
no need to specify the model.

```python
from django.contrib import admin
from admin_auto_filters.filters import AutocompleteFilterFactory


class AlbumAdmin(admin.ModelAdmin):
    list_filter = [
        AutocompleteFilterFactory('Artist', 'artist', 'admin:custom_search', True)
    ]

    def get_urls(self):
        """As above..."""
```

Customizing Widget Text
-----------------------

It is also possible to customize the text displayed in the filter
widget, to use something other than `str(obj)`. This needs to be
configured for both the dropdown endpoint and the widget itself.

In your views.py, override display_text:

```python
from admin_auto_filters.views import AutocompleteJsonView


class CustomSearchView(AutocompleteJsonView):

    @staticmethod
    def display_text(obj):
        return obj.my_str_method()

    def get_queryset(self):
        """As above..."""
```

Then use either of two options to customize the text. Option one is to
specify the form_field in an AutocompleteFilter in your admin.py:

```python
from django import forms
from django.contrib import admin
from django.shortcuts import reverse
from admin_auto_filters.filters import AutocompleteFilter


class FoodChoiceField(forms.ModelChoiceField):
    def label_from_instance(self, obj):
        return obj.my_str_method()


class ArtistFilter(AutocompleteFilter):
    title = 'Artist'
    field_name = 'artist'
    form_field = FoodChoiceField

    def get_autocomplete_url(self, request, model_admin):
        return reverse('admin:custom_search')


class AlbumAdmin(admin.ModelAdmin):
    list_filter = [ArtistFilter]

    def get_urls(self):
        """As above..."""
```

Or use option two. In an AutocompleteFilterFactory in your admin.py,
add a *label_by* argument:

```python
from django.contrib import admin
from admin_auto_filters.filters import AutocompleteFilterFactory


class AlbumAdmin(admin.ModelAdmin):
    list_filter = [
        AutocompleteFilterFactory('Artist', 'artist', 'admin:custom_search', True, label_by='my_str_method')
    ]

    def get_urls(self):
        """As above..."""
```

Contributing:
------------
This project is a combined effort of a lot of selfless developers who try to make things easier. Your contribution is most welcome.

Please make a PR to the branch `pre_release` and make sure your branch does not have any conflicts and clearly mention the problems or improvements your PR is addressing.



License:
--------
Django Admin Autocomplete Filter is an Open Source project licensed under the terms of the GNU GENERAL PUBLIC LICENSE.
