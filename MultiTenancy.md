The Library Simplified circulation manager can host circulation services for any number of libraries. The goal here is to reduce the hosting cost for small libraries and library consortia.

# When does it work?

Here are some use cases that benefit from multi-tenancy.

* Two (or more) libraries share all of their collections. In this case, every library after the first one can be 
  hosted for free.

  These libraries are like a family who live in the same house and share everything. As they say, two can live as 
  cheaply as one.

* Two (or more) libraries share a large collection, but each also has an additional collection which is not 
  shared. Hosting the second library isn't free, but hosting these libraries on the same circulation manager is 
  probably simpler and cheaper than giving each its own circulation manager.
  
  These libraries are like the inhabitants of an apartment building with a shared laundry room. It would be 
  wasteful for everyone to also have their own in-unit laundry.
* Two (or more) libraries have no resources in common, but neither has a large collection. These libraries can 
  split the cost of a single circulation manager.
  
  These libraries are like roommates who don't interact with each other at all, but who don't need much space.

* Two libraries have no resources in common, but they are located on opposite sides of the world. While people on 
  one side of the world are asleep, the patrons of the other library are borrowing books. The circulation manager 
  can serve twice as many patrons with no increase in peak demand.
  
  These libraries are like sailors on a submarine who work different shifts and share a bunk. At any given time, 
  the bunk is occupied. The submarine needs fewer bunks than it has sailors.

Here's the only case where it's known that multi-tenancy doesn't help:

* Two libraries with lots of patrons have large collections that are completely disjoint and not shared. Putting these two libraries on the same circulation manager would be like trying to make two mansions share a driveway. It's theoretically possible but the benefit would be small.

# Why does it work?

The scalability advantages of multi-tenancy come from two places:

1. A hosted circulation manager comes with a large startup and maintenance cost. If N libraries can live on the same server, the startup cost is reduced by a factor of N, and the maintenance cost by a slightly smaller factor.
2. When N libraries share a collection (e.g. an Overdrive collection), but use different databases, 
the amount of work necessary to keep that collection up-to-date with the source of truth (e.g. Overdrive) is N times greater than if they were all using the same database. Put them on the same circulation manager, and they start using the same database.

# Why not put everyone on one circulation manager?

Two libraries can share a circulation manager even if they use different ILS systems and don't have any collections in common. In fact, it's theoretically possible to host every library in the United States on a single clustered circulation manager. This isn't even an obviously bad idea -- this is basically what Overdrive does. It's instructive to examine the reasons why that's not a practical solution.

First, libraries like to keep local control. Putting everyone on the same circulation manager would create a single point of technical and political failure for all libraries in the United States. At the same time, there's no reason all members of a library consortium shouldn't be able to live on the same circulation manager--they've already agreed to give up some local control in the name of efficiency.

Second, the "script runner" component of the circulation manager is designed to keep track of a small number of collections. If it has to keep track of hundreds of collections, information on any particular collection won't be updated very often. This can be improved; we just haven't tackled the problem yet.

Third, the more simultaneous patrons a circulation manager has to handle, the bigger the cluster has to be. A surge in demand on one library can affect the performance of another. For a big  organization with a large dev ops group, automatically scaling the cluster isn't a problem, but for a smaller organization it might be smarter to set up two clusters and to scale them separately.
