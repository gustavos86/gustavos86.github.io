---
title: Python Reverse a String
date: 2024-07-28 19:30:00 -0700
categories: [PYTHON, ALGORITHMS]
tags: [python, algorithms]     # TAG names should always be lowercase
---

![]({{ site.baseurl }}/images/services/python_logo.png)

**NOTE:** Running **Python 3.10.5** and **IPython 8.22.2** in my environment when I created this post.

## Reverse a String the Pythonic way

Reverse a **String** (`str`) in Python is very, very... very simple.

Here we use `ipython` to test this concept. We create the variable the `text` and assigned the `str` with value "Hello World" to it.

```python
$ ipython
...

In [1]: text = "hello world"

In [2]:
```

Next, we make use of [Python Slicing](https://docs.python.org/3.10/whatsnew/2.3.html#extended-slices) and the **third** argument that can eb used since **Python 1.4**

```python
In [2]: text[::-1]
Out[2]: 'dlrow olleh'
```

See? That was veeery simple.

## Reverse a String the non-Pythonic way

Now, let's say that for some reason, we don't want to use [Python Slicing](https://docs.python.org/3.10/whatsnew/2.3.html#extended-slices) to reverse an string. In such case, we would need to reverse a string executing a series of steps... so, an algorithm. Luckily, this isn't that hard.

We would need to track the **initial** and **last** letters of the `str`

```python
In [3]: initial_index = 0

In [4]: last_index = len(text) -1
```

At first glance, and sticking to the `"hello world"` we have a pointer to the first letter (`h`) and the last letter (`d`).


```python
In [5]: text[initial_index]
Out[5]: 'h'

In [6]: text[last_index]
Out[6]: 'd'
```

Once we try to swap these letters directly in the `text` variable, we get an error... an expected one since **strings** are **inmutable** in Python, meaning we cannot modify them.

```python
In [7]: text[initial_index] = text[last_index]
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
Cell In[7], line 1
----> 1 text[initial_index] = text[last_index]

TypeError: 'str' object does not support item assignment
```

What do we do? Based on the `text` **string**, create a **Python List**. Here how to do it:

```python
In [8]: text_list = list(text)

In [9]: text_list
Out[9]: ['h', 'e', 'l', 'l', 'o', ' ', 'w', 'o', 'r', 'l', 'd']
```

Great, we now have the same **string**, but as a **Python List** which we can modify. Let's try again:


```python
In [9]: text_list
Out[9]: ['h', 'e', 'l', 'l', 'o', ' ', 'w', 'o', 'r', 'l', 'd']

In [10]: text_list[initial_index] = text[last_index]

In [11]: text_list
Out[11]: ['d', 'e', 'l', 'l', 'o', ' ', 'w', 'o', 'r', 'l', 'd']
```

We did not get any error this time, so that is good. However, we did not really *swap* the first and last letters, but we just copied the value of the last letter into the first one.

Let's make `test_list` have its original value.


```python
In [12]: text_list = list(text)

In [13]: text_list
Out[13]: ['h', 'e', 'l', 'l', 'o', ' ', 'w', 'o', 'r', 'l', 'd']
```

The correct way to *swap* these in Python would be by relying on **Python unpacking**

```python
In [15]: text_list[initial_index], text_list[last_index] = text_list[last_index], text_list[initial_index]

In [17]: text_list
Out[17]: ['d', 'e', 'l', 'l', 'o', ' ', 'w', 'o', 'r', 'l', 'h']
```

We really *swaped* the first and last letters now. So we keep doing this... with a **loop** of course.

What would be the correct steps?

1. We should increase `initial_index` by 1 (so we have access to the second letter - `e`)
2. And we should decrease `last_index` by 1 (so we have access to the one before the last letter - `l`)
3. We *swap* them again.

We reapeat... until when? There are 2 cases to consider:

1. The `str` has a odd number of characters, i.e. `abc`
2. The `str` has an even number of characters, i.e.  `abcd`

Now, let's imagine we are reversing the first case, the odd number of charactesr in **string** `abc`

1. `initial_index` points to `a`
2. `last_index` points to `c`
3. We swap them
4. We **increase** `initial_index` by 1 and **decrease** `last_index` by 1
5. `initial_index` points to `b`
6. `last_index` points to `b`
7. Since both point to `b` we should just not swap anymore.

So the condition to **stop** is that `initial_index` should NOT be equal to `last_index`.

Let's go now over the seconds case, the even number of characters in **string** `abcd`

1. `initial_index` points to `a`
2. `last_index` points to `d`
3. We swap them
4. We **increase** `initial_index` by 1 and **decrease** `last_index` by 1
5. `initial_index` points to `b`
6. `last_index` points to `c`
7. We swap them
8. We **increase** `initial_index` by 1 and **decrease** `last_index` by 1
9. `initial_index` points to `c`
10. `last_index` points to `b`
11. We already swaped them! so we should just stop now.

So the condition to **stop** is that `initial_index` should NOT be larger than `last_index`.

Considering both cases, `initial_index` should NOT be equal and shold NOT be larger than `last_index`. In other words, `initial_index` should ALWAYS be LESS than `last_index`

We can do this in Python using a `while` loop

```python
In [18]: text_list = list(text)

In [19]: while(initial_index < last_index):
    ...:     text_list[initial_index], text_list[last_index] = text_list[last_index], text_list[initial_index]
    ...:     initial_index += 1
    ...:     last_index -= 1
    ...:

In [20]: text_list
Out[20]: ['d', 'l', 'r', 'o', 'w', ' ', 'o', 'l', 'l', 'e', 'h']
```

This worked! Now if we want do get this result as a string, we should make use of the **string** method `join`.

```python
In [20]: text_list
Out[20]: ['d', 'l', 'r', 'o', 'w', ' ', 'o', 'l', 'l', 'e', 'h']

In [21]: "".join(text_list)
Out[21]: 'dlrow olleh'
```

Putting all together, here we have the code inside a function.

```python
def reverse_string(text: str) -> str:
    """
    text :str is the String to be reversed
    
    Returns :str
    A copy of the original string in reverse order
    """
    if len(text) <= 1:
        return text

    text_list = list(text)

    initial_index = 0
    last_index = len(text) -1

    while(initial_index < last_index):
        text_list[initial_index], text_list[last_index] = text_list[last_index], text_list[initial_index]

        initial_index += 1
        last_index -= 1

    return "".join(text_list)
```

Notice we added a couple of lines of code at the beginning of the function.

```python
if len(text) <= 1:
    return text
```

These are necessary in order to handle when the `str` is either of size 1 (nothing to reverse) or size zero (basically an empty string).

Here we see it running in `ipython`

```python
$ ipython
...

In [1]: def reverse_string(text: str) -> str:
   ...:     """
   ...:     text :str is the String to be reversed
   ...:
   ...:     Returns :str
   ...:     A copy of the original string in reverse order
   ...:     """
   ...:     if len(text) <= 1:
   ...:         return text
   ...:
   ...:     text_list = list(text)
   ...:
   ...:     initial_index = 0
   ...:     last_index = len(text) -1
   ...:
   ...:     while(initial_index < last_index):
   ...:         text_list[initial_index], text_list[last_index] = text_list[last_index], text_list[initial_index]
   ...:
   ...:         initial_index += 1
   ...:         last_index -= 1
   ...:
   ...:     return "".join(text_list)
   ...:

In [2]:  reverse_string("hello world")
Out[2]: 'dlrow olleh'

In [3]: reverse_string("abc")
Out[3]: 'cba'

In [4]: reverse_string("abcd")
Out[4]: 'dcba'

In [5]: reverse_string("a")
Out[5]: 'a'

In [6]: reverse_string("")
Out[6]: ''
```

## Algorithm to detect a Palindrome

What is a **Palindrome**? According to [Wikipedia](https://en.wikipedia.org/wiki/Palindrome): *A palindrome is a word, number, phrase, or other sequence of symbols that reads the same backwards as forwards, such as madam or racecar*.
So, we can use most of the code in the `reverse_string` algorithm to make it detect a word or phrase which is a Palindrome. Why? Because we already have code that traverses the word starting from the edges all the way to the middle.

```python
initial_index = 0
last_index = len(text) -1

while(initial_index < last_index):
    text_list[initial_index], text_list[last_index] = text_list[last_index], text_list[initial_index]

    initial_index += 1
    last_index -= 1
```

If we slighly modify it and **instead of swapping** the letter, it just verified they are the **same letter**, we would have the main code to check for **Palindrome** words.

```python
initial_index = 0
last_index = len(text) -1

while(initial_index < last_index):
    if text_list[initial_index] == text_list[last_index]
        initial_index += 1
        last_index -= 1
    else:
        # We should break as the word is NOT a palindrome
```

The full code would be:

```python
def is_palindrome(text: str) -> bool:
    """
    text :str is the String

    Returns :bool
    True if text is palindrome, False otherwise
    """
    if len(text) <= 1:
        return True

    text_list = list(text)

    initial_index = 0
    last_index = len(text) -1

    while(initial_index < last_index):
        if text_list[initial_index] == text_list[last_index]:
            initial_index += 1
            last_index -= 1
        else:
            return False

    return True
```

Note one more slight modifications, the function now returns a `bool` either `True` or `False` as opposed to the reversed string

Testing the code manually in `ipython`

```python
$ ipython
...

In [2]: def is_palindrome(text: str) -> bool:
   ...:     """
   ...:     text :str is the String
   ...:
   ...:     Returns :bool
   ...:     True if text is palindrome, False otherwise
   ...:     """
   ...:     if len(text) <= 1:
   ...:         return True
   ...:
   ...:     text_list = list(text)
   ...:
   ...:     initial_index = 0
   ...:     last_index = len(text) -1
   ...:
   ...:     while(initial_index < last_index):
   ...:         if text_list[initial_index] == text_list[last_index]:
   ...:             initial_index += 1
   ...:             last_index -= 1
   ...:         else:
   ...:             return False
   ...:
   ...:     return True
   ...:

In [3]: is_palindrome("hello world")
Out[3]: False

In [4]: is_palindrome("abc")
Out[4]: False

In [5]: is_palindrome("abcba")
Out[5]: True

In [6]: is_palindrome("XabcbaX")
Out[6]: True

In [7]: is_palindrome("XabcbaX_")
Out[7]: False

In [8]: is_palindrome("a")
Out[8]: True

In [9]: is_palindrome("")
Out[9]: True
```

Thanks for reading!