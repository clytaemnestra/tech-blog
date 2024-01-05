---
layout: post
title: "TIL: Real-time Collaboration in Various Tools on the Market"
description: "TIL: Real-time Collaboration in Various Tools on the Market"
date: 2024-01-05
tags: [system-design, algorithms]
---

I got a task to explore how real-time collaboration works in various tools on the market. These are my notes.

# Approaches
There are few possible approaches seen in tools on the market:
- OT
- CRDT
- hybrid approach

# OT
## Google Docs
### First Version - How not to do it
- the underlying principle was comparison of document versions → the server tries to merge all versions and create a one which is propagated to other users
- quite often it doesn’t work well, because merging algorithms don’t have enough of information about the context:
    - example:
        - initial sentence: The quick brown fox
        - Alice: The quick **brown** **fox**
        - Bob: The quick brown dog
        → the result should be The quick **brown dog,** but the server was mostly merging it into The quick **brown fox** dog or The quick **brown** dog or The quick **brown** dog **fox**

### Current version
- the document is stored as sequence of operations and they are applied in order
- the main principle of collaboration in this version is Operational Transformation
    - each change is represented as an operation
    - to handle concurrent operations, the transform function which takes two operations and computes a new one is used
- these systems use replicated documents storage, where each client has their own copy in a lock-free manner then changes are propagated to the rest of the clients → during the propagation the transformation function is run and all replicas are updated
- this solution is considered as memory-efficient
- example:
<img src="./images/posts/rtc1.png" alt="Transformation function" style="max-width: 100%; max-height: 100%; display: block;">

Example of a transformation function - code:
```
Tii(Ins[p1, c1], Ins[p2, c2]) {
  if (p1 < p2) || ((p1 == p2) && (order() == -1))  // order() – order calculation
		return Ins[p1, c1]; // Tii(Ins[3, ‘a’], Ins[4, ‘b’]) = Ins[3, ‘a’]
  else
		return Ins[p1 + 1, c1]; // Tii(Ins[3, ‘a’], Ins[1, ‘b’]) = Ins[4, ‘a’]
}

Tid(Ins[p1, c1], Del[p2]) {
  if (p1 <= p2)
    return Ins[p1, c1]; // Tid(Ins[3, ‘a’], Del[4]) = Ins[3, ‘a’]
  else
    return Ins[p1 – 1, c1]; // Tid(Ins[3, ‘a’], Del[1]) = Ins[2, ‘a’]
}

Tdi(Del[p1], Ins[p2, c1]) {
  // Exercise
}

Tdd(Del[p1], Del[p2]) {
  if (p1 < p2)
    return Del[p1]; // Tdd(Del[3], Del[4]) = Del[3]
  else
    if (p1 > p2) return Del[p1 – 1]; // Tdd(Del[3], Del[1]) = Del[2]
  else
    return Id; // Id – identity operator
}
```

## Etherpad (collaborative online text editor)
- also uses OT technique, but a slightly different one
- it sends changes to the server as changeset in the format (p1 -> p2)[c_1, c_2, …], where
    - p1 — length of the document before the change
    - p2 — length of the document after the change
    - c_i — definition if the document:if c_i — number or range of numbers then it means indices of *retained* character, orif c_i — character or string then it means insertion
- example:
    - “” + (0 -> 5)[“Hello”] = “Hello”
    - “Hllo World” + (10 -> 14)[0, ‘e’, 1–9, “!!!”] = “Hello World!!!”
- for conflict resolution Etherpad uses merge function, which takes 2 changesets and computes a new single one :
    - example of automatic conflict resolution:
    <img src="./images/posts/rtc2.png" alt="Automatic conflict resolution" style="max-width: 100%; max-height: 100%; display: block;">

- consistency
    - Google Docs solves this by using CCI model
        - it takes care of convergence - all replicas of the same document must be identical after execution of all operations
        - intention preservation - effect of executing an operation on any document state will be the same as the effect of its execution on a replica it was created
        - causality preservation - keeping correct order of operations
- keeping correct order of operations including the cases of network lags - for this they use data structure Vector clock

## Problems with OR
- error-prone implementation needs to account for each pair of operations
- combinatorial complexity - the number of possible combinations grows quadratically with the number of operations
- complex to implement

## Sources
- https://en.wikipedia.org/wiki/Operational_transformation
- https://medium.com/coinmonks/operational-transformations-as-an-algorithm-for-automatic-conflict-resolution-3bf8920ea447
- https://www.redhat.com/en/command-line-heroes/season-6/clarence-ellis
- https://github.com/ether/etherpad-lite/blob/develop/doc/easysync/easysync-full-description.pdf
- https://drive.googleblog.com/2010/09/whats-different-about-new-google-docs_22.html


# CRDT (conflict-free replicated data type)
- example apps: Apple Notes, Riak, TomTom GPS, Teletype for Atom
- the principle is to break data into small pieces, that don’t need to be transformed → only their position in the text change
    - for example, each character is a single entity with its id and it has its own position in the document
    - the position is represented mostly as path in a tree or as fractional number like in the example below
    <img src="./images/posts/rtc3.png" alt="CRDT representation" style="max-width: 100%; max-height: 100%; display: block;">
- supports collaboration on only two main types of data:
    - plain text
    - JSON structures
- some examples of types of CRDT:
    - grow-only set
        - it’s possible only to add elements
        - no conflicts as adding something twice is not an operation
    - last-writer-wins register
        - updates as registered as a new value with timestamp and user id
        - in case of conflict the last update is always used and other ones are discarded
- peer-to-peer communication, no single central authority to decide what the final state should be

## Problems
- newer, therefore fewer reference implementations
- easy to implement, but it’s hard to get it right in the way to satisfy user expectations
- many published algorithms have anomalies that cause them to behave strangely in some situations:
    - interleaving anomalies in text editing
    <img src="./images/posts/rtc4.png" alt="Anomalies problem" style="max-width: 100%; max-height: 100%; display: block;">
        - good algorithms: Treedoc, Woot
        - bad: Logoot, LSEQ, RGA, Astrong
    - moving or reordering list items
    <img src="./images/posts/rtc5.png" alt="Reordering list items problem" style="max-width: 100%; max-height: 100%; display: block;">
    - if there’s only one item then the last-writer-wins register is used
    <img src="./images/posts/rtc6.png" alt="Last-writer-wins register with one item" style="max-width: 100%; max-height: 100%; display: block;">
    - but the same principle doesn’t work on multiple items:
        - desired outcome:
        <img src="./images/posts/rtc7.png" alt="Multiple items desired outcome" style="max-width: 100%; max-height: 100%; display: block;">
        - end result:
        <img src="./images/posts/rtc8.png" alt="Multiple items end result" style="max-width: 100%; max-height: 100%; display: block;">
    - no solution for this problem with CRDTs at the moment 
    - moving subtrees of a tree 
    ```bash
    mkdir a
    mkdir a/b
    mv a a/b 

    mv: cannot move 'a' to a subdirectory of itself, 'a/b/a'
    ```
        - might create a cycle, so algorithms in this case don’t do anything to prevent cycles
    - reducing metadata overhead of CRDTs
        - big overhead, reduces efficiency
        <img src="./images/posts/rtc9.png" alt="Metadata overhead of CRDTs" style="max-width: 100%; max-height: 100%; display: block;">
    - library that provides fast implementations of CRDTs: https://github.com/automerge/automerge

## Sources:
- https://www.tiny.cloud/blog/real-time-collaboration-ot-vs-crdt/
- https://hal.inria.fr/file/index/docid/629503/filename/doce63-ahmednacer.pdf
- https://github.com/xi-editor/xi-editor/issues/1187#issuecomment-491473599
- https://www.youtube.com/watch?v=x7drE24geUw
- https://en.wikipedia.org/wiki/Conflict-free_replicated_data_type

# Hybrid approaches
## Figma
- Figma has client/server architecture where clients are web pages that talk with a cluster of servers over websockets
- typical flow:
    - when a document is opened, the client downloads a copy of the file
    - updates to that document are synced over the websockets connection
    - if user goes offline, they can edit it and after they go online again, the client downloads a fresh copy of the document and all offline edits are reapplied
- their approach is combination of multiple CRDTs

## Documents
- tree of objects, similar to the HTML DOM
- objects:
    - root - document
    - page objects under the root
    - hierarchy of objects under each page which represents the contents of the page
- each object has an ID and a collection of properties with values
    - one way to think about this is by picturing the document as a two-level map:
     `Map<ObjectID, Map<Property, Value>>`
    - another way to think about this is a database with rows that store `(ObjectID, Property, Value)tuples ->`  adding new features to Figma usually just means adding new properties to objects

## Merge conflict resolution
- servers keep track of latest values for a given property on a given objects - if clients are  changing unrelated properties or the same property on related objects there’s no conflict
- in case of conflict, which happens when two clients change the same property on the same object, the document will just end up with the last value that was sent to the server
- similar to a last-writer-wins register in CRDT except there’s no timestamp
- simultaneous text editing doesn’t work in Figma, but they don’t need it
    - if text value is B and one user changes it to AB and another one to BC the end result will be either AB or BC, but not ABC
    - animation: https://cdn.sanity.io/files/599r6htc/localized/e8b3d7dd4745e9fec062586d91f3eba62c2052be.mp4
- if an object is deleted by one user, it’s properties are not stored on the server but inside the undo buffer of the client that performed the delete
- parent-child relationship is stored as link to the parent as a property on the child, which solves an issue when one object is moved by one user and its color is changed by another
- if parent property updates would cause a cycle, servers reject the update
    - in this case objects are temporarily removed from the tree until the server rejects the client’s change and the object is reparented where it belongs
    - animation: https://cdn.sanity.io/files/599r6htc/localized/280f28f6e620d747f00ef024058310d07e151eff.mp4
- ordering of children is solved by the technique called fractional indexing
    - animation: https://cdn.sanity.io/files/599r6htc/localized/e7db3c4428336425aebca94ba6914b692b5e8a25.mp4

## Undo feature
- an undo operation modifies redo history at the time of the undo, and likewise a redo operation modifies undo history at the time of the redo
    - animation: https://cdn.sanity.io/files/599r6htc/localized/190727fe7a910cf7753edaa289825dd0cda3c3a1.mp4

## Problems
- high complexity - arranging objects in an eventually-consistent tree structure
- no other examples of the implementation except of Figma’s
- no documentation

## Apollo
- custom approach inspired by Figma - Atomic Operations
- the principle is that all edits to application state are broken down to their smallest atomic parts
    - two operations of different types cannot conflict with each other
    - if there’s two operations of the same type, a winner is picked → last-writer-wins, no merge
    - how to determine who was the last one?
        - the server keeps track of a monotonically increasing object counter per object that increments with each write
        - upon acknowledging an operation, the server includes the latest value of this counter
        - to determine which operation is a winner, the client simply chooses the operation with the higher value
- certain types of mutations (their API is in GraphQL) can be difficult to express as simple last-writer-wins operations - for example ordered collections
    - example: if someone inserts a new item between positions 1 and 2, and someone else inserts a new item between positions 3 and 4, each newly-created item's index value is dependent on insert order, which requires updating the index of all existing items
    - solution: fractional indexing - using fractional values (0-1) instead of natural numbers to describe the order of a collection → there’s an infinite number of fraction between any two indices to choose from
- it’s not possible to use last-writer-wins for test strings - in this case cell locking is used
    - animation: https://hex.tech/static/lock_hijack-ba8150eb8506f6aeef69f248cd322fc2.mp4

## Sources:
- https://hex.tech/blog/a-pragmatic-approach-to-live-collaboration/#fn-1
- https://www.figma.com/blog/how-figmas-multiplayer-technology-works/
- https://www.figma.com/blog/realtime-editing-of-ordered-sequences/