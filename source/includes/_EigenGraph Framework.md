# EigenGraph Framework (EGF) Modes

EGF can be deployed in three different modes:

1. Big Data - this mode will include Spark and either HDFS or S3 persistence layer in addition to a data store. Suitable for Big Data applications
2. Scalable - this mode is suitable for applications with any load level but without ML and Big Data processing requirements.
3. Small mode uses RethinkDB changes feed as a queue solution and thus micro services are not horizontally scalable. The cheapest option, suitable for small apps with little load and for development purposes

Each of the modes illustrated below with more details.

## Big Data Mode

![](EigenGraphBigData.png "EigenGraph Data Layer (Big Data)")

1. Client-data processes GET requests as follows: 
   * GET object request will use caching solution and in case object is not cached it will use data store solution 
   * GET edge request will retrieve edge data from data store and resolve objects using cache and, in some instances, data store
2. For modifying requests client-data updates the cache
3. For modifying requests client-data sends events to the queue solution 
4. For modifying requests client-data stores data in the data store
5. Spark streaming is listening for changes from queue
6. Spark streaming updates master data store in S3
7. Spark streaming updates ES
8. Micro services listen to the queue and react to events
9. Micro services use client-data to store / retrieve data
10. Batch jobs will send updates to client-data
11. Batch jobs will operate on data from the master data set

## Scalable Mode

![](EigenGraphScalable.png "EigenGraph Data Layer (Scalable)")

1. Client-data processes GET requests as follows:
  * GET object request will use cache and in case object is not cached it will use data store
  * GET edge request will retrieve edge data from data store and resolve objects using cache and, in some instances, data store
* For modifying requests client-data updates the cache
* For modifying requests client-data stores data in the data store
* For modifying requests client-data sends events to the queue solution 
* Micro services listen to the queue
* Micro services work with client-data to retrieve / modify data
* Sync service updates ES based on events coming from the queue solution
* Client-api uses ES to search for data

## Small Mode

![](EigenGraphSmall.png "EigenGraph Data Layer (Small)")

1. Client-api works with client-data to retrieve / modify data
* Client-data uses RethinkDB to store / get data
* Micro services work with client-data to store / get data
* Micro services listen to RethinkDB changes feed, process changes with events
* Sync service updates ES
* Client-api uses ES to search for data
