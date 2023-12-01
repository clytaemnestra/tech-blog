---
layout: post
title: "TIL: Rust & Advent of Code"
description: "TIL: Rust & Advent of Code"
date: 2023-12-01
tags: [rust]
---

For the first time I've decided to try to rewrite some solutions to Rust.

## Data structres
* if I have a counter, it has to be `mut`, because it's mutable: `let mut sum = 0;`
* when assigning a value which cannot be `None` then we use `Some` to avoid rull referece errors
```Rust
let mut first = None;
if first.is_none() {
    first = Some(ch);
}
```

## Iterating
* Rust uses `if let` construction for pattern matching and conditional execution:
    * firstly it checks the condition
    * then it extraacts values from `Option`
    * then it binds them to variables

```Rust
if let (Some(first_number), Some(last_number)) = (first, last) {
    ...
}
```
-> is equivalent to Python:
```Python
if first is not None and last is not None:
    first_number = first
    last_number = last

else:
    ...

``` 

## Casting 
TBA 

## Reading from a file
TBA