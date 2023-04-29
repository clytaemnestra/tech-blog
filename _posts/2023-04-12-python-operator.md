---
layout: post
title: "TIL: Union Operator"
description: "TIL: Union Operator"
date: 2022-04-12
tags: [python]
---

Today I discovered the dictionaries union operator '|'. 

```python
>>> dict1 = {"a": "a", "b": "b"}
>>> dict2 = dict1 | {"c": "c"}
>>> dict2
{'a': 'a', 'b': 'b', 'c': 'c'}
```

I found it to be very useful for testing APIs, where I create base payload with data that's common for all payloads and then I create a specific payload for each request:
```python
base_payload = {"my base": "field"}
client.post("/my/url", json = base_payload | {"my custom": "field"})
client.post("my/second/url", json = base_payload | {"other custom": "field"})

```