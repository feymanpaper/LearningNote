As a rule of thumb, LSM-trees are typically faster for writes, whereas B-trees are thought to be faster for reads.  
```
as a rule of thumb -- 根据经验
```
In this section, we will briefly discuss a few things that worth considering when measuring the performance of a storage engine.

This effect—one write to the database resulting in multiple writes to the disk over the course of the database’s lifetime—is known as write amplification.
It is of particular concern on SSDs, which can only overwrite blocks a limited number of times before wearing out
```
over the course of the databse's lifetime 在xxx的生命周期
wear out 磨损
```
As RAM becomes cheaper, the cost-per-gigabyte argument is eroded
```
eroded 减弱,侵蚀
```
At first, the same databases were used for both transaction processing and analytic queries. SQL turned out to be quite flexible in this regard: it works well for OLTP-type queries as well as OLAP-type queries.
```
in this regard 在这方面
```
One solution is to make sure that any writes that are causally related to each other are written to the same partition—but in some applications that cannot be done efficiently.
```
causal 因果的
casual 随意的
```
Pretending that replication is synchronous when in fact it is asynchronous is a recipe for problems down the line.
```
a recipe for problems down the line
```
It rarely makes sense to use a multi-leader setup within a single datacenter, because the benefits rarely outweigh the added complexity.
```
it makes sense to start savingearly for higher education
intelligible, justifiable, or practicable
```
In the next chapter we will continue looking at data that is distributed across multiple machines, through the counterpart of replication: splitting a large dataset into partitions.
```
"Counterpart" 表示对应的，而不是反方。它指的是某人或某物在另一方或另一种情况中的对等或相当的事物

Counter" 作为单独的词可以表示反对或相对的意思，比如"Counter argument"表示反方论点
```
However, partitioning is the most established term, so we’ll stick with that.
```
stick with that
```
However, the downside of key range partitioning is that certain access patterns can lead to hot spots.
```
downside
example: what's the downside of this approach?
```
This approach to querying a partitioned database is sometimes known as scatter/ gather, and it can make read queries on secondary indexes quite expensive. Even if you query the partitions in parallel, scatter/gather is prone to tail latency amplification.
```
approach to doing sth
```
There is one important question with regard to rebalancing that we have glossed over: does the rebalancing happen automatically or manually?
```
with regard to sth
gloss over
```
Many distributed datastores have abandoned multi-object transactions because they are difficult to implement across partitions, and they can get in the way in some scenarios where very high availability or performance is required.
```
get in the way 
```
Concurrency issues (race conditions) only come into play when one transaction reads data that is concurrently modified by another transaction, or when two transactions try to simultaneously modify the same data.
```
come into play
```
So far in this chapter we have explored the ways in which distributed systems are different from programs running on a single computer: there is no shared memory, only message passing via an unreliable network with variable delays, and the systems may suffer from partial failures, unreliable clocks, and processing pauses.
```
explored the ways in which distributed systems are different from programs
```
We have to reckon with the fact that any timing assumptions may be shattered occasionally
```
reckon with the fact
shatter
```
This chapter has been all about problems, and has given us a bleak outlook. In the next chapter we will move on to solutions, and discuss some algorithms that have been designed to cope with all the problems in distributed systems.
```
cope with 
```
