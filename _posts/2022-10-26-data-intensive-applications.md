---
layout: post
title: "Designing Data Intensive Applications"
description: "Notes from Books: Designing Data Intensive Applications"
date: 2022-10-26
tags: [books, system-design]
---

## Data models & query languages 
Historical representation of data:
* hierarchical model (one big tree) - wasn't good for representing many-to-many relationships 
* relational model
* NoSQL nonrelational model

<br>

Two main directions of nonrelational NoSQL databases:
* document databases - data comes in self-contained documents and relationships between one document and another are rare
* graph databases - targeting use case, where anything is potentially related to everything 

<br>

## Storage & retrieval
On high level storage engines fall into two broad categories:
* OLTP
  * user-facing - huge volume of requests
  * the applications requests records using some kind of key and the storage engine uses an index to find the data for the requested key
  * disk seek time is often the bottleneck
* OLAP
  * data warehouses and analytic systems used by business analysts
  * lower volume of requests, but each query is typically very demanging 
  * disk bandwidth is often the bottleneck
  * column-oriented storage is an increasingly popular solution 

<br>
  
| Property              | Transaction processing systems (OLTP)             | Analytic systems (OLAP)                   |
| ----------------------|---------------------------------------------------|-------------------------------------------|
| Main read pattern     | Small number of records per query fetched by key  | Aggregate over large number of records    |
| Main write pattern    | Random-access, low-latency writes from user input | Bulk import (ETL) or event stream         |
| Primarily used by     | End user/customer, via web application            | Internal analyst, for decision support    |
| What data represents  | Latest state of data (current point in time)      | History of events that happened over time |
| Dataset size          | Gigabytes to terabytes                            | Terabytes to petabytes                    |
 
<br> 
  
### OLTP  
OLTP two main schools of thought:
* the long-structured school
  * only permits appending to files and deleting obsolete files, but never updates a file 
  * LevelDB, Cassandra, Lucene, HBase, SSTables, Bitcask etc.
* the update-in-place school
  * treats the disk as a set of fixed-size pages that can be overwritten
  * B-Trees used in all major relational databases

<br>

B-Tree
 * break the database down into fixed-size blocks or segments (typically 4 KB) and read or write one page at a time
 * each page can be identified using an address or location, which allows one page to refer to another
 * one page is designated as the root of the B-tree - whenever you want to look up a key in the index you start here
 * the first page contains several keys and references to child pages 
 * depth O(log n)
 * most databases can fit into a B-tree that is three or four levels deep (a four level tree of 4 KB pages with a branching factor of 500 can store up to 250 TB)
  
    <img src="https://user-images.githubusercontent.com/38294198/194752603-a09d3bda-8b86-45f0-a1ea-72785543325f.png" width="500" />

<br>  


### OLAP 

<img src="https://user-images.githubusercontent.com/38294198/194752380-843302c0-7005-43fa-8685-595b941ce84e.png" width="500" />
 
Schemas for analytics:
* star schema - consists of dimension tables and fact table (foreign key references to dimension tables)
* snowflake schema 
 
<br>
 
Column-oriented storage
* the idea behind: don't store all the values from one row rogether, but store all the values from each column together instead
* if each column is stored in a separate file, a query only needs to read and parse those columns that are used in that query, which can save a lot of work
* as column values are often repearing, it's possible to compress it with bitmax indexes 

<img src="https://user-images.githubusercontent.com/38294198/194752961-d531dfa9-4ccc-43d8-9eb1-d7cd4c62f4f5.png" width="500" />

<br>

## Encoding & evolution
Programs usually work with data in at least two different representations:
* data structures in memory, they are optimized for efficient access and manipulation by the CPU 
* encoded self-contained sequence of bytes (for example JSON) for sending over the network  

<br>

Models of dataflow:
* through databases
* through services (REST & RPC)
* through message brokers 

<br>

### REST 
* philosophy built upon the principles of HTTP
* simple data formats
* URLs for identifying resources
* HTTP features for cache control, authentication and content type negotiation

<br>

### SOAP
* XML-based protocol
* it aims to be independent from HTTP
* it comes with complex multitude of related standards (the web service framework)
* the API of a SOAP web service is described using the web services description language (WSDL)

<br>

### Message brokers
* one process sends a message to a named queue or topic andthe broker ensures that the mesage is delivered to one r nore consumers or subscribers to that queue or topic
* RabbitMQ, Apache Kafka, ActiveMQ, HornetQ, NATS etc.

<br>

Advantages of message brokers:
* it can act as a buffer if the recipient is unavailable or overloaded
* it can automatically redeliever messages to a process that has crashed 
* avoids the sender needing to know the IP address and port number of the recipient 
* logically decouples the sender from the recipient 

<br>


## Derived Data 
On a high level, systems that store and process data can be grouped into two broad categories:
* Systems of record
  * also known as *source of truth*
  * holds the authoritative version of your data
  * when new data comes in, e.g., as user input, it is first written here
  * each fact is prepresented exactly once, the representation is typically normalized 
* Derived data systems
  * data in a derived system is the result of taking some existing data from another system and transforming or processing it in some way
  * if you lose derived data, you can recreate it from the original source
  * a classic example is a cache - data can be served from the cache if present, but if the cache doesn't contain what you need, you can fall back to the underlying database 
  * denormalized values, indexes, and materialized views also fall into this category 
  * in recommendation systems, predictive summary data is often derived from usage logs 
  * this data is technically speaking *redundant*, in teh sense that it duplicates existing information, but it is often essential for getting good performance on read queries
  * it is commonly denormalized
  
<br>

  ### Batch Processing 
  The two main problems that distributed batch processing frameworks need to solve:
  * partitioning
    * in MapReduce mappers are partitioned according to input file blocks
    * the output of mappers is repartitioned, sorted, and merged into a configurable number of reducer partitions
    * the purpose of this process is to bring all the related data, e.g., all the records with the same key - together in the same place
    * Post-MapReduce dataflow engineers try to avoid sorting unless it is required, but they otherwise take a broadly similar approach to partitioning
  * fault tolerance
    * MapReduce frequently writes to disk, which makes it easy to recover from an individual failed task without restarting the entire job, but slows down execution in the failure-free case
    
    How do partitioned algorithms work?
    * sort-merge joins
      * each of the inputs being joined goes through a mapper that extracts the join key
      * by partitioning, sorting, and merging, all the records with the same key end up going to the same call of the reducer
      * this function can then output the joined records
    * broadcast hash joins
      * one of the two join inputs is small, so it is not partitioned and it can be entirely loaded into a hash table
      * thus, you can start a mapper for each partition of the large join input, load the hash table for the small input into each mapper, and then scan over the large input one record at a time, querying the hash table for each record
    * partitioned hash joins
      * if the two join inputs are partitioned in the same way (using the same key, same hash function, and same number of partitions), then the hash table approach can be used independently for each partition

Two common ways data is distributed accross multiple nodes:
* replication - keeping a copy of the same data on several different nodes, potentially in different locations 
* partitioning (sharding) - splitting a big database into smaller subsets called partitions so that different partitions can be assigned to different nodes 

<br/> 

### Replication 
Purporses of replication:
* high availability - the system is running even when one machine or several machines go down 
* latency - data is geographically close to users, thus user interaction is faster 
* disconnected operation - application is working even when there is a network interruption 
* scalability - being able to handle a higher volume of reads

<br/>

Three main approaches to replication:
* single-leader replication 
  * client send all writes to a single node (the leader), which sends a stream of data change events to the other replicas (followers)
  * reads cna be performed on any replica, but reads from followers might be stale 
  * popular, easy to understand, no conflict resolution to worry about 
* multi-leader replication
  * clients send each write to one of several leader nodes, any of which can accept writes 
  * the leaders send streams of data change events to each other and to any follower nodes 
  * more robust, but weak consistency guarantees 
* leaderless replication
  * clients send each write to several nodes, and read from several nodes in parallel in order to detect and correct nodes with stale data 
  * same as MLR - weak consistency guarantees 
  
<br/>

Techniques how an application should behave under replication lag:
* read-after-write consistency - users should always see data that they submitted themselves 
* monotonic reads - after users have seen the data at one point in time,they shouldn't later see the data from some earlier point in time 
* consistent prefix reads - users should see the data in a state that makes casual sense: for example, seeing a question and its reply in the correct order 

<br/>

Techniques to implement read-after-write consistency in a system with leader-based replication:
* When reading something something that the user may have modified, read it from the leader, otherwise read it from a follower. This required that you have some way of knowing whether something might have been modified, without actually querying it. FOr example, user profile information on a social network is normally editable only by the owner of the profile, not by anybody else. Thus, a simple rule is: always read the user's own profile from the leader, and any other users' profiles from a follower. 
* If most things in the application are potentially editable by the user, that approach won't be effective, as most things would have to be read from the leader (negating the benefit of read scaling). In that case, other criteria may be used to decide whether to read from the leader. You could also monitor the time of the last update and, for one minute after the last update, make all reads from the leader. You could also monitor the replication lag on followers and prevent queries on any follower that is more than one minute behind the leader. 
* The client can remember the timestapm of the its most recent write - then the system can ensure that the replica serving any reads from that user reflects updates at least until that timestamp. If a replica is not sufficiently up to date, either the read can be handled by another replica or the query can wait until the replica has caught up. The timestamp could be a logical timestamp (something that indicates ordering of writes, such as the log sequence number) or the actual system clock (in which case clock synchronization becomes critical). 

<br/>

Additional issue to consider:
* if user is accessing service from multiple devices, you may want to provide cross-device read-after-write consistency 
* metadata (timestamp of the user's last update) will need to be centralized, so to be seen from all devices 
* if your replicas are distributed accross different datacenters, there is no guarantee that connections from different devices will be routed to the same datacenter

<br/>

Monotonic reads
* if one user makes several reads in sequence, they will not see time go backwrd - i.e., they will not read older data after having previously read newer data
* one way of achieving it is making sure that each user always makes their reads from the same replica 
* the replica can be chosed based on a hash of the user ID, rather than randomly 
* if that replica fails, the user's queries will need to be rerouted to another replica 


 <img src="https://user-images.githubusercontent.com/38294198/195819008-4afe50cf-bca4-46bf-a8ed-161b048bd3ed.png" width="550" />
 
 <br/>
 
 Consistent prefix reads
 * if a sequence of writes happens in a certain order, then anyone regarding those writes will see them appear in the same order 


#### Multi-Leader Replication
Advantages of multi-reader replication:
* better performance 
* better tolerance of datacenter outages
* better tolerance of network problems 

<br>

Problems:
* autoincrementing keys, triggers and integrity constraints can be problematic
* multi-reader replication is often considered dangerous territory that should be avoided if possible

<br>

Common use case when multi-leader replication is appropriate:
* apps on mobile phone, laptop and other decies - every device has a local database that acts as a leader (it accepts write requests) and there is an asynchronous multi-leader replication process (sync) between the replicas of all devices
* real-time collaborative editing applications, which allow several people to edit a document simultaneously

<br>

Ways of achieving convergent conflict resolution:
* give each write a unique ID (e.g., a timestamp, a long random number, a UUID, or a hash of the key and value), prick the write with the highest ID as the winner and throw away the other writes
* if a timestamp is used, this technique is known as last write wins (LWW) -> popular approach, but dangerously prone to data loss 
* give each replica a unique ID and let writes that originated at a higher-numbered replica always take precedence over writes that originated at a lower-numbered replica -> this approach implies data loss
* merge values together - for example order them alphabeticallz and then concatenate them 
* record the conflict in an explicit data structure that preserves all information, and write application code that resolves the conflict at some later time (perhaps by prompting the user)

<br>

Multi-Leader Topologies
* the most general is all-to-all
* however, more restricted topologies are also used - MySQL by default supports only a circular topology
* a problem with circular and star topologies is that if just one node fails, it can interrupt the flow of replication messages between other nodes 
* the fault tolerance of a more densely connected topology (such as all-to-all) is better because it allows messages to travel along different paths, avoiding a single point of failure 

 <img src="https://user-images.githubusercontent.com/38294198/195837268-c8eda8bb-71c6-4675-a659-de27c3598589.png" width="550" />
 
<br>

 Leaderless Replication
* systems allows any replica to directly accept writes from clients 
* in some implementations the client directly sends its writes to several replicas, while in others, a coordinator node does this on behalf of the client (the coordinator doesn't enfore a particular ordering of writes, unlike a leader database)
* the problem with node being offline is solved by sending a read request to sevelar nodes in patallel
* version numbers are used to determin which value is newer
* approach to achieving eventual convergence is to declare that each replica need only store the most recent value and allow older values to be overwritten and discarded (last write wins)

<br>

### Partitioning
* the goal of partitioning is t ospread the data and query load evenly across multiple machines, avoiding hot spots (nodes with disproportionately high load)

<br>

Two main approaches to partitioning:
* key range partitioning
 * keys are sorted, and a partition owns all the keys from some minimum up to some maximum
 * sorting has the advantage that efficient range queries are possible, but there is a risk of hot spots if the application often accesses keys that are close together in the sorted order
 * in this approach, partitions are typically rebalanced dynamically by splitting the range into two subranges when a partition gets too big
* hash partitioning
 * a hash function is applied to each keyy, and a partition owns a range of hashes
 * this method destroys the ordering of keys, making range queries inefficient, but may distribute load more evenly
 * when partitioning by hash, it is common to create a fixed number of partitions in advance, to assign several partitions to each node, and to move entire partitions from one node to another when nodes are added or removed
 * dynamic partitioning can also be used

<br>

Approaches to the problem of finding out to which partition to route a request (service discovery problem):
* allow client to contact any node (e.g., via a round-robin load balancer) - if that node coincidentally owns the partition to which the request applies, it can handle the request directly; otherwise, it forwards the request to the appropriate node, receives the reply, and passes the reply along to the client
* send all requests from clients to a routing tier first, which determines the node that should handle each request and forwards it accordingly - this routing tier does not itself handle any requests, it only acts as a partition-aware load balancer
* require that clients be aware of the partitioning and the assignment of partitions to nodes - in this case, a client can connect directly to the appropriate node, without any intermediary 

 <img src="https://user-images.githubusercontent.com/38294198/196003489-26a431c5-7980-4cf0-be4b-3ec932fb2a69.png" width="550" />

<br>

### Transactions
Race conditions:
* dirty reads  
 * one client reads another client's writes before they have been committed
 * the read committed isolation level and stronger levels prevent dirty reads
* dirty writes
 * one client overwrites data that another client has written, but not yet committed
 * almost all transaction implementations prevent dirty writes
* read skew
 * a client sees different parts of the database at different points in time
 * some cases on read skew are also known as *nonrepeatable reads*
 * this issue is most commonly prevented with snapshop isolation, which allows a transaction to read from a consistent snapshot corresponding to one particular point in time - it's usually implemented with *multi-version concurrency control* (MVCC)
* lost updates
 * two clients concurrently perform a read-modify-write cycle 
 * one overwrites the other's write without incorporating its changes, so data is lost
 * some implementations of snapshot isolation prevent this anomaly automatically, while other require a manual lock (SELECT FOR UPDATE)
* write skew
 * a transaction reads something, makes a decision based on the value it saw, and writes the decision to the database 
 * however, by the time the write is made, the premise of the decision is no longer true
 * only serializable isolation prevents this anomaly
* phantom reads
 * a transaction reads objects that match some search condition
 * another client makes a write that affects the results of that search
 * snapshot isolation prevents straight forward phantom reads, but phantoms inthe context of write skew require special treatment, such as index/range locks 

<br>

Solution to all of these issues isserializable isolation. Three different approaches:
* literally executing transactions in a serial order 
 * simple and effective option, if you can make each transaction very fast to execute, and the transaction throughput is low enough to process on a single CPU core
* two-phase locking
 * for decades this has been the standard way of implementing serializability, but many applications avoid using it because of its performance characteristics 
* serializable snapshot isolation (SSI)
 * new algorithm that avoids omst of the downsides of the previous approaches
 * uses an optimistic approach, allowing transactions to proceed without blocking
 * when a transaction wants to commit, it is chacked, and it is aborted if the execution was not serializable 

<br>

### The trouble with distributed systems
Building a system from unreliable components
* in computing there is an idea to contruct a more reliable system from a less reliable underlying base
* for example, IP is unreliable, but TCP built on top of IP ensures that missing packets are retransmitted, duplicates are eliminated, and packets are reassembled into the order in which they were sent
* it is thus not the case that a system can only be as reliable as its least reliable component (its weakest link)

<br>

Common problems which can occur in distributed system:
* whenever you try to send a packet over the network, it may be lost or arbitary delayed - likewise, the reply may be lost or delayed, so if you don't get a reply, you have no idea whether the message got through
* a node's slock may be significantly out of sync with other nodes (despite your best efforts to set up NTP), it may suddenly jump forward or back in time, and relying on it is dangerous because you most likely don't have a good measure of your clock's confidence interval
* a process may pause for a substantial amount of time at any point in its execution (perhaps due to a stop-the-world garbage collector), be declared dead by other nodes, and then come back to life again without realizing that it was paused 

<br>

Two different kinds of clocks:
* a time-of-day clock
 * returns the current date and time according to some calendar (for example the number of seconds since the epoch - midnight UTC on January 1, 1970, according to the Gregorian calendar, not counting leap seconds)
 * some of these clocks are usually synchronized with NTP - timestamp on one machine means the same as a timestamp on another machine 
 * the issue with it may appear if the local clock is too far ahead of the NTP server, then it may be forcibly reset and appear to jump back to a previous point in time - these jumps, as well as similar jumps caused by leap seconds, make time0of0dat clocks unsuitable for measuring elapsed time
* monotonic clock
 * suitable for measuring a duration, such as a timeout or a service's response time 
   




















