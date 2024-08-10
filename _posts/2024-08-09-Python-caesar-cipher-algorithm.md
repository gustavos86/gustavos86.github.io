---
title: Python Caesar Cipher algorithm
date: 2024-08-09 22:30:00 -0700
categories: [PYTHON, ALGORITHMS]
tags: [python, algorithms]     # TAG names should always be lowercase
---

![]({{ site.baseurl }}/images/services/python_logo.png)

**NOTE:** Running **Python 3.10.5** and **IPython 8.22.2** in my environment when I created this post.

## The Caesar Ciper

The Caesar Cipher is one of the most simplest cipher algorithms. It consists on replacing each letter of a given text by the following letter `key` number of times after that letter in the alphabet.

For example, let's say we want to replace `a` by `key=1` letters. Since `b` follows `a` after 1 place then we replace `a` with `b`. Easy enough, right?

So, we can say `a` -> `key=1` -> `b`.

We also can say that `a` -> `key=2` -> `c`. Because `c` is 2 places in the alphabet after `a`.

To give a better idea, this table shows the equivalent of every letter in the alphabet when shifted 1 time (`key=1`)

| alphabet |  shifted key=1 |
|----------|----------------|
| a        | b              |
| b        | c              |
| c        | d              |
| d        | e              |
| e        | f              |
| f        | g              |
| g        | h              |
| h        | i              |
| i        | j              |
| j        | k              |
| k        | l              |
| l        | m              |
| m        | n              |
| n        | o              |
| o        | p              |
| p        | q              |
| q        | r              |
| r        | s              |
| s        | t              |
| t        | u              |
| u        | v              |
| v        | w              |
| w        | x              |
| x        | y              |
| y        | z              |
| z        | a              |

If `key=2`, then this would be the table of equivalents

| alphabet |  shifted key=2 |
|----------|----------------|
| a        | c              |
| b        | d              |
| c        | e              |
| d        | f              |
| e        | g              |
| f        | h              |
| g        | i              |
| h        | j              |
| i        | k              |
| j        | l              |
| k        | m              |
| l        | n              |
| m        | o              |
| n        | p              |
| o        | q              |
| p        | r              |
| q        | s              |
| r        | t              |
| s        | u              |
| t        | v              |
| u        | w              |
| v        | x              |
| w        | y              |
| x        | z              |
| y        | a              |
| z        | b              |

If we want to encrypt `abcde` with `key=1`, the encrypted version would be `bcdef`. In case of `key=2`, the encrypted version would be `cdefg`

How to implement the **Caesar Cyper** in Python? I will explain 2 different approaches.

## Shift clockwise with % (module) operator

The first approach is relying on the Python `%` (module) operator. See the [documentation](https://docs.python.org/3.10/reference/expressions.html#binary-arithmetic-operations).

Based on the ASCII Table.

![]({{ site.baseurl }}/images/2024/07-29-Python-caesar-cipher-algorithm/01-ASCII-Table.png)

We see that the lowercase `a` is decimal **97** in ascii, while the uppercase `A` is **65**. We can confirm this in `ipython`

```python
$ ipython
...

In [1]: ord('a')
Out[1]: 97

In [2]: ord('A')
Out[2]: 65
```

For lowercase `z` its decimal equivalent is **122**, while for uppercase `Z` it is **90**

```python
In [3]: ord('z')
Out[3]: 122

In [4]: ord('Z')
Out[4]: 90
```

To create the offset we need to add `key` number of times to it.

```python
In [5]: key=1

In [6]: ord('a') + key
Out[6]: 98
```

And to convert from **Decimal** number back to **ascii character** we used the `chr()` function

```python
In [7]: chr(98)
Out[7]: 'b'
```

The most important part is how do we handle and offset to the `z` letter, for example

```python
In [8]: key=1

In [9]: ord('z') + key
Out[9]: 123

In [10]: chr(123)
Out[10]: '{'
```

We need to do some simple math using the Python `%` (module) operator.

```python
In [19]: key=1

In [20]: ((ord('z') + key - 97) % 26) + 97
Out[20]: 97

In [21]: chr(97)
Out[21]: 'a'
```

Where did those "magic numbers" come from? Let's replace them with proper variable nanes to help identifying them

```python
In [25]: key=1

In [26]: NUM_LETTERS_IN_ALPHABET = 26

In [27]: ascii_base = 97

In [28]: ((ord('z') + key - ascii_base) % NUM_LETTERS_IN_ALPHABET) + ascii_base
Out[28]: 97

In [29]: chr(97)
Out[29]: 'a'
```

We add the `key` offset and substract the ascii `a` value in decimal. We then make sure of the **module operator** to get to the starting point of the alphabet. Lastly we add the ascii value of `a` in decimal.

Having this is the most important part of this algorithm. Based on it we can put together a simple function that has a variable `ascii_base` with different values depending if the letter is **lowercase** or **uppercase**.

The complete implementation of the code: 

```python
def cesar_encrypt(text: str, key: int) -> str:
    """
    text :str is the text to encrypt
    key: int is the offset to encrypt the letters

    Returns the encrypted string
    """
    def rotate_ascii(letter: str, key: int, ascii_base: int):
        NUM_LETTERS_IN_ALPHABET = 26
        return chr(((ord(letter) + key - ascii_base) % NUM_LETTERS_IN_ALPHABET) + ascii_base)

    encrypted_text = list()

    for letter in text:
        ascii_base = 0
        if letter.islower():
            ascii_base = 97
        elif letter.isupper():
            ascii_base = 65
        
        if ascii_base:
            encrypted_text += rotate_ascii(letter, key, ascii_base)
        else:
            encrypted_text += letter

    return "".join(encrypted_text)
```

Testing manually this implementation

```python
In [3]: cesar_encrypt("Hello World", 1)
Out[3]: 'Ifmmp Xpsme'

In [4]: cesar_encrypt("Hello World", 10)
Out[4]: 'Rovvy Gybvn'

In [5]: cesar_encrypt("Programming in Python", 1)
Out[5]: 'Qsphsbnnjoh jo Qzuipo'

In [6]: cesar_encrypt("Programming in Python", 5)
Out[6]: 'Uwtlwfrrnsl ns Udymts'
```

Finally to decrypt, we use a slight variation.

We replace this line

```python
return chr(((ord(letter) + key - ascii_base) % NUM_LETTERS_IN_ALPHABET) + ascii_base)
```

with this line

```python
return chr(((ord(letter) - key - ascii_base) % NUM_LETTERS_IN_ALPHABET) + ascii_base)
```

Note that the only difference is that we substract `key` instaead of adding it.

Complete code for Cesar decrypt

```python
def cesar_decrypt(text: str, key: int) -> str:
    """
    text :str is the text to decrypt
    key: int is the offset to decrypt the letters

    Returns the decrypted string
    """
    def rotate_ascii(letter: str, key: int, ascii_base: int):
        NUM_LETTERS_IN_ALPHABET = 26
        return chr(((ord(letter) - key - ascii_base) % NUM_LETTERS_IN_ALPHABET) + ascii_base)

    decrypted_text = list()

    for letter in text:
        ascii_base = 0
        if letter.islower():
            ascii_base = 97
        elif letter.isupper():
            ascii_base = 65
        
        if ascii_base:
            decrypted_text += rotate_ascii(letter, key, ascii_base)
        else:
            decrypted_text += letter

    return "".join(decrypted_text)
```

Some manual testing and we are done.

```python
In [14]: cesar_decrypt('Ifmmp Xpsme', 1)
Out[14]: 'Hello World'

In [15]: cesar_decrypt('Rovvy Gybvn', 10)
Out[15]: 'Hello World'

In [16]: cesar_decrypt('Qsphsbnnjoh jo Qzuipo', 1)
Out[16]: 'Programming in Python'

In [17]:

In [17]: cesar_decrypt('Uwtlwfrrnsl ns Udymts', 5)
Out[17]: 'Programming in Python'
```

## Create a mapping table

Another way to do a Caesar Cipher is creating a mapping table in a HashMap (in Python, this would be a dictionary) so we can lookup the equivalente of each character.

We will see first how to do this "manually" and after how it can be done using the **string module** and **string methods**

The steps are the following

1. Have the complete alphabet in order
2. Shift the alphabet `key` values to the right and store it in a new variable
3. Create a mapping table (a `Dict` in Python)
4. In a loop, lookup each letter of the alphabet in the `Dict` to obtain its encrypted equivalent

### Manually

We need to manually create the alphabet

```python
In [1]: alphabet = "abcdefghijklmnopqrstuvwxyz"
```

To shift all the letters in the alphabet we use of Python Slicing. We store the result in a new variable

```python
In [2]: key = 1

In [3]: shifted = alphabet[key:] + alphabet[:key]

In [4]: shifted
Out[4]: 'bcdefghijklmnopqrstuvwxyza'
```

We would need to manually create the mapping table using a loop

```python
In [5]: table = dict()

In [9]: table
Out[9]: {}

In [10]: for idx in range(len(alphabet)):
    ...:     table[alphabet[idx]] = shifted[idx]
    ...:

In [11]: table
Out[11]:
{'a': 'b',
 'b': 'c',
 'c': 'd',
 'd': 'e',
 'e': 'f',
 'f': 'g',
 'g': 'h',
 'h': 'i',
 'i': 'j',
 'j': 'k',
 'k': 'l',
 'l': 'm',
 'm': 'n',
 'n': 'o',
 'o': 'p',
 'p': 'q',
 'q': 'r',
 'r': 's',
 's': 't',
 't': 'u',
 'u': 'v',
 'v': 'w',
 'w': 'x',
 'x': 'y',
 'y': 'z',
 'z': 'a'}
```

To encrypt the text, we will run another loop and lookup each of the characters in the mapping table. We store the result in a Python `List`

```python
In [13]: text = "hello world"

In [14]: encrypted = list()

In [15]: for character in text:
    ...:     encrypted.append(table.get(character, character))
    ...:

In [16]: encrypted
Out[16]: ['i', 'f', 'm', 'm', 'p', ' ', 'x', 'p', 's', 'm', 'e']
```

We finally convert the Python `List` into a **string**

```python
In [17]: "".join(encrypted)
Out[17]: 'ifmmp xpsme'
```

The `table.get(character, character)` may seem weird but it is actually intended. If the `key` is not found in the Dictionary it will insert a default value. In this caes the original character

The complete implementation, consider lowercase and uppercase in the text

```python
In [1]: from typing import Dict

In [2]: def cesar_encrypt(text: str, key: int) -> str:
   ...:     """
   ...:     text :str is the text to encrypt
   ...:     key: int is the offset to encrypt the letters
   ...:
   ...:     Returns the encrypted string
   ...:     """
   ...:     def create_map_table(alphabet: str, shifted: str) -> Dict:
   ...:         table = dict()
   ...:         for idx in range(len(alphabet)):
   ...:             table[alphabet[idx]] = shifted[idx]
   ...:
   ...:         return table
   ...:
   ...:     alphabet_lowercase = "abcdefghijklmnopqrstuvwxyz"
   ...:     alphabet_uppercase = "ABCDEFGHIJKLMNOPQRSTUVWXYZ"
   ...:
   ...:     shifted_lowercase = alphabet_lowercase[key:] + alphabet_lowercase[:key]
   ...:     shifted_uppercase = alphabet_uppercase[key:] + alphabet_uppercase[:key]
   ...:
   ...:     table = create_map_table(alphabet_lowercase, shifted_lowercase)
   ...:     table_uppercase = create_map_table(alphabet_uppercase, shifted_uppercase)
   ...:
   ...:     table.update(table_uppercase)  # Merge the 2 tables
   ...:
   ...:     encrypted = list()
   ...:
   ...:     for character in text:
   ...:         encrypted.append(table.get(character, character))
   ...:
   ...:     return "".join(encrypted)
   ...:

In [3]: cesar_encrypt("Hello World", 1)
Out[3]: 'Ifmmp Xpsme'

In [4]: cesar_encrypt("Hello World", 10)
Out[4]: 'Rovvy Gybvn'

In [5]: cesar_encrypt("Programming in Python", 1)
Out[5]: 'Qsphsbnnjoh jo Qzuipo'

In [6]: cesar_encrypt("Programming in Python", 5)
Out[6]: 'Uwtlwfrrnsl ns Udymts'
```

### String module and string methods

With this approach, we first need to import the **string** module

```python
In [1]: import string
```
Now, using the **string** mdoule, we create an alphabet

```python
In [2]: alphabet = string.ascii_lowercase

In [3]: alphabet
Out[3]: 'abcdefghijklmnopqrstuvwxyz'
```

**NOTE:** There is an equivalent constante `string.ascii_uppercase`

To shift all the letters in the alphabet, we can again make use of Python Slicing

```python
In [4]: key = 1

In [5]: shifted = alphabet[key:] + alphabet[:key]

In [6]: shifted
Out[6]: 'bcdefghijklmnopqrstuvwxyza'
```

Now, we can create the mapping table using the `maketrans` string method

```python
In [7]: table = str.maketrans(alphabet, shifted)

In [8]: table
Out[8]:
{97: 98,
 98: 99,
 99: 100,
 100: 101,
 101: 102,
 102: 103,
 103: 104,
 104: 105,
 105: 106,
 106: 107,
 107: 108,
 108: 109,
 109: 110,
 110: 111,
 111: 112,
 112: 113,
 113: 114,
 114: 115,
 115: 116,
 116: 117,
 117: 118,
 118: 119,
 119: 120,
 120: 121,
 121: 122,
 122: 97}

In [9]: type(table)
Out[9]: dict
```

Note the table is created using actually the **ASCII values** of the letters

We finally make use of another string method called `translate` to encrypt the text

```python
In [12]: text = "hello world"

In [13]: encrypted = text.translate(table)

In [14]: encrypted
Out[14]: 'ifmmp xpsme'
```

The complete implementation, consider lowercase and uppercase in the text

```python
In [25]: import string

In [26]: def cesar_encrypt(text: str, key: int) -> str:
    ...:     """
    ...:     text :str is the text to encrypt
    ...:     key: int is the offset to encrypt the letters
    ...:
    ...:     Returns the encrypted string
    ...:     """
    ...:     alphabet_lowercase = string.ascii_lowercase
    ...:     alphabet_uppercase = string.ascii_uppercase
    ...:
    ...:     shifted_lowercase = alphabet_lowercase[key:] + alphabet_lowercase[:key]
    ...:     shifted_uppercase = alphabet_uppercase[key:] + alphabet_uppercase[:key]
    ...:
    ...:     table = str.maketrans(alphabet_lowercase, shifted_lowercase)
    ...:     table_uppercase = str.maketrans(alphabet_uppercase, shifted_uppercase)
    ...:
    ...:     table.update(table_uppercase)  # Merge the 2 tables
    ...:
    ...:     return text.translate(table)
    ...:

In [27]: cesar_encrypt("Hello World", 1)
Out[27]: 'Ifmmp Xpsme'

In [28]: cesar_encrypt("Hello World", 10)
Out[28]: 'Rovvy Gybvn'

In [29]: cesar_encrypt("Programming in Python", 1)
Out[29]: 'Qsphsbnnjoh jo Qzuipo'

In [30]: cesar_encrypt("Programming in Python", 5)
Out[30]: 'Uwtlwfrrnsl ns Udymts'
```

## One more implementation

The best would be to test against all cases

```python
def run_unit_testing():
    assert cesar_encrypt("abc", 1) == "bcd"  # case 1 - lower case letters
    assert cesar_encrypt("ABC", 1) == "BCD"  # case 2 - upper case letters
    assert cesar_encrypt("xyz", 3) == "abc"  # case 3 - rotation
    assert cesar_encrypt("XyZ", 3) == "AbC"  # case 4 - mix of previous cases
    assert cesar_encrypt("!a #7", 2) == "!c #7"  # case 5 - non alphabet characters
    assert cesar_encrypt("Δα", 2) == "Δα"
    print("All tests completed successfully!")
```

And with this, we can see the latest implementation is not entirely working when passing greek characters

```python
In [7]: def run_unit_testing():
   ...:     assert cesar_encrypt("abc", 1) == "bcd"  # case 1 - lower case letters
   ...:     assert cesar_encrypt("ABC", 1) == "BCD"  # case 2 - upper case letters
   ...:     assert cesar_encrypt("xyz", 3) == "abc"  # case 3 - rotation
   ...:     assert cesar_encrypt("XyZ", 3) == "AbC"  # case 4 - mix of previous cases
   ...:     assert cesar_encrypt("!a #7", 2) == "!c #7"  # case 5 - non alphabet characters
   ...:     assert cesar_encrypt("Δα", 2) == "Δα"
   ...:     print("All tests completed successfully!")
   ...:

In [8]: run_unit_testing()
---------------------------------------------------------------------------
AssertionError                            Traceback (most recent call last)
Cell In[8], line 1
----> 1 run_unit_testing()

Cell In[7], line 7, in run_unit_testing()
      5 assert cesar_encrypt("XyZ", 3) == "AbC"  # case 4 - mix of previous cases
      6 assert cesar_encrypt("!a #7", 2) == "!c #7"  # case 5 - non alphabet characters
----> 7 assert cesar_encrypt("Δα", 2) == "Δα"
      8 print("All tests completed successfully!")

AssertionError:

In [9]:
```

I actually think this is an even better implementation:

```python
import string

def cesar_encrypt(text: str, key: int) -> str:
    """
    text :str is the text to encrypt
    key: int is the offset to encrypt the letters

    Returns the encrypted string
    """
    alphabet_lowercase = string.ascii_lowercase
    alphabet_uppercase = string.ascii_uppercase

    mapping_lowercase = dict(enumerate(alphabet_lowercase))
    mapping_uppercase = dict(enumerate(alphabet_uppercase))

    encrypted = list()

    for letter in text:
        if letter in alphabet_lowercase or letter in alphabet_uppercase:
            if letter.islower():
                idx = (alphabet_lowercase.index(letter) + key) % len(alphabet_lowercase)
                encrypted += mapping_lowercase[idx]
            elif letter.isupper():
                idx = (alphabet_uppercase.index(letter) + key) % len(alphabet_uppercase)
                encrypted += mapping_uppercase[idx]
        else:
            encrypted += letter

    return "".join(encrypted)
```

Testing it with `iPython`

```python
C:\Users\user>ipython
Python 3.10.5 (tags/v3.10.5:f377153, Jun  6 2022, 16:14:13) [MSC v.1929 64 bit (AMD64)]
Type 'copyright', 'credits' or 'license' for more information
IPython 8.22.2 -- An enhanced Interactive Python. Type '?' for help.

In [1]: import string
   ...:
   ...: def cesar_encrypt(text: str, key: int) -> str:
   ...:     """
   ...:     text :str is the text to encrypt
   ...:     key: int is the offset to encrypt the letters
   ...:
   ...:     Returns the encrypted string
   ...:     """
   ...:     alphabet_lowercase = string.ascii_lowercase
   ...:     alphabet_uppercase = string.ascii_uppercase
   ...:
   ...:     mapping_lowercase = dict(enumerate(alphabet_lowercase))
   ...:     mapping_uppercase = dict(enumerate(alphabet_uppercase))
   ...:
   ...:     encrypted = list()
   ...:
   ...:     for letter in text:
   ...:         if letter in alphabet_lowercase or letter in alphabet_uppercase:
   ...:             if letter.islower():
   ...:                 idx = (alphabet_lowercase.index(letter) + key) % len(alphabet_lowercase)
   ...:                 encrypted += mapping_lowercase[idx]
   ...:             elif letter.isupper():
   ...:                 idx = (alphabet_uppercase.index(letter) + key) % len(alphabet_uppercase)
   ...:                 encrypted += mapping_uppercase[idx]
   ...:         else:
   ...:             encrypted += letter
   ...:
   ...:     return "".join(encrypted)
   ...:

In [2]: def run_unit_testing():
   ...:     assert cesar_encrypt("abc", 1) == "bcd"  # case 1 - lower case letters
   ...:     assert cesar_encrypt("ABC", 1) == "BCD"  # case 2 - upper case letters
   ...:     assert cesar_encrypt("xyz", 3) == "abc"  # case 3 - rotation
   ...:     assert cesar_encrypt("XyZ", 3) == "AbC"  # case 4 - mix of previous cases
   ...:     assert cesar_encrypt("!a #7", 2) == "!c #7"  # case 5 - non alphabet characters
   ...:     assert cesar_encrypt("Δα", 2) == "Δα"
   ...:     print("All tests completed successfully!")
   ...:

In [3]: run_unit_testing()
All tests completed successfully!

In [4]:
```

Win, win, win!

## References

- [Caesar cipher](https://en.wikipedia.org/wiki/Caesar_cipher)
- [Simple Caesar Encryption in Python](https://www.youtube.com/watch?v=JEsUlx0Ps9k)
- [How to Encrypt Data Using Caesar Cipher in Python (Simple)](https://www.youtube.com/watch?v=REStjp3sP4s)
- [Python String maketrans() Method](https://www.w3schools.com/python/ref_string_maketrans.asp)
- [Python String translate() Method](https://www.w3schools.com/python/ref_string_translate.asp)
- [Python Merge Dictionaries – Merging Two Dicts in Python](https://www.freecodecamp.org/news/python-merge-dictionaries-merging-two-dicts-in-python/)