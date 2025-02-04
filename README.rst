================
Dataclass Mage
================

Full documentation is available at `Read The Docs`_. (`Installation`_)

.. image:: https://img.shields.io/pypi/v/dataclass-mage.svg
        :target: https://pypi.org/project/dataclass-mage

.. image:: https://img.shields.io/pypi/pyversions/dataclass-mage.svg
        :target: https://pypi.org/project/dataclass-mage

.. image:: https://github.com/Frequency0/dataclass-mage/actions/workflows/dev.yml/badge.svg
        :target: https://github.com/Frequency0/dataclass-mage/actions/workflows/dev.yml

.. image:: https://readthedocs.org/projects/dataclass-mage/badge/?version=latest
        :target: https://dataclass-mage.readthedocs.io/en/latest/?version=latest
        :alt: Documentation Status

This library provides a set of simple, yet elegant *wizarding* tools for
interacting with the Python ``dataclasses`` module.

    The primary use is as a fast serialization framework that enables dataclass instances to
    be converted to/from JSON; this works well in particular with a *nested dataclass* model.

-------------------

**Behold, the power of the Dataclass Mage**::

    >>> from __future__ import annotations
    >>> from dataclasses import dataclass, field
    >>> from dataclass_mage import JSONWizard
    ...
    >>> @dataclass
    ... class MyClass(JSONWizard):
    ...     my_str: str | None
    ...     is_active_tuple: tuple[bool, ...]
    ...     list_of_int: list[int] = field(default_factory=list)
    ...
    >>> string = """
    ... {
    ...   "my_str": 20,
    ...   "ListOfInt": ["1", "2", 3],
    ...   "isActiveTuple": ["true", false, 1]
    ... }
    ... """
    ...
    >>> instance = MyClass.from_json(string)
    >>> instance
    MyClass(my_str='20', is_active_tuple=(True, False, True), list_of_int=[1, 2, 3])
    >>> instance.to_json()
    '{"myStr": "20", "isActiveTuple": [true, false, true], "listOfInt": [1, 2, 3]}'
    >>> instance == MyClass.from_dict(instance.to_dict())
    True

---

.. contents:: Contents
   :depth: 1
   :local:
   :backlinks: none


Installation
------------

The Dataclass Mage library is available `on PyPI`_, and can be installed with ``pip``:

.. code-block:: shell

    $ pip install dataclass-mage

The ``dataclass-mage`` library officially supports **Python 3.9** or higher.

Features
--------

Here are the supported features that ``dataclass-mage`` currently provides:

-  *JSON/YAML (de)serialization*: marshal dataclasses to/from JSON, YAML, and Python
   ``dict`` objects.
-  *Field properties*: support for using properties with default
   values in dataclass instances.
-  *JSON to Dataclass generation*: construct a dataclass schema with a JSON file
   or string input.


Wizard Mixins
-------------

In addition to the ``JSONWizard``, here are a few extra Mixin_ classes that might prove quite convenient to use.

* `JSONListWizard`_ -- Extends ``JSONWizard`` to return `Container`_ -- instead of *list* -- objects where possible.
* `JSONFileWizard`_ -- Makes it easier to convert dataclass instances from/to JSON files on a local drive.
* `YAMLWizard`_ -- Provides support to convert dataclass instances to/from YAML, using the default ``PyYAML`` parser.


Supported Types
---------------

The Dataclass Mage library provides inherent support for standard Python collections
such as ``list``, ``dict`` and ``set``, as well as most Generics from the typing
module, such as ``Union`` and ``Any``. Other commonly used types such as ``Enum``,
``defaultdict``, and date and time objects such as ``datetime`` are also natively
supported.

For a complete list of the supported Python types, including info on the
load/dump process for special types, check out the `Supported Types`_ section
in the docs.


Usage and Examples
------------------

Using the built-in JSON marshalling support for dataclasses:

    Note: The following example should work in **Python 3.7+** with the included ``__future__``
    import.

.. code:: python3

    from __future__ import annotations  # This can be removed in Python 3.10+

    from dataclasses import dataclass, field
    from datetime import date
    from enum import Enum

    from dataclass_mage import JSONWizard


    @dataclass
    class Data(JSONWizard):

        class _(JSONWizard.Meta):
            # Sets the target key transform to use for serialization;
            # defaults to `camelCase` if not specified.
            key_transform_with_dump = 'LISP'

        a_sample_bool: bool
        values: list[Inner] = field(default_factory=list)


    @dataclass
    class Inner:
        vehicle: Car | None
        my_dates: dict[int, date]


    class Car(Enum):
        SEDAN = 'BMW Coupe'
        SUV = 'Toyota 4Runner'
        JEEP = 'Jeep Cherokee'


    def main():

        my_dict = {
            'values': [
                {
                    'vehicle': 'Toyota 4Runner',
                    'My-Dates': {'123': '2023-01-31'}
                },
                {
                    'vehicle': None,
                    'my_dates': {}
                }
            ],
            'aSampleBool': 'TRUE'
        }

        # De-serialize (a JSON string or dictionary data) into a `Data` instance.
        data = Data.from_dict(my_dict)

        print(repr(data))
        # > Data(a_sample_bool=True, values=[Inner(vehicle=<Car.SUV: 'Toyota 4Runner'>, ...)])

        # assert enums values are as expected
        assert data.values[0].vehicle is Car.SUV

        print(data.to_json(indent=2))
        # {
        #   "a-sample-bool": true,
        #   "values": [
        #     {
        #       "vehicle": "Toyota 4Runner",
        #       "my-dates": {
        #         "123": "2023-01-31"
        #       },
        #   ...

        # True
        assert data == data.from_json(data.to_json())

    if __name__ == '__main__':
        main()

... and with the ``property_wizard``, which provides support for
`field properties`_ with default values in dataclasses:

.. code:: python3

    from __future__ import annotations  # This can be removed in Python 3.10+

    from dataclasses import dataclass, field
    from typing_extensions import Annotated

    from dataclass_mage import property_wizard


    @dataclass
    class Vehicle(metaclass=property_wizard):
        # Note: The example below uses the default value from the `field` extra in
        # the `Annotated` definition; if `wheels` were annotated as `int | str`,
        # it would default to 0, because `int` appears as the first type argument.
        #
        # Any right-hand value assigned to `wheels` is ignored as it is simply
        # re-declared by the property; here it is simply omitted for brevity.
        wheels: Annotated[int | str, field(default=4)]

        # This is a shorthand version of the above; here an IDE suggests
        # `_wheels` as a keyword argument to the constructor method, though
        # it will actually be named as `wheels`.
        # _wheels: int | str = 4

        @property
        def wheels(self) -> int:
            return self._wheels

        @wheels.setter
        def wheels(self, wheels: int | str):
            self._wheels = int(wheels)


    if __name__ == '__main__':
        v = Vehicle()
        print(v)
        # prints:
        #   Vehicle(wheels=4)

        v = Vehicle(wheels=3)
        print(v)

        v = Vehicle('6')
        print(v)

        assert v.wheels == 6, 'The constructor should use our setter method'

        # Confirm that we go through our setter method
        v.wheels = '123'
        assert v.wheels == 123

... or generate a dataclass schema for JSON input, via the `wiz-cli`_ tool:

.. code:: shell

    $ echo '{"myFloat": "1.23", "Products": [{"created_at": "2021-11-17"}]}' | wiz gs - my_file

    # Contents of my_file.py
    from dataclasses import dataclass
    from datetime import date
    from typing import List, Union

    from dataclass_mage import JSONWizard


    @dataclass
    class Data(JSONWizard):
        """
        Data dataclass

        """
        my_float: Union[float, str]
        products: List['Product']


    @dataclass
    class Product:
        """
        Product dataclass

        """
        created_at: date


JSON Marshalling
----------------

``JSONSerializable`` (aliased to ``JSONWizard``) is a Mixin_ class which
provides the following helper methods that are useful for serializing (and loading)
a dataclass instance to/from JSON, as defined by the ``AbstractJSONWizard``
interface.

.. list-table::
   :widths: 10 40 35
   :header-rows: 1

   * - Method
     - Example
     - Description
   * - ``from_json``
     - `item = Product.from_json(string)`
     - Converts a JSON string to an instance of the
       dataclass, or a list of the dataclass instances.
   * - ``from_list``
     - `list_of_item = Product.from_list(l)`
     - Converts a Python ``list`` object to a list of the
       dataclass instances.
   * - ``from_dict``
     - `item = Product.from_dict(d)`
     - Converts a Python ``dict`` object to an instance
       of the dataclass.
   * - ``to_dict``
     - `d = item.to_dict()`
     - Converts the dataclass instance to a Python ``dict``
       object that is JSON serializable.
   * - ``to_json``
     - `string = item.to_json()`
     - Converts the dataclass instance to a JSON string
       representation.
   * - ``list_to_json``
     - `string = Product.list_to_json(list_of_item)`
     - Converts a list of dataclass instances to a JSON string
       representation.

Additionally, it adds a default ``__str__`` method to subclasses, which will
pretty print the JSON representation of an object; this is quite useful for
debugging purposes. Whenever you invoke ``print(obj)`` or ``str(obj)``, for
example, it'll call this method which will format the dataclass object as
a prettified JSON string. If you prefer a ``__str__`` method to not be
added, you can pass in ``str=False`` when extending from the Mixin class
as mentioned `here <https://dataclass-mage.readthedocs.io/en/latest/common_use_cases/skip_the_str.html>`_.

Note that the ``__repr__`` method, which is implemented by the
``dataclass`` decorator, is also available. To invoke the Python object
representation of the dataclass instance, you can instead use
``repr(obj)`` or ``f'{obj!r}'``.

To mark a dataclass as being JSON serializable (and
de-serializable), simply sub-class from ``JSONSerializable`` as shown
below. You can also extend from the aliased name ``JSONWizard``, if you
prefer to use that instead.

Check out a `more complete example`_ of using the ``JSONSerializable``
Mixin class.

No Inheritance Needed
---------------------

It is important to note that the main purpose of sub-classing from
``JSONWizard`` Mixin class is to provide helper methods like ``from_dict``
and ``to_dict``, which makes it much more convenient and easier to load or
dump your data class from and to JSON.

That is, it's meant to *complement* the usage of the ``dataclass`` decorator,
rather than to serve as a drop-in replacement for data classes, or to provide type
validation for example; there are already excellent libraries like `pydantic`_ that
provide these features if so desired.

However, there may be use cases where we prefer to do away with the class
inheritance model introduced by the Mixin class. In the interests of convenience
and also so that data classes can be used *as is*, the Dataclass
Wizard library provides the helper functions ``fromlist`` and ``fromdict``
for de-serialization, and ``asdict`` for serialization. These functions also
work recursively, so there is full support for nested dataclasses -- just as with
the class inheritance approach.

Here is an example to demonstrate the usage of these helper functions:

.. note::
  As of *v0.18.0*, the Meta config for the main dataclass will cascade down
  and be merged with the Meta config (if specified) of each nested dataclass. To
  disable this behavior, you can pass in ``recursive=False`` to the Meta config.

.. code:: python3

    from __future__ import annotations

    from dataclasses import dataclass, field
    from datetime import datetime, date

    from dataclass_mage import fromdict, asdict, DumpMeta


    @dataclass
    class A:
        created_at: datetime
        list_of_b: list[B] = field(default_factory=list)


    @dataclass
    class B:
        my_status: int | str
        my_date: date | None = None


    source_dict = {'createdAt': '2010-06-10 15:50:00Z',
                   'List-Of-B': [
                       {'MyStatus': '200', 'my_date': '2021-12-31'}
                   ]}

    # De-serialize the JSON dictionary object into an `A` instance.
    a = fromdict(A, source_dict)

    print(repr(a))
    # A(created_at=datetime.datetime(2010, 6, 10, 15, 50, tzinfo=datetime.timezone.utc),
    #   list_of_b=[B(my_status='200', my_date=datetime.date(2021, 12, 31))])

    # Set an optional dump config for the main dataclass, for example one which
    # converts converts date and datetime objects to a unix timestamp (as an int)
    #
    # Note that `recursive=True` is the default, so this Meta config will be
    # merged with the Meta config (if specified) of each nested dataclass.
    DumpMeta(marshal_date_time_as='TIMESTAMP',
             key_transform='SNAKE',
             # Finally, apply the Meta config to the main dataclass.
             ).bind_to(A)

    # Serialize the `A` instance to a Python dict object.
    json_dict = asdict(a)

    expected_dict = {'created_at': 1276185000, 'list_of_b': [{'my_status': '200', 'my_date': 1640926800}]}

    print(json_dict)
    # Assert that we get the expected dictionary object.
    assert json_dict == expected_dict

Custom Key Mappings
-------------------

If you ever find the need to add a `custom mapping`_ of a JSON key to a dataclass
field (or vice versa), the helper function ``json_field`` -- which can be
considered an alias to ``dataclasses.field()`` -- is one approach that can
resolve this.

Example below:

.. code:: python3

    from dataclasses import dataclass

    from dataclass_mage import JSONSerializable, json_field


    @dataclass
    class MyClass(JSONSerializable):

        my_str: str = json_field('myString1', all=True)


    # De-serialize a dictionary object with the newly mapped JSON key.
    d = {'myString1': 'Testing'}
    c = MyClass.from_dict(d)

    print(repr(c))
    # prints:
    #   MyClass(my_str='Testing')

    # Assert we get the same dictionary object when serializing the instance.
    assert c.to_dict() == d

Extending from ``Meta``
-----------------------

Looking to change how ``date`` and ``datetime`` objects are serialized to JSON? Or
prefer that field names appear in *snake case* when a dataclass instance is serialized?

The inner ``Meta`` class allows easy configuration of such settings, as
shown below; and as a nice bonus, IDEs should be able to assist with code completion
along the way.

.. note::
  As of *v0.18.0*, the Meta config for the main dataclass will cascade down
  and be merged with the Meta config (if specified) of each nested dataclass. To
  disable this behavior, you can pass in ``recursive=False`` to the Meta config.

.. code:: python3

    from dataclasses import dataclass
    from datetime import date

    from dataclass_mage import JSONWizard
    from dataclass_mage.enums import DateTimeTo


    @dataclass
    class MyClass(JSONWizard):

        class _(JSONWizard.Meta):
            marshal_date_time_as = DateTimeTo.TIMESTAMP
            key_transform_with_dump = 'SNAKE'

        my_str: str
        my_date: date


    data = {'my_str': 'test', 'myDATE': '2010-12-30'}

    c = MyClass.from_dict(data)

    print(repr(c))
    # prints:
    #   MyClass(my_str='test', my_date=datetime.date(2010, 12, 30))

    string = c.to_json()
    print(string)
    # prints:
    #   {"my_str": "test", "my_date": 1293685200}

Other Uses for ``Meta``
~~~~~~~~~~~~~~~~~~~~~~~

Here are a few additional use cases for the inner ``Meta`` class. Note that
a full list of available settings can be found in the `Meta`_ section in the docs.

Debug Mode
##########

Enables additional (more verbose) log output. For example, a message can be
logged whenever an unknown JSON key is encountered when
``from_dict`` or ``from_json`` is called.

This also results in more helpful error messages during the JSON load
(de-serialization) process, such as when values are an invalid type --
i.e. they don't match the annotation for the field. This can be particularly
useful for debugging purposes.

.. note::
  There is a minor performance impact when DEBUG mode is enabled;
  for that reason, I would personally advise against enabling
  this in a *production* environment.

Handle Unknown JSON Keys
########################

The default behavior is to ignore any unknown or extraneous JSON keys that are
encountered when ``from_dict`` or ``from_json`` is called, and emit a "warning"
which is visible when *debug* mode is enabled (and logging is properly configured).
An unknown key is one that does not have a known mapping to a dataclass field.

However, we can also raise an error in such cases if desired. The below
example demonstrates a use case where we want to raise an error when
an unknown JSON key is encountered in the  *load* (de-serialization) process.

.. code:: python3

    import logging
    from dataclasses import dataclass

    from dataclass_mage import JSONWizard
    from dataclass_mage.errors import UnknownJSONKey


    # Sets up application logging if we haven't already done so
    logging.basicConfig(level='INFO')


    @dataclass
    class Container(JSONWizard):

        class _(JSONWizard.Meta):
            # True to enable Debug mode for additional (more verbose) log output.
            debug_enabled = True
            # True to raise an class:`UnknownJSONKey` when an unmapped JSON key is
            # encountered when `from_dict` or `from_json` is called. Note that by
            # default, this is also recursively applied to any nested dataclasses.
            raise_on_unknown_json_key = True

        element: 'MyElement'


    @dataclass
    class MyElement:
        my_str: str
        my_float: float


    d = {
        'element': {
            'myStr': 'string',
            'my_float': '1.23',
            # Notice how this key is not mapped to a known dataclass field!
            'my_bool': 'Testing'
        }
    }

    # Try to de-serialize the dictionary object into a `MyClass` object.
    try:
        c = Container.from_dict(d)
    except UnknownJSONKey as e:
        print('Received error:', type(e).__name__)
        print('Class:', e.class_name)
        print('Unknown JSON key:', e.json_key)
        print('JSON object:', e.obj)
        print('Known Fields:', e.fields)
    else:
        print('Successfully de-serialized the JSON object.')
        print(repr(c))

Date and Time with Custom Patterns
----------------------------------

As of *v0.20.0*, date and time strings in a `custom format`_ can be de-serialized
using the ``DatePattern``, ``TimePattern``, and ``DateTimePattern`` type annotations,
representing patterned `date`, `time`, and `datetime` objects respectively.

This will internally call ``datetime.strptime`` with the format specified in the annotation,
and also use the ``fromisoformat()`` method in case the date string is in ISO-8601 format.
All dates and times will continue to be serialized as ISO format strings by default. For more
info, check out the `Patterned Date and Time`_ section in the docs.

A brief example of the intended usage is shown below:

.. code:: python3

    from dataclasses import dataclass
    from datetime import time, datetime
    from typing import List
    # Note: in Python 3.9+, you can import this from `typing` instead
    from typing_extensions import Annotated

    from dataclass_mage import fromdict, asdict, DatePattern, TimePattern, Pattern


    @dataclass
    class MyClass:
        date_field: DatePattern['%m-%Y']
        dt_field: Annotated[datetime, Pattern('%m/%d/%y %H.%M.%S')]
        time_field1: TimePattern['%H:%M']
        time_field2: Annotated[List[time], Pattern('%I:%M %p')]


    data = {'date_field': '12-2022',
            'time_field1': '15:20',
            'dt_field': '1/02/23 02.03.52',
            'time_field2': ['1:20 PM', '12:30 am']}

    class_obj = fromdict(MyClass, data)

    # All annotated fields de-serialize as just date, time, or datetime, as shown.
    print(class_obj)
    # MyClass(date_field=datetime.date(2022, 12, 1), dt_field=datetime.datetime(2023, 1, 2, 2, 3, 52),
    #         time_field1=datetime.time(15, 20), time_field2=[datetime.time(13, 20), datetime.time(0, 30)])

    # All date/time fields are serialized as ISO-8601 format strings by default.
    print(asdict(class_obj))
    # {'dateField': '2022-12-01', 'dtField': '2023-01-02T02:03:52',
    #  'timeField1': '15:20:00', 'timeField2': ['13:20:00', '00:30:00']}

    # But, the patterned date/times can still be de-serialized back after
    # serialization. In fact, it'll be faster than parsing the custom patterns!
    assert class_obj == fromdict(MyClass, asdict(class_obj))

Dataclasses in ``Union`` Types
------------------------------

The ``dataclass-mage`` library fully supports declaring dataclass models in
`Union`_ types as field annotations, such as ``list[Wizard | Archer | Barbarian]``.

As of *v0.19.0*, there is added support to  *auto-generate* tags for a dataclass model
-- based on the class name -- as well as to specify a custom *tag key* that will be
present in the JSON object, which defaults to a special ``__tag__`` key otherwise.
These two options are controlled by the ``auto_assign_tags`` and ``tag_key``
attributes (respectively) in the ``Meta`` config.

To illustrate a specific example, a JSON object such as
``{"oneOf": {"type": "A", ...}, ...}`` will now automatically map to a dataclass
instance ``A``, provided that the ``tag_key`` is correctly set to "type", and
the field ``one_of`` is annotated as a Union type in the ``A | B`` syntax.

Let's start out with an example, which aims to demonstrate the simplest usage of
dataclasses in ``Union`` types. For more info, check out the
`Dataclasses in Union Types`_ section in the docs.

.. code:: python3

    from __future__ import annotations

    from dataclasses import dataclass

    from dataclass_mage import JSONWizard


    @dataclass
    class Container(JSONWizard):

        class Meta(JSONWizard.Meta):
            tag_key = 'type'
            auto_assign_tags = True

        objects: list[A | B | C]


    @dataclass
    class A:
        my_int: int
        my_bool: bool = False


    @dataclass
    class B:
        my_int: int
        my_bool: bool = True


    @dataclass
    class C:
        my_str: str


    data = {
        'objects': [
            {'type': 'A', 'my_int': 42},
            {'type': 'C', 'my_str': 'hello world'},
            {'type': 'B', 'my_int': 123},
            {'type': 'A', 'my_int': 321, 'myBool': True}
        ]
    }


    c = Container.from_dict(data)
    print(f'{c!r}')

    # True
    assert c == Container(objects=[A(my_int=42, my_bool=False),
                                   C(my_str='hello world'),
                                   B(my_int=123, my_bool=True),
                                   A(my_int=321, my_bool=True)])


    print(c.to_dict())
    # prints the following on a single line:
    # {'objects': [{'myInt': 42, 'myBool': False, 'type': 'A'},
    #              {'myStr': 'hello world', 'type': 'C'},
    #              {'myInt': 123, 'myBool': True, 'type': 'B'},
    #              {'myInt': 321, 'myBool': True, 'type': 'A'}]}

    # True
    assert c == c.from_json(c.to_json())

Serialization Options
---------------------

The following parameters can be used to fine-tune and control how the serialization of a
dataclass instance to a Python ``dict`` object or JSON string is handled.

Skip Defaults
~~~~~~~~~~~~~

A common use case is skipping fields with default values - based on the ``default``
or ``default_factory`` argument to ``dataclasses.field`` - in the serialization
process.

The attribute ``skip_defaults`` in the inner ``Meta`` class can be enabled, to exclude
such field values from serialization.The ``to_dict`` method (or the ``asdict`` helper
function) can also be passed an ``skip_defaults`` argument, which should have the same
result. An example of both these approaches is shown below.

.. code:: python3

    from collections import defaultdict
    from dataclasses import field, dataclass
    from typing import DefaultDict, List

    from dataclass_mage import JSONWizard


    @dataclass
    class MyClass(JSONWizard):

        class _(JSONWizard.Meta):
            skip_defaults = True

        my_str: str
        other_str: str = 'any value'
        optional_str: str = None
        my_list: List[str] = field(default_factory=list)
        my_dict: DefaultDict[str, List[float]] = field(
            default_factory=lambda: defaultdict(list))


    print('-- Load (Deserialize)')
    c = MyClass('abc')
    print(f'Instance: {c!r}')

    print('-- Dump (Serialize)')
    string = c.to_json()
    print(string)

    assert string == '{"myStr": "abc"}'

    print('-- Dump (with `skip_defaults=False`)')
    print(c.to_dict(skip_defaults=False))

Exclude Fields
~~~~~~~~~~~~~~

You can also exclude specific dataclass fields (and their values) from the serialization
process. There are two approaches that can be used for this purpose:

* The argument ``dump=False`` can be passed in to the ``json_key`` and ``json_field``
  helper functions. Note that this is a more permanent option, as opposed to the one
  below.

* The ``to_dict`` method (or the ``asdict`` helper function ) can be passed
  an ``exclude`` argument, containing a list of one or more dataclass field names
  to exclude from the serialization process.

Additionally, here is an example to demonstrate usage of both these approaches:

.. code:: python3

    from dataclasses import dataclass
    from typing import Annotated

    from dataclass_mage import JSONWizard, json_key, json_field


    @dataclass
    class MyClass(JSONWizard):

        my_str: str
        my_int: int
        other_str: Annotated[str, json_key('AnotherStr', dump=False)]
        my_bool: bool = json_field('TestBool', dump=False)


    data = {'MyStr': 'my string',
            'myInt': 1,
            'AnotherStr': 'testing 123',
            'TestBool': True}

    print('-- From Dict')
    c = MyClass.from_dict(data)
    print(f'Instance: {c!r}')

    # dynamically exclude the `my_int` field from serialization
    additional_exclude = ('my_int',)

    print('-- To Dict')
    out_dict = c.to_dict(exclude=additional_exclude)
    print(out_dict)

    assert out_dict == {'myStr': 'my string'}

Field Properties
----------------

The Python ``dataclasses`` library has some `key limitations`_
with how it currently handles properties and default values.

The ``dataclass-mage`` package natively provides support for using
field properties with default values in dataclasses. The main use case
here is to assign an initial value to the field property, if one is not
explicitly passed in via the constructor method.

To use it, simply import
the ``property_wizard`` helper function, and add it as a metaclass on
any dataclass where you would benefit from using field properties with
default values. The metaclass also pairs well with the ``JSONSerializable``
mixin class.

For more examples and important how-to's on properties with default values,
refer to the `Using Field Properties`_ section in the documentation.

Contributing
------------

Contributions are welcome! Open a pull request to fix a bug, or `open an issue`_
to discuss a new feature or change.

Check out the `Contributing`_ section in the docs for more info.

TODOs
-----

All feature ideas or suggestions for future consideration, have been currently added
`as milestones`_ in the project's GitHub repo.

Credits
-------

This package was created with Cookiecutter_ and the `rnag/cookiecutter-pypackage`_ project template.

.. _Read The Docs: https://dataclass-mage.readthedocs.io
.. _Installation: https://dataclass-mage.readthedocs.io/en/latest/installation.html
.. _on PyPI: https://pypi.org/project/dataclass-mage/
.. _Cookiecutter: https://github.com/cookiecutter/cookiecutter
.. _`rnag/cookiecutter-pypackage`: https://github.com/rnag/cookiecutter-pypackage
.. _`Contributing`: https://dataclass-mage.readthedocs.io/en/latest/contributing.html
.. _`open an issue`: https://github.com/Frequency0/dataclass-mage/issues
.. _`JSONListWizard`: https://dataclass-mage.readthedocs.io/en/latest/common_use_cases/wizard_mixins.html#jsonlistwizard
.. _`JSONFileWizard`: https://dataclass-mage.readthedocs.io/en/latest/common_use_cases/wizard_mixins.html#jsonfilewizard
.. _`YAMLWizard`: https://dataclass-mage.readthedocs.io/en/latest/common_use_cases/wizard_mixins.html#yamlwizard
.. _`Container`: https://dataclass-mage.readthedocs.io/en/latest/dataclass_mage.html#dataclass_mage.Container
.. _`Supported Types`: https://dataclass-mage.readthedocs.io/en/latest/overview.html#supported-types
.. _`Mixin`: https://stackoverflow.com/a/547714/10237506
.. _`Meta`: https://dataclass-mage.readthedocs.io/en/latest/common_use_cases/meta.html
.. _`pydantic`: https://pydantic-docs.helpmanual.io/
.. _`Using Field Properties`: https://dataclass-mage.readthedocs.io/en/latest/using_field_properties.html
.. _`field properties`: https://dataclass-mage.readthedocs.io/en/latest/using_field_properties.html
.. _`custom mapping`: https://dataclass-mage.readthedocs.io/en/latest/common_use_cases/custom_key_mappings.html
.. _`wiz-cli`: https://dataclass-mage.readthedocs.io/en/latest/wiz_cli.html
.. _`key limitations`: https://florimond.dev/en/posts/2018/10/reconciling-dataclasses-and-properties-in-python/
.. _`more complete example`: https://dataclass-mage.readthedocs.io/en/latest/examples.html#a-more-complete-example
.. _custom format: https://docs.python.org/3/library/datetime.html#strftime-and-strptime-format-codes
.. _`Patterned Date and Time`: https://dataclass-mage.readthedocs.io/en/latest/common_use_cases/patterned_date_time.html
.. _Union: https://docs.python.org/3/library/typing.html#typing.Union
.. _`Dataclasses in Union Types`: https://dataclass-mage.readthedocs.io/en/latest/common_use_cases/dataclasses_in_union_types.html
.. _as milestones: https://github.com/Frequency0/dataclass-mage/milestones
