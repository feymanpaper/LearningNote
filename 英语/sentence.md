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