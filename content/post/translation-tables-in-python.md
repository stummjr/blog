---
title: "Translation Tables In Python"
author: "Valdir Stumm Jr"
tags: ["python", "string"]
date: 2022-03-16T15:55:13-03:00
author: Valdir Stumm Jr
draft: false
---

Back when I was a teacher, I used to ask my students to implement a
[Caeser Cipher](https://en.wikipedia.org/wiki/Caesar_cipher) encoder/decoder.
If you're not familiar with it, it is a simple substitution cipher that replaces
characters on a string by the corresponding characters from a shifted alphabet.

The algorithm takes a numerical cipher `n` as its input (the key) and then rotates the letters
in the alphabet by `n` characters. Then, a translation table is created by lining up the original
and the rotated alphabets. For example, if we use `2` as the key, we get this translation table:

```
a b c d e f g h i j k l m n o p q r s t u v w x y z
c d e f g h i j k l m n o p q r s t u v w x y z a b
```

So, if we were to encode the message `hello` using the key `2`, we'd get `jgnnq` as a result.
To decode `jgnnq` back, we'll use a reverse table:

```
c d e f g h i j k l m n o p q r s t u v w x y z a b
a b c d e f g h i j k l m n o p q r s t u v w x y z
```

The algorithm is simple and there are many possible ways to implement it. But as a Pythonista,
my gut tells me that there must be something in the standard library to help me here.


## `maketrans` to the rescue
The Python standard library allows to create a **translation table** that we can later use when
translating strings. That's what the [`str.maketrans`](https://docs.python.org/3/library/stdtypes.html#str.maketrans)
is for.

Let's say we want to quickly create a translation table to encode strings using the Caesar Cipher,
having `2` as the key.

First of all, we'll create a translation table using `str.maketrans`,
passing in the original alphabet and the rotated one:

```python
>>> encoding_table = str.maketrans("abcdefghijklmnopqrstuvwxyz", "cdefghijklmnopqrstuvwxyzab")
>>> encoding_table
{97: 99,
 98: 100,
 .
 .
 .
 120: 122,
 121: 97,
 122: 98}
 ```

As you can see above, all it does is to create a mapping between the ASCII value of each character
from the source to the target alphabet (remember that `ord("a") == 97`, and so on).

Once we have the translation table in hand, we can encode our text via the
[`translate`](https://docs.python.org/3/library/stdtypes.html#str.translate) method of
Python string objects:

```python
>>> "hello".translate(encoding_table)
'jgnnq'
```

And if we want to decode the text back to its original value, we can use a reverse table:

```python
>>> decoding_table = str.maketrans("cdefghijklmnopqrstuvwxyzab", "abcdefghijklmnopqrstuvwxyz")
>>> "jgnnq".translate(decoding_table)
'hello'
```

## Generalizing it
We can dynamically create these tables using another great Python builtin feature: **slicing**.

Let's say that we want to rotate the alphabet `2` letters to the right. All we need is to break
the alphabet in two parts and join them. For example, `abcdefghijklmnopqrstuvwxyz` becomes
`cdefghijklmnopqrstuvwxyz + ab`. In Python, we can do this:

```python
>>> from string import ascii_lowercase as alphabet
>>> alphabet
'abcdefghijklmnopqrstuvwxyz'
>>> alphabet[2:]
'cdefghijklmnopqrstuvwxyz'
>>> alphabet[:2]
'ab'
>>> alphabet[2:] + alphabet[:2]
'cdefghijklmnopqrstuvwxyzab'
```

As you can see above, we are taking two slices of the alphabet:
- one starting from the third letter of the alphabet: `alphabet[2:]`.
- the other with just the first two letters: `alphabet[:2]`.

Now we can write utilities to build the encoding/decoding tables for us, given a key:

```python
from string import ascii_lowercase as alphabet


def build_encoding_table(key):
    return str.maketrans(alphabet, alphabet[key:] + alphabet[:key])

def build_decoding_table(key):
    return str.maketrans(alphabet[key:] + alphabet[:key], alphabet)
```

And then we can encode/decode strings using the `.translate` method and the proper
translation tables.


## Building an abstraction
Given what we've seen above, we can now create a class to encode/decode strings using
Caesar Cipher. Something like this:

```python
from string import ascii_lowercase as alphabet


class CaesarCipher:

    def __init__(self, key):
        key = key % len(alphabet)
        rotated_alphabet = alphabet[key:] + alphabet[:key]
        self.encoding_table = str.maketrans(alphabet, rotated_alphabet)
        self.decoding_table = str.maketrans(rotated_alphabet, alphabet)

    def encode(self, text):
        return text.translate(self.encoding_table)

    def decode(self, text):
        return text.translate(self.decoding_table)
```

The code above is just a simplification to show how we can take advantage of the standard library to simplify
solutions. It does not handle uppercase/numbers/special characters and possibly other scenarios.


## Testing it all
We want to make sure it all works as expected and for that we can add some unit tests:

```python
import pytest
from caesar import CaesarCipher


@pytest.mark.parametrize("key, input_text, output_text", [
    (0, "hello", "hello"),
    (3, "hello", "khoor"),
    (29, "hello", "khoor"),
    (-3, "hello", "ebiil"),
    (-29, "hello", "ebiil"),
])
def test_encode(key, input_text, output_text):
    assert CaesarCipher(key=key).encode(input_text) == output_text


@pytest.mark.parametrize("key, input_text, output_text", [
    (0, "hello", "hello"),
    (3, "khoor", "hello"),
    (29, "khoor", "hello"),
    (-3, "ebiil", "hello"),
    (-29, "ebiil", "hello"),
])
def test_decode(key, input_text, output_text):
    assert CaesarCipher(key=key).decode(input_text) == output_text
```

If you're not familiar with Pytest parametrization, check this out: https://docs.pytest.org/en/latest/parametrize.html


## That's all for today
I am pretty sure that there are a lot of corners in the standard library that are not used very often,
but that can be very helpful. What's your favorite?
