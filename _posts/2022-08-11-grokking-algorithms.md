---
layout: post
title: "Grokking Algorithms"
description: "Notes from Books: Grokking Algorithms"
date: 2022-08-11
tags: [books, algorithms]
---


# Linked lists & Arrays 
### Access
* linked list can only do sequential access 
  * if you want to read the 10th element, you need to read the first 9
  * elements are strewn all over, and one element stores the address of the next one
  * allow fast inserts & deletes 
* arrays can do random access
  * you directly jump to the 10th element
  * arrays thus allow fast reads 
  * all elements are stored right next to each other


### Runtimes 
![obrazek](https://user-images.githubusercontent.com/38294198/174497967-b0531a88-b9ae-491a-9357-c6f65a3d005c.png)

## Creation of a linked list
### Principle
* create an empty linked list 
* add head and assign to it data (data is in class Node)
* add second node and assign to it data
* add third node and assign to it data
* link head with second node 
* link second with the third node
* third node has default `None`, so no need to do anything with it

### Code
```python
llist = LinkedList()
llist.head = Node(1)
second = Node(2)
third = Node(3)

llist.head.next = second
second.next = third
```
## Insertion 
### Beginning
### Principle
* create a new node and add data to it
* switch head & new node:  
    * as next add there current head
    * move head to the new node

### Code
```python
def insert(self, data):
    new_node = Node(data)
    new_node.next = self.head
    self.head = new_node
```
### After a given node 
### Principle
* create a new node and add data do it 
* change next of the previous node
* set up next to point to the next node

### Code
```python
def insert_after(self, previous, data):
    new_node = Node(data)
    new_node.next = previous.next
    previous.next = new_node
```

### End
### Principle
* create a new node and add data to it (next is by default `None`)
* set up previous next from `NULL` to new node

### Code
```python
def append(self, data):
    new_node = Node(data)
    # if the list is empty, make the new node as head
    if self.head is None:
        self.head = new_node
        return
    # else traverse till the last node
    # set up first one as 'last'
    # while there is node that has 'next' 
    # set up last as the current node
    last = self.head 
    while last.next:
        last = last.next
    
    # change the next of last node
    last.next = new_node 
    
```

## Deletion
### Principle 
* store the head node 
* if head node stores the key to be deleted, then set up the next node as head
* if not, search for the key to be deleted -> until it's not `None`
* when it finds the key:
    * save the current node as previous
    * save the next as current
* unlink the node from the list     

### Code
```python
def delete(self, key):
    to_delete = self.head 
    # if head node is to be deleted
    if to_delete is not None:
        if to_delete.data == key:
            self.head = to_delete.next
            to_delete = None
            return 

    while to_delete is not None:
        if to_delete.data == key:
            break
        previous = to_delete
        to_delete = to_delete.next

    previous.next = to_delete.next
    to_delete = None
```

## Reverse
### Principle 
* create a variable previous - at the beginning none
* create a variable current node - head of the list  
* move temp to the second place  
* update current's pointer's value to previous -> here changing value 
* assign the current as previous -> iteration 
* assign the next one temp as current -> iteration 

### Code
```python
def reverse(head):
    current = head
    previous = None

    while current:
        tmp = current.next        # set up the next one as temp 
        current.next = previous   # changing values of the pointer to the next one!
        previous = current        # iteration 
        current = tmp             # iteration

    head = previous
```

# Big-O
## The five most common run times 
![obrazek](https://user-images.githubusercontent.com/38294198/174497886-2bee4006-31a5-48cb-b38f-e59f3acf9c72.png)


* fun fact: O(log n) gets a lot faster once the list of items you're searching through grows

## The travelling salesperson problem 
 * sales person needs to visit 5 cities 
 * for 5 cities it's 120 permutations
 * n! operations:
     * for 6 cities - 720 permutations
     * for 7 cities - 5040 permutations
 * only approximate solution existing 


 ## Pseudocode
```
A = sorted array 
n = length of array 
x = value to be searched 

set low = 0
set high = n - 1

while x not found:
    mid = high - low // 2
    
    if x = A[mid]   
        EXIT: x found at location A[mid]
       
    if x > A[mid]
        low = mid + 1
       
    if x < A[mid]
        high = mid - 1 
      
end while
```
## Code
```python
def search(numbers_list, guess):
    low = 0
    high = len(numbers_list) - 1

    while low <= high:
        mid = (low + high) // 2

        if guess == numbers_list[mid]:
            return print("found the item: ", numbers_list[mid])

        elif guess > numbers_list[mid]:
            low = numbers_list[mid] + 1

        elif guess < numbers_list[mid]:
            high = numbers_list[mid] - 1

    return None


search(numbers_list=[1, 2, 3, 4, 5, 6, 7, 8, 9, 10], guess=8)

```

# Hashtables
## Principle
![obrazek](https://user-images.githubusercontent.com/38294198/177056909-d7b57771-efb4-4055-8f20-644736deb9c9.png)

### Performance
![obrazek](https://user-images.githubusercontent.com/38294198/177056899-15f3fb0e-23f4-4edb-87aa-217dae1d2a7e.png)

### Colisions
* the simplest way: if multiple keys map to the same slot, start a linked list at that slot
* Python uses open addressing:
    * linear probing - if collision occurs, find the next slot 
    * quadratic probing - find the next slot + x*x

Linear probing:
```
If slot hash(x) % S is full, then we try (hash(x) + 1) % S
If (hash(x) + 1) % S is also full, then we try (hash(x) + 2) % S
If (hash(x) + 2) % S is also full, then we try (hash(x) + 3) % S 
```

Quadratic probing:
```
let hash(x) be the slot index computed using hash function.  
If slot hash(x) % S is full, then we try (hash(x) + 1*1) % S
If (hash(x) + 1*1) % S is also full, then we try (hash(x) + 2*2) % S
If (hash(x) + 2*2) % S is also full, then we try (hash(x) + 3*3) % S
```



### Operations principle
* insert(k): keep probing until an empty slot is found. Once an empty slot is found, insert k 
* search(k): keep probing until slot’s key doesn’t become equal to k or an empty slot is reached
* slots of deleted keys are marked specially as “deleted”, the insert can insert an item in a deleted slot, but the search doesn’t stop at a deleted slot

### Load factor
* measures how many empty slots remain in the hash table
* hashes use array for storage, so you count the number of occupied slots in an array, i.e. 2/5 = 0.4
* resize when your load factor is greater than 0.7 


# Trees
### Binary Search Tree 
![image](https://user-images.githubusercontent.com/38294198/179522822-56fdf0c4-2622-4625-b7a0-e8ee14058783.png)
* cons - don't have random access 
* performance times are on average and rely on the tree being balanced

Unbalanced tree:
![image](https://user-images.githubusercontent.com/38294198/179523066-8eac29b8-1b84-4829-8da0-0420f0080cd8.png)

### B-Trees
* self-balancing search tree
* time complexity of all operations is O(log n)
* insertion happens only at leaf node
* shrinks from the root (binary search tree grows downward and also shrinks from downward)
![image](https://user-images.githubusercontent.com/38294198/179523576-fe09d4e2-a061-45ca-ac19-59ecb8f30238.png)


### Red-black trees
* self-balancing binary search tree
* search time - O(log n)
* every node has an extra bit, which is either red or black - used for balance
* 1 bit to store the colour information, identical memory footprints to the classic binary search tree
* root is always black
* a red node cannot have a red parent or red child
* every path from a node (including root) to any of its descendants NULL nodes has the same number of black nodes
* all leaf nodes are black nodes 
* used in KNN for reducing time complexity
* MySQL uses B+ trees fot data indexing in database engine


### Heaps
* similar to the binary tree with the difference that heap has in the root either minimum or maximum values
* it's possible to have duplicates in heap    

# Quicksort
## Principle
* figure out the base case - > very often it's an array with 0 or 1 element
* more closer to an empty array, reduce the size by using recursion


## Steps
* pick an element (pivot) from the array  
* find elements smaller than the pivot and the elements larger than the pivot - partitioning
* call quicksort on both sub-arrays  
* if sub-arrays are sorted, then you can combine the whole thing like left array + pivot + right array

## Code
```python
def quicksort(array):
    if len(array) < 2:
        return array

    else:
        pivot = array[0]
        left_array = [i for i in array[1:] if i <= pivot]
        right_array = [i for i in array[1:] if i > pivot]
    
    return quicksort(left_array) + [pivot] + quicksort(right_array)
```

# Breath-first Search
## Principle
* used to answer the following 2 questions:
    * is there a path from node A to node B? 
    * what is the shortest path from node A to node B?
* running time: O(V+E) - number of vertices + number of edges 
* once you check an edge, make sure not to check it again - otherwise you might end up in an infinite loop

![image](https://user-images.githubusercontent.com/38294198/178215601-fa82ad9a-1315-49eb-9424-cc00dc81dd02.png)


### Pseudocode
* create a queue Q 
* mark v as visited and put v into Q 
* while Q is non-empty 
    * remove the head u of Q 
    * mark and enqueue all (unvisited) neighbours of u

![image](https://user-images.githubusercontent.com/38294198/178245559-f5dd169a-0efa-41aa-8e56-72decf862c3f.png)
![image](https://user-images.githubusercontent.com/38294198/178245596-5a37a33b-918d-439b-a428-5068e86e8483.png)
![image](https://user-images.githubusercontent.com/38294198/178245628-661d1909-d37e-4c81-8817-6bfacb7b834e.png)
![image](https://user-images.githubusercontent.com/38294198/178245652-bff1417b-0795-4da7-bbb3-50607043ad4d.png)
![image](https://user-images.githubusercontent.com/38294198/178245679-a88ececa-8ff6-471f-9b8a-c266a6e74466.png)
![image](https://user-images.githubusercontent.com/38294198/178245713-deb054a1-c391-4760-9005-f8d37f056518.png)


### Code
```python
import collections

def bfs(graph, root):
    visited, queue = set([root]), collections.deque([root])
    while queue:
        vertex = queue.popleft()
        visit(vertex)
        for node in graph[vertex]:
            if node not in visited:
                visited.add(node)
                queue.append(node)

def visit(n):
    print(n)

if __name__ == '__main__':
    graph = {0: [1, 2], 1: [2, 0], 2: []} 
    bfs(graph, 0)
```


# Bloom Filters
## Principle
* probabilistic data structure
* tells that the element either definitely is not in the set or may be in the set
* algorithm:
  * the base is a bit vector
  * to add an element to the Bloom filter, the element is hashed few times and bits in the bit vector are set to `1`
  * to test for membership, you hash the test string and see if those values are set in the bit vector    
* used for example for checking, if URL is malicious - stored in browser, if URL might be malicious, then a request to a remote server is sent
* doesn't store the elements themselves, this is the crucial point
* you don't use a bloom filter to test if an element is present, you use it to test whether it's certainly not present, since it guarantees no false negatives



![image](https://user-images.githubusercontent.com/38294198/184173762-46957757-8ff2-480c-9c84-b789c5589e52.png)


![image](https://user-images.githubusercontent.com/38294198/184173811-604b4026-f053-496a-a962-971c2edfddd3.png)

![image](https://user-images.githubusercontent.com/38294198/184173846-f6dcfd25-7bf4-4e84-bb97-f99c8ccacb41.png)

![image](https://user-images.githubusercontent.com/38294198/184173911-95c2c1ef-cc1f-4d90-a04b-2f4a026cbc14.png)


# Dijkstra Algorithm
## Principle
* terminology:
    * weight - number associate to every edge, makes weighted graph
    * undirected graph - both nodes point to each other, a cycle
* Dijkstra works only with directs acyclic grapgs (DAG)
* Dijkstra works when all weights are positive - if they're negative, use Bellman-Ford algorithm

### Pseudocode
* find the cheapest node (the node you can get in the least amount of time)
* update the costs of the neigbvors of this node
* repeat until you've done this for every node in the graph
* calculate the final path


### Code
```python
def find_lowest_cost_node(costs):
    lowest_cost = float(“inf”)
    lowest_cost_node = None
    for node in costs: # go through each node
        cost = costs[node]
        if cost < lowest_cost and node not in processed: # if it's the lowest so far and hasn't been processed
            lowest_cost = cost # set it as the new lowest-cost node.
            lowest_cost_node = node
    return lowest_cost_node

node = find_lowest_cost_node(costs) 
while node is not None:
    cost = costs[node]
    neighbors = graph[node]
    for n in neighbors.keys(): # go through the neighbours of this node
        new_cost = cost + neighbors[n] 
        if costs[n] > new_cost: # if it's cheaper to get to this neighbour by going through this node
            costs[n] = new_cost # update the cost for this node
            parents[n] = node # this node becomes the new parent for this neighbour
    processed.append(node) # mark the node as processed 
    node = find_lowest_cost_node(costs) # find the next node to process and loop 
```


# Dynamic Programming
### Principle
* technique to solve a hard problem by breaking it up into subproblems and solving those subproblems first
* every dynamic programming solution involves a grid
* the values in the cells are usually what you're trying to optimize 
* each cell is a subproblem, so think about how you can divide your problem into subproblems

#### Example
You’re a thief with a knapsack that can carry 4 lb of goods. You have three items that you can put into the knapsack.

![obrazek](https://user-images.githubusercontent.com/38294198/179278920-c2849d63-64d3-4702-9dcf-c7c45b7c8a47.png)


![obrazek](https://user-images.githubusercontent.com/38294198/179279101-eeb2dfca-df1c-4dd4-a591-080fdb425263.png)


![obrazek](https://user-images.githubusercontent.com/38294198/179279217-0dcec6da-d98e-4774-a5a2-a3c3dd29997c.png)


![obrazek](https://user-images.githubusercontent.com/38294198/179279305-fd8cd52a-3257-426e-a945-9570c242716a.png)


![obrazek](https://user-images.githubusercontent.com/38294198/179279647-8a869f89-de79-46be-bba7-2a0d7d098c3f.png)


![obrazek](https://user-images.githubusercontent.com/38294198/179280276-b58dded6-6554-40d3-96b8-5c09bce563bf.png)


![obrazek](https://user-images.githubusercontent.com/38294198/179280431-21d4e978-c3c3-4ec6-90de-f4d331b36eab.png)



# Greedy Algorithms
## Principle
* simple to write 
* get pretty close 
* NP-complete problems
    * hard to solve
    * better to use approximation algorithms 
    * example: travelling salesperson
* how to know if a problem is NP-complete:
    * algorithm slows down with more items
    * 'all combination of x' kind of problems
    * if you have to calculate every possible version of x, because you can't break it down into smaller subproblems
    * if it involves a set or sequence and it's hard to solve

### Example Problem
Suppose you’re starting a radio show. You want to reach listeners in all 50 states. You have to decide what stations to play on to reach all those listeners. 


It costs money to be on each station, so you’re trying to minimize the number of stations you play on. You have a list of stations. Each station covers a region, and there’s overlap.


How do you figure out the smallest set of stations you can play on to cover all 50 states? 

#### Greedy Algorithm Principle
1. Pick the station that covers the most states that haven’t been covered yet. It’s OK if the station covers some states that have been covered already. 
2. Repeat until all the states are covered.

#### Greedy Algorithm Code
```python

# You pass an array in, and it gets converted to a set.
states_needed = set(["mt", "wa", "or", "id", "nv", "ut", "ca", "az"])

stations = {}
stations["kone"] = set(["id", "nv", "ut"])
stations["ktwo"] = set(["wa", "id", "mt"])
stations["kthree"] = set(["or", "nv", "ca"])
stations["kfour"] = set(["nv", "ut"])
stations["kfive"] = set(["ca", "az"])

final_stations = set()

while states_needed:
  best_station = None
  states_covered = set()
  for station, states_for_station in stations.items():
    covered = states_needed & states_for_station
    if len(covered) > len(states_covered):
      best_station = station
      states_covered = covered

  states_needed -= states_covered
  final_stations.add(best_station)

print(final_stations)
```



# Knn Neighbors
## Principle
* can do two basic things with KNN
    * classification - categorisation into a group
    * regression - predicting a response (like a number)
* the most important is to pick rights features:
    * features that directly correlate (example: movies - features related to movies)
    * features that don't have bias (movies - if you ask users to only rate comedy movies, that desn't tell you whether they like action movies)
![image](https://user-images.githubusercontent.com/38294198/179393465-18e0ba87-87df-436e-acd1-f740220e737d.png)
* classifying objects by similarity
    * Pythagora's theorem
    * cosine similarity


# Linear Programming
## Principle
* the basic principle is to maximize or minimize some function
* small example - feasible solutions are in the gray area, where all three are satisfied
  * ![image](https://user-images.githubusercontent.com/38294198/184233679-292f1b49-6488-49dd-8347-94f794ae8c6d.png)
  * ![image](https://user-images.githubusercontent.com/38294198/184233751-6b187f02-2365-454f-889c-d8789ea4173a.png)
  * added green one - equality constraint
  * ![image](https://user-images.githubusercontent.com/38294198/184234123-2203d2a0-fecb-47db-989f-ae38a09ff313.png)
  * integers only
  * ![image](https://user-images.githubusercontent.com/38294198/184234193-9ba34ddb-a1b8-45d5-8af9-ef783efca05e.png)
  * the solution is the highest green spot, i.e. the closest one to the red area


# Linked Lists
## Creation
### Principle:
* in C it's represented by using structures, in Python or C++ by class
* classes
    * three important things:
        * node -> node object
        * data -> data assigned to the given node 
        * head -> first node
    * first class is a node, which consists of data and pointer to the next one (pointer is intialized as null)
    * second class is Linked List, which consist of head, which is also initiaulized as None 

### Code
```python
class Node:
    def __init__(self, data):
        self.data = data
        self.next = None

class LinkedList:
    def __init__(self):
        self.head = None
```



# Recursion
* every recursive function has two parts:
    * the base case - condition, when to stop the loop
    * the recursive case - when the function calls itself
* tail recursion 
    * tail - the last thing
    * tail recursion = recursion is the last thing that happens 
    * some compilers optimize the tail recursion - it's space complexity is O(n), sometimes it's better to use loops
  

### Example
Factorial calculation:
```python
def factorial(x):
    if x == 1:
        return 1
    else:
        return x * factorial(x-1)
```
fac n = n * fac (n - 1)

Example: 
fac 4 = 4 * fac3
      = 4 * (3 * fac2)
      = 4 * (3 * (2 * fac1))
      = 4 * (3 * (2 * 1))
      = 4 * (3 * 2)
      = 4 * 6
      = 24


Tail-recursive (takes number & variable, which accumulates factorials):
```python
def factorial(x, a = 1):
    if x == 1:
        return a
    return factorial(x - 1, x * a)
```
fac n = go n 1
go 1 a = a
go n a = go (n - 1)

Example: 
fac 4 = go 4 1
      = go (4 - 1) (1 * 4)
      = go 3 4
      = go 2 12
      = go 1 24
      = 24


# OCR
## Principle
* OCR = optical character recoginition
* you take a photo of a page of text and computer automatically reads text for you
* you can use KNN for it - example how to figure out which number it is:
    * go through a lot of images of numbers and extract features of numbers
    * when you get a new image, extract the features of that image and see what its nearest neighbours are

![image](https://user-images.githubusercontent.com/38294198/179393512-4d515327-31ac-4484-8a31-3fecd25121ee.png)


# Parallel Algorithms
## Principle
### MapReduce
* distributed algorithm, it's possible to use it through Apache Hadoop 
![image](https://user-images.githubusercontent.com/38294198/179539159-03330ef0-2a9e-4cb5-8d46-9ec29dc06452.png)
![image](https://user-images.githubusercontent.com/38294198/179540117-f94970b6-f4b2-4eb5-846a-e853ecddec28.png)
