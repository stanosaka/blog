---
title: splunk
date: 2019-07-15 10:40:04
tags: 
  - splunk
  - monitoring
---
[code](https://github.com/PacktPublishing/Splunk-7.x-Quick-Start-Guide)
# What is Splunk?
Splunk is a software platform that collects and stores all this machine data in one place.
![splunk data sources and use cases](https://i.imgur.com/wObsIBw.png)

## Splunk products
- **Splunk Enterprise**: designed for on-premise deployments
- **Splunk Cloud**: a cloud-based **software as a service (SaaS)** version of Splunk Enterprise.
- **Splunk Light**: is designed to be a small-scale solution.
- **Splunk Free**: is a free version of the core Splunk Enterprise product that has limits on users(one user), ingestion volume (500 MB/day), and other features.

**Splunk components**
- Universal forwarder
- Indexer and indexer clusters
- Search head and search head clusters
- Deployment server
- Deployer
- Cluster master
- License master
- Heavy forwarder
Universal forwarders, indexers, and search heads constitute the majority of Splunk functionality; the other components provide supporting roles for larger clustered/distributed environments. 
![Splunk components in a distributed deployment](https://i.imgur.com/sqJ8esH.png)

The **universal forwarder (UF)** is a free small-footprint version of Splunk Enterprise that is installed on each application, web, or other type of server to collect data from specified log files and forward this data to Splunk for indexing(storage). In A large Splunk deployment, you may have hundreds or thousands of forwards that consume and forward data for indexing.

An **indexer** is the Splunk component that creates and manages indexes, which is where machine data is stored. Indexers perform two main functions: parsing and storing data, which has been received from forwarders or other data sources into indexes, and searching and returning the indexed data in response to search requests.

An indexing cluster is a group of indexers that have been configured to work together to handle higher volumes of both incmoing data to be indexed and search requests to be serviced, as well as providing redundancy by keeping duplicate copies of indexed data spread across the cluster members.

A **search head** is an instance of Splunk Enterprise that handles search management functions. This includes providing a web-based user interface called Splunk Web, from which users issue search requests in what is called **Search Processing Language (SPL)**. Search reqeusts initiated by a user ( or a report or dashboard) are sent to one or more indexers to locate and return the requested data; the search head then formates the returned data for presentation to the user.

```
index=_internal | stats count by source, sourcetype
```
an example of executing a simple search in Splunk Web. The SPL specifies searching in the `_internal` index, which is where Splunk saves data about its internal operations, and provides a count of the number of events in each log for Today. The SPL command specified an `index`, and then pipes the returned results to the `stats` command to return a `count` of all the events by their `source` and `sourcetype
![Simple search in Splunk Web](https://i.imgur.com/exMXaN5.jpg)

A **deployment server** is a Splunk Enterprise instance that acts as a centralized configuration manager ofr a number of Splunk components, but which in practice is used to manage UFs.

A **deployer** is a Splunk Enterprise instance that is ued to distribute Splunk apps and certain other configuration updates to search head cluster memebers.

A **cluster master** is a Splunk Enterprise instance that coordinates the activities of an indexing cluster.

A **license master** is a single Splunk Enterprise instance that provides a licensing service for the multiple instances of Splunk that have been deployed in a distributed environment.

A **heavy forwarder** is an instance of Splunk Enterprise that can receive data from other forwarders or data sources and parse, index, and/or send data to another Splunk instance for indexing. 

Splunk Enterprise also has a monitoring tool function called the **monitoring console**, which lets you view detailed topology and performance information about your entire distributed deployment from one interface. 

![Splunk data pipeline](https://i.imgur.com/ElU7rTv.png)



