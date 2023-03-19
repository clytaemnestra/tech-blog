---
layout: post
title: "TIL: partial functions"
description: "TIL: partial functions"
date: 2023-03-19
tags: [python]
---

Today I discovered partial functions in Python. Partial functions are functions which are derived from general functions. The derived function has default parameters of the general function, unless it's overwritten.

Very simple example: 
```python
>>> from functools import partial
>>> def foo(a, b):
...     return a + b
... 
>>> bar = partial(foo, a=5) 
>>> bar(b=5)
10
>>> bar(a=10, b=10)
20
>>> 
```

Why do they exist and when to use them? The main reason to use them is to improve reusability. I found a real-world example on [Stack Overflow](https://stackoverflow.com/a/39598787) when to use them - if we want to use `re.search` it's method signature is as follows: 
```python
search(pattern, string, flags=0) 
```

If we use partials, we can have multiple versions of the `search` function, for example: 
```python
is_spaced_apart = partial(re.search, '[a-zA-Z]\s\=') # pattern '[a-zA-Z]\s\=' applied
is_grouped_together = partial(re.search, '[a-zA-Z]\=') # pattern '[a-zA-Z]\=' applied
```

Later in the code we could use it as:
```python
for text in lines:
    if is_grouped_together(text):
        some_action(text)
    elif is_spaced_apart(text):
        some_other_action(text)
    else:
        some_default_action()
```