---
title: Python Threading examples
date: 2024-07-21 20:00:00 -0700
categories: [PYTHON, THREADING]
tags: [python]     # TAG names should always be lowercase
---

![]({{ site.baseurl }}/images/services/python_logo.png)

## ThreadPoolExecutor

The code creates a `ThreadPoolExecutor` as a **Context Manager**. It then uses `.map()` which takes a **function** and an iterable of things. `.map()` steps through an iterable of things, in this case `network_device_hostnames: list[str]`, passing each one to a thread in the pool.

```python
import concurrent.futures

def example_of_threading(network_device_hostnames: list[str]) -> list:
    """
    :param list[str] network_device_hostnames: list of hostnames of network devices

    :return: a Generator where each element is information about the network device
    """
    def get_network_device_info(hostname: str) -> str:
        """
        some logic here
        """
        return ""

    with concurrent.futures.ThreadPoolExecutor() as executor:
        results = executor.map(get_network_device_info, network_device_hostnames)

    return results
```

The most interesting code here is the **Context Manager** using `concurrent.futures.ThreadPoolExecutor`.

```python
    with concurrent.futures.ThreadPoolExecutor() as executor:
        results = executor.map(get_network_device_info, network_device_hostnames)
```

**NOTE:** The generator `result` can easily be converted to a Python list with `list(results)`

### Proof of Concept

First, using `ipython`, I will copy the code with a few additions to see the times.

```python
Python 3.9.6 (default, Nov 10 2023, 13:38:27)
Type 'copyright', 'credits' or 'license' for more information
IPython 8.18.1 -- An enhanced Interactive Python. Type '?' for help.

In [1]: import concurrent.futures
   ...: import datetime
   ...: from time import sleep
   ...:
   ...:
   ...: def example_of_threading_returning_a_list(network_device_hostnames: list[str]) -> list:
   ...:     """
   ...:     :param list[str] network_device_hostnames: list of hostnames of network devices
   ...:
   ...:     :return: a Python List where each element is information about the network device
   ...:     """
   ...:     def get_network_device_info(hostname: str):
   ...:         """
   ...:         some logic here
   ...:         """
   ...:         current_time_start = datetime.datetime.now()
   ...:         print(f"start_{hostname}_{current_time_start}")
   ...:
   ...:         sleep(10)
   ...:
   ...:         current_time_end = datetime.datetime.now()
   ...:         print(f"end_{hostname}_{current_time_end}")
   ...:
   ...:         return f"{hostname}_result"
   ...:
   ...:     with concurrent.futures.ThreadPoolExecutor() as executor:
   ...:         results = executor.map(get_network_device_info, network_device_hostnames)
   ...:
   ...:     return results
```

I will simulate a few network device hostnames in a Python list

```python
In [2]: network_device_hostnames = [
   ...:   "device0",
   ...:   "device1",
   ...:   "device2",
   ...:   "device3",
   ...:   "device4",
   ...:   "device5",
   ...:   "device6",
   ...:   "device7",
   ...:   "device8",
   ...:   "device9",
   ...: ]

In [3]:
```

I will finally call the function `example_of_threading_returning_a_list` and store its return value in a variable called `results`

```python
In [3]: results = example_of_threading_returning_a_list(network_device_hostnames)
start_device0_2024-07-21 20:31:46.958015
start_device1_2024-07-21 20:31:46.959030
start_device2_2024-07-21 20:31:46.960256
start_device3_2024-07-21 20:31:46.960464
start_device4_2024-07-21 20:31:46.960626
start_device5_2024-07-21 20:31:46.960786
start_device6_2024-07-21 20:31:46.960956
start_device7_2024-07-21 20:31:46.961154
start_device8_2024-07-21 20:31:46.961328
start_device9_2024-07-21 20:31:46.961486
end_device1_2024-07-21 20:31:56.961024
end_device2_2024-07-21 20:31:56.961074
end_device3_2024-07-21 20:31:56.961132
end_device0_2024-07-21 20:31:56.961107
end_device4_2024-07-21 20:31:56.965011
end_device6_2024-07-21 20:31:56.965056
end_device8_2024-07-21 20:31:56.965097
end_device5_2024-07-21 20:31:56.965178
end_device7_2024-07-21 20:31:56.965157
end_device9_2024-07-21 20:31:56.965128

In [4]:
```

We can see that the function called to each of the 10 network devices, we waited 10 seconds (simulation **I/O bound**) and we finally got our result back. The result is actually of type `generator`.

```python
In [4]: type(results)
Out[4]: generator

...

In [6]: next(results)
Out[6]: 'device0_result'

In [7]: next(results)
Out[7]: 'device1_result'

In [8]: next(results)
Out[8]: 'device2_result'

In [9]:
```

It can be easily converted to a Python List

```python
In [9]: list(results)
Out[9]:
['device3_result',
 'device4_result',
 'device5_result',
 'device6_result',
 'device7_result',
 'device8_result',
 'device9_result']

In [10]:
```

## Using zip() to create a Dictionary

We may want to have a Dictionary as a return value. We may want this so we can keep track of each inputs passed as a List to the function and the corresponding return values after calling `ThreadPoolExecutor`.
One way to accomplish this is using `zip()`. See the previous example modified.

```python
import concurrent.futures

def example_of_threading(network_device_hostnames: list[str]) -> list:
    """
    :param list[str] network_device_hostnames: list of hostnames of network devices

    :return: a Generator where each element is information about the network device
    """
    def get_network_device_info(hostname: str) -> str:
        """
        some logic here
        """
        return ""

    result_dict = dict()

    with concurrent.futures.ThreadPoolExecutor() as executor:
        for hostname, result in zip(network_device_hostnames, executor.map(get_network_device_info, network_device_hostnames)):
                result_dict[hostname] = result

    return result_dict
```

The most important code snippet now becomes:

```python
    result_dict = dict()

    with concurrent.futures.ThreadPoolExecutor() as executor:
        for hostname, result in zip(network_device_hostnames, executor.map(get_network_device_info, network_device_hostnames)):
                result_dict[hostname] = result
```

## Proof of Concept

We again make use of `ipython` to test this code.

```python
Python 3.9.6 (default, Nov 10 2023, 13:38:27)
Type 'copyright', 'credits' or 'license' for more information
IPython 8.18.1 -- An enhanced Interactive Python. Type '?' for help.

   ...:     """
   ...:     def get_network_device_info(hostname: str):
   ...:         """
   ...:         some logic here
   ...:         """
   ...:         current_time_start = datetime.datetime.now()
   ...:         print(f"start_{hostname}_{current_time_start}")
   ...:
   ...:         sleep(10)
   ...:
   ...:         current_time_end = datetime.datetime.now()
   ...:         print(f"end_{hostname}_{current_time_end}")
   ...:
   ...:         return f"{hostname}_result"
   ...:
   ...:     result_dict = dict()
   ...:
   ...:     with concurrent.futures.ThreadPoolExecutor() as executor:
   ...:         for hostname, result in zip(network_device_hostnames, executor.map(get_network_device_info, ne
   ...: twork_device_hostnames)):
   ...:                 result_dict[hostname] = result
   ...:
   ...:     return result_dict
   ...:
   ...:
   ...:
   ...: network_device_hostnames = [
   ...:   "device0",
   ...:   "device1",
   ...:   "device2",
   ...:   "device3",
   ...:   "device4",
   ...:   "device5",
   ...:   "device6",
   ...:   "device7",
   ...:   "device8",
   ...:   "device9",
   ...: ]

In [2]:
```

We call `example_of_threading_returning_a_list` and store the results in `result_dict`

```python
In [2]: result_dict = example_of_threading_returning_a_list(network_device_hostnames)
start_device0_2024-07-21 20:43:57.597709
start_device1_2024-07-21 20:43:57.598008
start_device2_2024-07-21 20:43:57.598161
start_device3_2024-07-21 20:43:57.598345
start_device4_2024-07-21 20:43:57.598520
start_device5_2024-07-21 20:43:57.598684
start_device6_2024-07-21 20:43:57.598884
start_device7_2024-07-21 20:43:57.599032
start_device8_2024-07-21 20:43:57.599181
start_device9_2024-07-21 20:43:57.599325
end_device0_2024-07-21 20:44:07.602269
end_device1_2024-07-21 20:44:07.602401
end_device6_2024-07-21 20:44:07.602535
end_device9_2024-07-21 20:44:07.602585
end_device2_2024-07-21 20:44:07.602754
end_device4_2024-07-21 20:44:07.602805
end_device7_2024-07-21 20:44:07.602650
end_device8_2024-07-21 20:44:07.602695
end_device3_2024-07-21 20:44:07.603248
end_device5_2024-07-21 20:44:07.602478
In [3]:
```

We can confirm the return value is a `dict` and we managed to store the inputs as **keys** and the results as **values** in this dictionary.

```python
In [3]: type(result_dict)
Out[3]: dict

In [4]: result_dict
Out[4]:
{'device0': 'device0_result',
 'device1': 'device1_result',
 'device2': 'device2_result',
 'device3': 'device3_result',
 'device4': 'device4_result',
 'device5': 'device5_result',
 'device6': 'device6_result',
 'device7': 'device7_result',
 'device8': 'device8_result',
 'device9': 'device9_result'}

In [5]:
```

## References

- [An Intro to Threading in Python](https://realpython.com/intro-to-python-threading/)