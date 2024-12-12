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