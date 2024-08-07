---
title: Python Pytest examples
date: 2024-07-21 21:00:00 -0700
categories: [PYTHON, PYTEST]
tags: [python]     # TAG names should always be lowercase
---

![]({{ site.baseurl }}/images/services/python_logo.png)

## @pytest.mark.parametrize example

Used to pass more than one case to a test.

The next example is to test a funcion that validates the **string** conforms to the Juniper naming convention. We expect the function to return a **boolean**.

We test the function by passing many differet inputs (where not all of them are **strings**) along with the expected **boolean** value we expect the function to return on each case.

```python
import pytest

def is_valid_juniper_interface_format(interface) -> bool:
    """
    Function that validates that we are passing as a string a Juniper interface
    that adheres to the Junos OS naming convention
    """
    ...
    return result

@pytest.mark.parametrize("interface, result", [
    ('ge-0/0/1', True),
    ('ge-0/1/0', True),
    ('ge-1/1/0', True),
    ('xe-0/0/2', True),
    ('et-0/0/3', True),
    ('xe-0/0/1:0', True),
    ('et-0/0/0:1', True),
    ('et-0/0/0:2', True),
    ('et-0/0/0:3', True),
    ('et-0/0/0:4', True),
    ('ge-0/0/12xy', False),
    ('xxe-0/0/7', False),
    ('eth1/2', False),
    ('FastEthernet0/0', False),
    ('GigabitEthernet0/1', False),
    ('', False),
    ('1234', False),
    (1234, False),
    ([], False),
    ((), False),
    ({}, False),
    (True, False),
    (False, False),
])
def test_juniper_interface_format(interface, result):
    """
    Test to validate that we are passing as a string a Juniper interface
    that adheres to the Junos OS naming convention
    """
    assert is_valid_juniper_interface_format(interface) == result
```

## References

- [Effective Python Testing With Pytest](https://realpython.com/pytest-python-testing/)