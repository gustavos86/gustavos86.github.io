---
title: Python handling JSON data
date: 2024-07-05 18:00:00 -0700
categories: [PYTHON, JSON]
tags: [python, json]     # TAG names should always be lowercase
---

![]({{ site.baseurl }}/images/services/python_logo.png)

## The JSON library

```python
import json
```

## To remember

- When converting a Python dictionary to JSON, the dictionary keys will always be **strings** in JSON.
- JSON keys are always strings, and not all Python data types can be converted to JSON data types.
- JSON key must be a string wrapped in double quotes `(")`
- JSON strings don’t support single quotes `(')`.
- Python data types **lists** and **tuples** serialize to the same **JSON array** data type.
- Python **dictionaries**, **lists**, or **tuples** can't be used as JSON keys.
- When `json.dumps()` serialize a **Python tuple**, it becomes a **JSON array**. When `json.loads()` deserializee it, a JSON array correctly deserializes into a **Python list**. There is no way of knowing that you want the **JSON array** to be a **Python tuple**.

## JSON Data types

| JSON Data Type |                       Description                        |
|----------------|----------------------------------------------------------|
| object         | A collection of key-value pairs inside curly braces ({}) |
| array          | A list of values wrapped in square brackets ([])         |
| string         | Text wrapped in double quotes ("")                       |
| number         | Integers or floating-point numbers                       |
| boolean        | Either true or false without quotes                      |
| null           | Represents a null value, written as null                 |


## Valid vs Invalid JSON

### Valid JSON example

```json
{
  "name": "Frieda",
  "isDog": true,
  "hobbies": ["eating", "sleeping", "barking"],
  "age": 8,
  "address": {
    "work": null,
    "home": ["Berlin", "Germany"]
  },
  "friends": [
    {
      "name": "Philipp",
      "hobbies": ["eating", "sleeping", "reading"]
    },
    {
      "name": "Mitch",
      "hobbies": ["running", "snacking"]
    }
  ]
}
```

### Invalid JSON example

```json
 1:  {
 2:    "name": 'Frieda',
 3:    "address": {
 4:      "work": null, // Doesn't pay rent either
 5:      "home": "Berlin",
 6:    },
 7:    "friends": [
 8:      {
 9:        "name": "Philipp",
10:        "hobbies": ["eating", "sleeping", "reading",]
11:      }
12:    ]
13:  }
```

- **Line 2** wraps the string in single quotes.
- **Line 4** uses an inline comment.
- **Line 5** has a trailing comma after the final key-value pair.
- **Line 10** contains a trailing comma in the array.

## Serialize (Python to JSON)

This table shows Python data types to JSON data types conversions:

| Python |  JSON  |
|--------|--------|
| dict   | object |
| list   | array  |
| tuple  | array  |
| str    | string |
| int    | number |
| float  | number |
| True   | true   |
| False  | false  |
| None   | null   |

Not all Python **dictionary keys** can be converted into **JSON key strings**

| Python Data Type | Allowed as JSON Key |
|------------------|---------------------|
| dict             | ❌                  |
| list             | ❌                  |
| tuple            | ❌                  |
| str              | ✅                  |
| int              | ✅                  |
| float            | ✅                  |
| bool             | ✅                  |
| None             | ✅                  |

----

`json.dumps()` converts a Python dictionary to a JSON-formatted string, which represents a JSON object.

Returns a Python `string`.

```python
json.dumps()
```

Example:

```python
In [1]: import json

In [2]: products = {
   ...:     'Jeans': 49.99,
   ...:     'T-Shirt': 14.99,
   ...:     'Suit': 89.99,
   ...: }

In [3]: json_data = json.dumps(products)

In [4]: json_data
Out[4]: '{"Jeans": 49.99, "T-Shirt": 14.99, "Suit": 89.99}'

In [5]: type(json_data)
Out[5]: str
```

Other arguments:

- **skipkeys** ignores unsupported JSON data.

Tuples cannot be used as JSON keys.

Example:

```python
>>> available_nums = {(1, 2): True, 3: False}
>>> json.dumps(available_nums)
Traceback (most recent call last):
  ...
TypeError: keys must be str, int, float, bool or None, not tuple

>>> json.dumps(available_nums, skipkeys=True)
'{"3": false}'
```

- **sort_keys** sort the dictionary keys.

Example:

```python
>>> toy_conditions = {"chew bone": 7, "ball": 3, "sock": -1}
>>> json.dumps(toy_conditions, sort_keys=True)
'{"ball": 3, "chew bone": 7, "sock": -1}'
```

- **indent** used to prettify the JSON output with the purpose to make it easy to be read by humans.

Example:

```python
>>> print(json.dumps(dog_friend, indent=2))
{
  "name": "Mitch",
  "age": 6.5
}

>>> print(json.dumps(dog_friend, indent=4))
{
    "name": "Mitch",
    "age": 6.5
}
```

----

`json.dump()` is used when to save the generated JSON data to a local file.

```python
json.dump()
```

Example:

```python
with open("hello_frieda.json", mode="w", encoding="utf-8") as write_file:
    json.dump(dog_data, write_file)
```

where **dog_data** is the Python Dictionary

## Deserialize (JSON to Python)

This table shows JSON data types to Python data types conversions:

|  JSON  | Python |
|--------|--------|
| object | dict   |
| array  | list   |
| string | str    |
| number | int    |
| number | float  |
| true   | True   |
| false  | False  |
| null   | None   |

`json.loads()` converts a JSON-formatted string to a Python dictionary.

```python
json.loads()
```

Example:

```python
In [6]: json_data = '{"Jeans": 49.99, "T-Shirt": 14.99, "Suit": 89.99}'

In [7]: type(json_data)
Out[7]: str

In [8]: python_data = json.loads(json_data)

In [9]: python_data
Out[9]: {'Jeans': 49.99, 'T-Shirt': 14.99, 'Suit': 89.99}

In [10]: type(python_data)
Out[10]: dict
```

----

`json.load()` is used when to load a local JSON file to Python.


```python
json.load()
```

Example:

```python
>>> with open("hello_frieda.json", mode="r", encoding="utf-8") as read_file:
...     frie_data = json.load(read_file)
```

## JSON in the Terminal

From the terminal

```bash
python -m json.tool <json_file>
```

usage:

```bash
$ python -m json.tool -h
usage: python -m json.tool [-h] [--sort-keys] [--no-ensure-ascii] [--json-lines]
                           [--indent INDENT | --tab | --no-indent | --compact]
                           [infile] [outfile]

A simple command line interface for json module to validate and pretty-print JSON objects.

positional arguments:
  infile             a JSON file to be validated or pretty-printed
  outfile            write the output of infile to outfile

options:
  -h, --help         show this help message and exit
  --sort-keys        sort the output of dictionaries alphabetically by key
  --no-ensure-ascii  disable escaping of non-ASCII characters
  --json-lines       parse input using the JSON Lines format. Use with --no-indent or --compact to produce valid JSON
                     Lines output.
  --indent INDENT    separate items with newlines and use this number of spaces for indentation
  --tab              separate items with newlines and use tabs for indentation
  --no-indent        separate items with spaces rather than newlines
  --compact          suppress all whitespace separation (most compact)
```

### Validate JSON

```bash
$ python -m json.tool dog_friend.json
{
    "name": "Mitch",
    "age": 6.5
}
```

on error:

```bash
$ python -m json.tool dog_friend.json
Expecting ',' delimiter: line 3 column 5 (char 26)
```

### Pretty Print JSON

Default is ``--indent 4``, but it can be different:

```bash
$ python -m json.tool hello_frieda.json --indent 2
{
  "name": "Frieda",
  "is_dog": true,
  "hobbies": [
    "eating",
    "sleeping",
    "barking"
  ],
  "age": 8,
  "address": {
    "work": null,
    "home": [
      "Berlin",
      "Germany"
    ]
  },
  "friends": [
    {
      "name": "Philipp",
      "hobbies": [
        "eating",
        "sleeping",
        "reading"
      ]
    },
    {
      "name": "Mitch",
      "hobbies": [
        "running",
        "snacking"
      ]
    }
  ]
}
```

## Minify

It is a good idea to remove newlines and whitespaces when sending JSON data over the network.
This is to make the JSON payload as light as possible.
For this purpose use the following:

Use args `indent=None, separators=(",", ":")` with `json.dumps()` or `json.dump()`

```python
In [4]: json_data = '''{
   ...:   "name": "Frieda",
   ...:   "isDog": true,
   ...:   "hobbies": ["eating", "sleeping", "barking"],
   ...:   "age": 8,
   ...:   "address": {
   ...:     "work": null,
   ...:     "home": ["Berlin", "Germany"]
   ...:   },
   ...:   "friends": [
   ...:     {
   ...:       "name": "Philipp",
   ...:       "hobbies": ["eating", "sleeping", "reading"]
   ...:     },
   ...:     {
   ...:       "name": "Mitch",
   ...:       "hobbies": ["running", "snacking"]
   ...:     }
   ...:   ]
   ...: }'''

In [7]: import json

In [8]: python_data = json.loads(json_data)

In [9]: python_data
Out[9]:
{'name': 'Frieda',
 'isDog': True,
 'hobbies': ['eating', 'sleeping', 'barking'],
 'age': 8,
 'address': {'work': None, 'home': ['Berlin', 'Germany']},
 'friends': [{'name': 'Philipp', 'hobbies': ['eating', 'sleeping', 'reading']},
  {'name': 'Mitch', 'hobbies': ['running', 'snacking']}]}

In [10]: type(python_data)
Out[10]: dict

In [11]: mini_json = json.dumps(python_data, indent=None, separators=(",", ":"))

In [12]: mini_json
Out[12]: '{"name":"Frieda","isDog":true,"hobbies":["eating","sleeping","barking"],"age":8,"address":{"work":null,"home":["Berlin","Germany"]},"friends":[{"name":"Philipp","hobbies":["eating","sleeping","reading"]},{"name":"Mitch","hobbies":["running","snacking"]}]}'

In [13]: type(mini_json)
Out[13]: str
```

or

```python
>>> mini_json = json.dumps(python_data, indent=None, separators=(",", ":"))

>>> with open("mini_frieda.json", mode="w", encoding="utf-8") as output_file:
...     output_file.write(mini_json)
...
```

# References

- https://realpython.com/python-json/
