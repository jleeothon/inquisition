# django-inquisition

Flexible search integrated into [Django](https://www.djangoproject.com/) managers.

## Version 0.1

## Introduction

Everybody wants to make searches to Django managers, right? With this app, you can make simple searches such as `Pokemon.objects.search(q="saur")` and your manager will know how to find Bulbasaur, Ivysaur and Venasaur; or `Pokemon.objects.search(q="nin Ground") and you'll get Nincada (Bug/Ground) but not Ninjask (Bug/Flying).

This project uses an [MIT license](http://opensource.org/licenses/MIT). That is, use it however you want to; don't blame if it doesn't quite work; please give me credit (especially if it works). And, if you have any time, please contribute to this project.

## Requirements

- Developed in Python 3.4 (probaby works with any Python 3).
- Developed in Django 1.6 (probably works with any Django).

(Yeah, I know, it sucks).

## Getting started

I don't know yet how to make this installable >_<. Just copy & paste anything you need.

## Usage

### The managers a.k.a. "inquisitioners"

Currently, the search manager (aka "inquisitioners") can be included as a mixin (if you also want to implement another mixin) or a class (if you're only extending this class).

First, create your own inquisitioner subclass.

```Python
# managers.py

class PokemonManager(inquisition.SimpleQSearchManager):
    search_fields = ['name', 'poketype__name']
    order_by = ['name', 'number']

# models.py

from django.db include models

class Pokemon(models.Model):
    name = CharField(max_length=50)
    poketype = ForeignKey('PokeType')
    # TODO poketype2
    
    objects = Pokemon.managers.PokemonManager()
```

Notes:
- `search_fields` support lookups across models`.
- `order_by` supports the same "reverse" syntax as querysets, i.e. `"-name"` to order from Z to A.
- The above are true because they're internally dealt with using Django's querysets.

## Why & when it works

> Only `CharField`s are currently supported, and the type of search is always an `icontains`. Basically, any model instance for which every word split from the `q` string matches (`icontains`) one or more field provided in `search_fields` will be returned in a queryset. This queryset will be ordered if `order_by` is provided.

## Proposals

**For future reference and development of this project.**

Wishing that:
- Searches will not only work on `CharField`s but also number fields.
- We can specify something other than `icontains` for the type of lookup.
- We can specify, for every "search field", against what keyword to lookup.
- We might want to specify the words to be searched for in `*args` instead of a `'q'` keyword argument.

We might be able to achieve the above stated throught providing tuples instead of strings for `search_fields`.

But we should be careful about how to deal with the special `'q'` keyword argument. We might want to include it as `*args` instead of a `'q'` in `**kwargs`.

The following example is just a draft example:

** TODO: this example sucks.**

```Python
class SomeManager(EvenCoolerManager):
    search_fields = [
        'name', 'national_index', 'poketype__name'
    ]
    
    search_fields = {
        # 
        # 'name': 'icontains',
        # 'field2': None, -- implies plain `=`
        # 'field': (None, inquisitor.all) -- implies same as above
        # 'field3': ('icontains', inquisitor.all) ,
        # 'field4': ('istartswith', inquisitor.exclude('name'), -- all search fields except 'name'
        # 'field5': ('endswith', '
    }

# somewhere else

Pokemon.objects.search("bell", poketype="Plant")
```

### Checking for number types

The easiest way would be make a list of numbers from elements in `*args` that can be parsed into numbers. Then for every search field that is a number, use values from this list to check against in lieu of `args` elements. The lookup should default to a plain `=`. However, determining which fields are numeric might be too tricky to attempt right now, it should be better specified in `lookup_types`.

### Type of lookup 

Currently, it is `icontains` for `CharField`s, but it could be plain equality check. For number fields, it could be plain equality check.

To specify the type of lookup, we should use a `lookup_types` dictionary as exemplified above.

### Checking against specific keyword arguments

Currently, the only keyword argument supported is `q`. It is intented to migrate this argument (a single string that is internally tokenized) to manually specify the list of words as in `*args` instead.

Instead of :
```Python
Product.objects.search("couch leather 1998")
```

Take:
```Python
Product.objects.search("couch", "leather" "avant garde")` # might give a slightly better performance
```
