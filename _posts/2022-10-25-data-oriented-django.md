---
layout: post
title: "Notes from Conferences: Data-Oriented Design"
description: "Notes from Conferences: Data-Oriented Design"
date: 2022-10-25
tags: [conferences, django]
---

# Data-Oriented Design
Talk: https://www.youtube.com/watch?v=gfPNeQR1aSc

<br> 

## Software's Only Job is to *Transform Data* 
![image](https://user-images.githubusercontent.com/38294198/197749836-3679b739-e315-4fd9-9d5e-9bb9c4a7cfad.png)

<br> 

### What Do Users Care About?
* getting their output data -> they don't care about what do you do in the middle 

<br> 

Data Characteristics:
* format
* volume
* latency
* throughput
* statistical distribution

<br> 

### Context Is Everything
Example: check if a number exists in a given set

```python
numbers: set[int]

def is_match(number: int) -> bool:
    return number in numbers 
```
<br> 

But what is there's just a single number in that set? In that case `in` operator is wasteful.

```python
def is_match(number: int) -> bool:
    return number == 42
```
<br>

What is a billion numbers? Building a set in this case would exhaust the memory.

```python
def is_match(number: int) -> bool:
    with connection.cursor():
        cursor.execute(
            "SELECT 1 FROM numbers WHERE n = %s",
            (number, ),
        )
        return (cursor.fetchone() is not None) 
```

<br> 

What is matching many numbers against few? Many functions aren't called just once, they're called repeatedly, which might not be efficient.

```python
numbers = set[int]

def matches(sought: set[int]) -> set[int]:
    return numbers.intersection(sought)
```

<br>

What is set is all off numbers between 1 and 1 million? In that case we could add more maths.

What if false positives are acceptable? In that case the bloom filter would be a better data structure.

... 


-> the data that we use as the input will influence which function do we write and also the context

-> implementation depends on data characteristics and that is data-oriented design 

<br> 

## Software's only job is to transform data using *specific hardware*
### General-purpose CPU
* Django is mostly run on general-purpose CPU 
![image](https://user-images.githubusercontent.com/38294198/197799408-0f21a876-2795-4d87-b313-24287fb2021e.png)
* there's a gap betweek processor and memory performance since 1980
  ![image](https://user-images.githubusercontent.com/38294198/197799609-a456a327-0499-4757-a5d8-75e4c1bc3725.png)
* processor can do a lot of stuff fast, but it can't get stuff from memory fast
* the solution to this issue is CPU cache between CPU and memory, so the data is transferred into cache, so it can be fetched quickly
  ![image](https://user-images.githubusercontent.com/38294198/197799933-1031364d-77ef-49b2-910a-ac87432e5c2a.png)
* modern CPU's have 3 levels of cache (sometimes even 4)
* each layer of cache is 3x faster than the previous one
  ![image](https://user-images.githubusercontent.com/38294198/197800235-5300e6e3-242c-467a-ae86-933e2d0f58a0.png)
* how many cycles takes fetch from cache:
  ![image](https://user-images.githubusercontent.com/38294198/197800526-264f780d-3a5d-4583-a875-8c2563cd9f6e.png)

<br>

### Implications of This Design 
* use smaller representations 
* lay out data in access order
* Python built-in types are big
  ```python
  In [1]: 64 // 8
  Out [1]: 8
  
  In [2]: import sys
  
  In [3]: sys.getsizeof(9001)
  Out [3]: 28
  ```
    * 8 bytes for an int + 20 bytes for another stuff (PyObject id, etc.)
  ```python
  In [4]: numbers = list(range(1000))
  
  In [5]: len(numbers) * 8
  Out[5]: 8000
  
  In [6]: (
     ...:    sys.getsizeof(numbers)
     ...:    + sum(sys.getsizeof(n) for n in numbers)
     ...: )
  Out[6]: 36052
  ```
* C data types are much slimmer 
  ```python
  In [7]: import array
  
  In [8]: numbers2 = array.array('Q', range(1000))
  
  In [9]: sys.getsizeof(numbers2)
  Out[9]: 8320
  ```
* normally use NumPy or Pandas for arrays 

<br> 

## Data-Oriented Design *for the Web*
* typical website:
  ![image](https://user-images.githubusercontent.com/38294198/197803136-f4e66069-dfe3-4bf9-ac68-f2ae46c2cf12.png)
* user <-> browser loop is already super optimized and quick
  ![image](https://user-images.githubusercontent.com/38294198/197803537-c333885f-69dc-4f09-ace5-6c35493291bb.png)
* but what to put in the *something* part?
  ![image](https://user-images.githubusercontent.com/38294198/197803837-88131b4e-28e8-43c9-8c6d-2eb1fa3cbc3b.png)
* one option is to put there Django & Postgres 
  ![image](https://user-images.githubusercontent.com/38294198/197804051-b10fb606-742a-4e3f-9159-be8018c37417.png)
* another possible architecture:
  ![image](https://user-images.githubusercontent.com/38294198/197808997-f3fa85e6-2384-4107-96f7-ce8c4cc8bd55.png)
* how fast does the data transfer need to be?
    * <100ms - great
    * <1s - pretty good
    * <3s - at 3s lose ~50% of visitors
    * >10s - users likely to retry or give up    

<br> 

### How to Improve the Flow Between Browser and Django?
![image](https://user-images.githubusercontent.com/38294198/197809919-5d5b3095-66ab-4339-b2f2-2019e273ed38.png)
* write minimal, performant HTML - if you get your CSS and JavaScript tags in the first kilobyte the browser will stop requesting those early on, don't load HTML with loads of classes as some CSS frameworks to like to add etc. 
* introduce HTTP caching - tell browsers what to store forever, so for example images are loaded once only
* get on the latest version of HTTP - 3, or at least 2
* response compression (GZipMiddleware) - compresses requests, it's a compression algorithm for text formats like HTML
* HTML minification (django-minify-html) - saves around 1-3% of the request size
* use a CDN (Content Delivery Network) - saving a lot of network time, as servers as closer to users

<br> 

#### Resources
* MDN: Web Performance
* web.dev/learn
* WebPageTest.org
* web.dev/measure 

<br> 


### How to Improve the flow Between Django and Database?
![image](https://user-images.githubusercontent.com/38294198/197811897-20d5cf2b-1af4-468e-ab51-fb8e616429b5.png)
![image](https://user-images.githubusercontent.com/38294198/197813454-e2185398-78d0-4eb9-9995-d310fb5e7e61.png)
* avoid N+1 queries 
  ```python
  books = Book.objects.order_by("title")
  for book in books:
        print(book.title, "by", book.author.name)
  ```
  * iterate books + for each of N books fetch book.autor 
  * instead you can use `select_related`
    ```python
        books = (
            Book.objects.order_by("title")
            .select_related("author")
        )
        for book in books:
            print(book.title, "by", book.author.name)
    ```
    * `select_related` sends one query, where fetches books with author joined in
    * the drawback of `select_related`
    ![image](https://user-images.githubusercontent.com/38294198/197815074-f00f9b72-e242-4d36-bc2f-d50fe26dd9e5.png)
    * comes as single table of data
    * some values might be repetitive
  * another solution is to use `prefetch_related`
    ```python
        books = (
            book.objects.order_by("title")
            .prefetch_related("author")
        )
        for book in books:
            print(book.title, "by", book.author.name)
    ```
    * the first query fetches the books and the second query fetches related authors for the fetched books
    * so if there's 5 authors, which have written 1000 of books it will pull back the 5 authors once, no repetition
    * depends n the data volumes, it can be faster than `select_related`
  * you can also use `django-auto-prefetch` which works in the following way:
    * fetch books
    * on first access of book.author fetch related authors for all books
    * solves N+1 problem for foreign keys and one-to-one fields (it doesn't help with many-to-many)
* split models 
    * imagine the following model:
    ```python
        class User(AbstractUser):
            avatar = models.ImageField(...)
            ...
    ```
    * you're given the following task: store user's ACME access token and refresh token 
    * one solution:
    ```python
        class User(AbstractUser):
            avatar = models.ImageField(...)
            ...
            acme_access_token = models.TextField()
            acme_access_expires = models.DateTimeField()
            acme_refresh_token = models.TextField()
    ```
    * the problem with this solution is that it's going to slow down every place where users are queried 
    * every time you update a row, Postgres creates a whole new copy of that data every time
    * better solution:
    ```python
        class User(AbstractUser):
            avatar = models.ImageField(...)
            ...
            
        class UserAcmeToken(models.Model):
            user = models.OneToOneField(User, primary_key=True)
            acme_access_token = models.TextField()
            acme_access_expires = models.DateTimeField()
    ```
* multiple counts in one pass
    * example:
    ```python
        published_count = (
            author.book_set.filter(published=True).count()
        )
        unpublished_count = (
            author.book_set.filter(published=False).count()
        )
    ```
    * we can use another way to ask for both parts at the same time:
    ```python
        counts = (
            author.book_set.aggregate(
                published=Count('pk', filter=Q(published=True)),
                unpublished=Count('pk', filter=Q(published=False)),
            )
        )
    ```
    * in this way Postgres goes only ones over data 

<br>

#### Resources
* Docs: Database access optimization
* The Temple of Django Database Performance
* Post: "Django and the N+1 Queries Problem"
* django-debug-toolbar or Kolo

<br>

#### Resources for Data-Oriented Design
* Mike Acton - Data-Oriented Design and C++
* Andrew Kelley - Practical DOD
* Andreas Fredriksson - Context is Everything
