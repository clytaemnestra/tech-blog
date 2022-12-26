---
layout: post
title: "Notes from Conferences: Run your tests in hundreds of different environments fast. I mean really fast"
description: "Notes from Conferences: Run your tests in hundreds of different environments fast. I mean really fast"
date: 2022-10-25
tags: [conferences, django]
---

Talk: https://www.youtube.com/watch?v=cLptIbomkoM


### Problem
* large amount of tests: ~450 tests in test suite
* large amount of environments to run test suite in: > 400
* 7 Python versions
* 20 Python web frameworks 
* 2-9 versions of each web framework 
* total duration: 38 min 51 sec



The aim is to reduce the total duration of tests running time.



Testing stack:
* Pytest
* Flake8, Black & MyPy
* Tox
* Make
* GitHub Actions 

<br> 

### Refactoring Ideas

#### Running tests in paralel
#### Tox parallel auto 
* first idea was to use `tox --parallel auto`
* default:  
![image](https://user-images.githubusercontent.com/38294198/197529098-e7988789-b49d-45fe-8cda-d88f26a595c4.png)
* with `tox --parallel auto`:
![image](https://user-images.githubusercontent.com/38294198/197529295-14249173-1a79-4dc3-8456-b25d70f54387.png)
* improved speed from 40 to 25 minutes, but that was not good enough
 
 <br> 

#### Splitting it up 
* design: ![image](https://user-images.githubusercontent.com/38294198/197529651-e291737d-6dd8-485e-9e39-6e9e67bc7560.png)
* custom writted script to parse the tox file and create a yaml file for each configuration
* ![image](https://user-images.githubusercontent.com/38294198/197529899-cea9aa7d-2a0b-4dad-bd24-6e4533688ff3.png)
* reality: ![image](https://user-images.githubusercontent.com/38294198/197530708-06d0d385-aa93-48e5-8343-baf2be248315.png)
* there were only 15 concurrent runners, as there was limited amount of runners on the company level, so when someone else was running tests, there would be less runners available

<br> 

#### A bit of both worlds - parallelize CPU + parallelize runners
* design: ![image](https://user-images.githubusercontent.com/38294198/197530906-707be83e-2da0-4a24-815f-7746a67f99ed.png)
* saved 20-30 minutes

<br> 

#### Cleaning up YAML file 
* removed node.js as it's not used anymore
* removed Redis and Postgres services, as Redis was not used anymore and Postgres was used only for testing
* result: saved 2 minutes 

<br> 
 
#### GitHub Actions Cache 
* cache is used for caching dependencies from `pypi` 
* the idea was to cache the whole directory, so the dependencies are not installed over and over again
* it works in a way that if any dependency is change, the cache is invalidated, but in the reality the cache didn't refresh and they were running the tests against the old dependencies 
* they decided not to implement it, as it did not work correctly 

<br>

#### RAM drives
* the basic idea was - if you can't use the cache, use RAM drives 
* directories in your file system, which is not sitting on the disk but in the memory 
* memory is 50x faster than disk, therefore it might be the better to have virtual environments directly in the memory
* ![image](https://user-images.githubusercontent.com/38294198/197532516-0012739f-a9aa-43c2-ae02-f743801fc757.png)
* saved 2 minutes 

<br> 

#### Local PyPI Server
* the virtual environments are in the memory now, but all dependencies still have to be downloaded from the PyPI server, before they're installed, so the idea is to create a local PyPI server
* it was not implemented 

<br> 

### Total Results 
#### Ideas Considered
![image](https://user-images.githubusercontent.com/38294198/197533315-f9c8fca6-795e-41cf-8c1d-86245160118d.png)

#### Tests Running Time
* from ~40 minutes to <7 minutes 
