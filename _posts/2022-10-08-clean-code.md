---
layout: post
title: "Clean Code"
description: "Notes from Books: Clean Code"
date: 2022-10-08
tags: [books]
---

## Clean Code Principles 
* DRY - don't repeat yourself
* KISS - keep it simple and stupid
* follow standard conventions

## Design
* keep configurable data at high levels
* prefer polymorphism to if/else and switch/case
* follow the law of demeter - a class should know only its direct dependencies 
* be consistent

## Meaningful Names 
* should be intention-revealing
* don't use 'list' for grouped stuff if it's not a list (DS), use 'group' instead
* don't use abbreviations
* use pronounceable and searchable names 


## Functions 
* should do only one thing
* should be small
* ideally should have zero (niladic) or one argument (monadic), followed by two (dyadic) and three (polyadic)
* more than three requires very special justification and shouldn't be used 
* don't pass a boolean into a function - does one thing if the flag istrue and another if the flag is false 
* up to 10-20 lines

## Comments
* if you're thinking about writing a comment, then the code should be refactored 
* don't write comments if you can explain your intent in code 
* comment should explain WHY 
* comments can be used to warn about certain consequences 
* TODO comments
* not every function needs to have docstring - it clutters up the code 


## Formatting 
* check recommendations for lines per file 
* smaller files are easier to understand than large ones 