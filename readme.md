# Udemy Lecture: Complete Guide to Elasticsearch
[course](https://www.udemy.com/elasticsearch-complete-guide/learn/v4/overview)
[repo](https://github.com/codingexplained/complete-guide-to-elasticsearch)

## Section 1 - Getting Started

### Lecture 2 - Intro to Elasticsearch

* Elasticsearch is an analytics and full text search engine. it enables search functionality in application. 
* We can query structured data like numbers.
* Data si stored as JSON objects.so we can search anything that can be added to a JSON object.
* Elastic search is highly scalable and offers near realtime performance.
* A tool named Kibana is often used to visualize data to add BI
* We can perform powerful analytic queries on an Elastic cluster.
* Elastic search is  able to find,retrieve and analyze data.
Elastic stores data in a way optimized for searching.
* A common pattern is to store data in a DB and ingest thsese data to Elastic.
* Elasticsearch focuses on searching data. It persists data on a nonstandard way. so that we can restore data in cluster from another data source. it offers replication but does not support transactions
* It is written in Java on Apache Lucene.
* It provides a REST API for JSON.
* It is distributed by nature and scalable.

### Lecture 3 - Overview of the ELK Stack (ELK+)

* ElK stands for Elasticsearch, Logstash, Kibana
* Elastic is the search tool
* Logstash is a tool to get data into the Elasticsearch cluster (originally built to add logs to Elasticsearch). It is a dat aprocessing pipeline that can ingest data from varius sources. It can transform and filter data before stashing them to a datastore.
* THe datastore naturally is Elasticsearch but MongoDB, Redis can be used.
* A common flow for logstash is to get logs parse and filter them ans save them to a store.
* Kibana is an analytics and visualization platform. It supports a number of diagrams and graphs out of the box. it works well with Elasticsearch aggregations. find trends in data, ploting
* Elastic Stack is a superset of ELK stack as it adds Beats and x-pack.
* Beats is a way to get hold on data to stash in elastic. It contains a number of data shippers (called beats). they can be installed on servers and send data to logstash for processing or directly to an elastic cluster.
* There is a Filebeat for handling logfiles and a metricbeat for collectig CPU or memory, a Packetbeat for collecting uptime data or we can build a custom beat
* x-pack adds security capbilies to the elastic cluster and offers alerting monitoring etc. we can integrate with LDAP and add login to our Kibana dashboard. manage users and roles and give permissions on files, docum,ents, indexes or fields. and alerts in unusual behaviour. add reporting.
X-Pack is clustrer manager

## Section 2 - Architecture of Elasticsearch

### Lecture 5 - Nodes & Clusters

* a node is a server that stores data and is part of a cluster
* a cluster is acollection of nodes. eachj node contains a part of the cluster's data
* the collection of nodes contains clusters entire data set
* each node 
	* participates in the indexing and search capabilitites of the cluster
	* participates in a query by searching the data it stores
	* supports searching its data and indexing new data, or manipulating existing data
	* handles HTTP requests for clients that want to send requerst to the cluster through the HTTP REST API the cluster exposes
	* knows about every node and can forward requests to a given node and can forward requests using transport layer, while HTTP layer is used for external clients
* a node receives the requset and is responsible to coordinate the rest of work
* each node may be assigned to be the master node. master node coordinates changes to the cluster (adding/removing nodes, create remove indices)
* master is the only who can updates the state of the cluster
* clusters and nodes are identified by unique names. the default name for the clusetr is *elasticsearch* the default name for nodes is a UUID. 
* names of nodes are important as this si how you can identify which physical or VMs correspond to each elasticsearch node
* by default nodes join a cluster named *easticsearch* but we can configure them to join a specific cluster with custom name. But if we start a number of nodes on our network they will join a cluster called elasticsearch and if no cluster with this name exists it will be formed. 
* its good to change the default name in a production environment, so that no nodes accidentaly join the prod cluster, whil eperforming maintenance or developing on same network
* all in all its not good idea our dev machine to operateon the productionnetwork
* the cluster can have from 1 to n nodes
* elasticsearch has innate scalability and can handle massive amounts of data

### Lecture 6 - Indices & Documents

* Each data item we store in the cluster is called a document.
* a document is a basic unit of info that can be indexed
* documents are JSON objects similar to rows ina Relational DB
* if we would like to store a person, we could add an object with name and birthday property
* where are these JSON objects stored? data are stored across all nodes int he cluster
* how documents are organized? docs are stored within sthing called indices
* index is a collection of objects with similar characteristics (e.g an index for product data, one for customer data, one for orders). an index can have arbitrary number of docs. 
* docs have ids assigned. automatically by elasticsearch or by us when adding them to an index. a doc is uniquely identified by the index it belongs and its id
* indices are indentified by names (all lowercase)
* docs are added to indices, indices are collection of docs. docs are JSON obj

### Lecture 7 - A word on types

* [index vs type](https://www.elastic.co/blog/index-vs-type)
* Apart from inidces elastidsearch uses types. types are phased out and are obsolete. tutor added a type default. so when he specs the word default within an index he refers to a type 

### Lecture 8 - Sharding

* elasticsearch is scalable due to its distributed architecture.
* one reason for this is sharding.
* suppose we have an index with lots of docs totalling 1TB of data. we have 2 nodes in the cluster each with 512GB available for storing data. the entire index wont fit in any of nodes. we have to split index somehow to avoid running out of disk space
* in cases when the size of an index exceeds the HW limits, sharding come to the rescue
* Sharding dicvides indices into smaller pieces called shards
* shard will contain a subset of an index data and is fully functional and independent, like an independent index
* when an index is sharded, a document in that index will be stored within one of the shards
* shards can be hosted on any node in the cluster
* an indexes shards wont necessarily be distributed across multiple physical or virtual machines. this depends on the number of nodes in our cluster.
* sharding is important because: it allows to split and scale volumes of data. in growing amounts of data we dont have bottleneck as we can tweak the number of shards  for this index
* with shards operations can be distributed across multiple nodes and parallelized (faster) transparent from client
* we can specify the number of shards at index creation time (if not the default is 5)
* if index has been created we cannot change the number of shards. we make anew index with the num of shards we want aand move the data to the new index

### Lecture 9 - Replication

* replication = fault tolerance & failover
* elasticsearch natively supports replication of the shards. shards are copied
* when a shard is replicated it is  referred as replica shard or just replica
* shards that have been replicated are referred as primary shards
* a primary shard and its replicasa re referred as a replication group
* replication gives HA (high availability) in case nodes or shards fail
* replica shards are never allocated to the same node as primary shards
* if an entire node fails we will have at least one replica of any shard svailable.
* a side benefit of replication is increased performance for search queries. queries can be executed on all replicas n parallel. replicas are part of clusters search capabilities
* nu of replicas is defined when creating an index (default is 1). by default a cluster with more than 1 noe will have 5 pramiary shards and 5 replicas (10 shards per index)

### Lecture 10 - Keeping replicas synchronized

* how replicas are updated when data is changed or removed? replicas need to be kept in sync to avoid trouble of data incosistency between replicas
* to keep replicas in sync elasticsearch uses a model named primary-backup for data replication
* primary shard in a replication group acts as an enty pont for indexing operations. 	* all operations that affect the index (ceate update remove  data) are sent to the primary shard
* primary shard then is responsible for validating operations and ensuring that all is good. (checking is request is structurally invalid e.g trying to add a number to an object field)
* when the operation is accepted by the primary shard it will be performed locally onthe primary shard itself.
* when operation completes it will be forwarded to each of the replica shards in the group
* if the hard has multiple replicas the operation will be performed in parallel on each one
* when the operation has completed successfully on every replica and responded to the proimary shard, the primary shard will respond to the client that the operation was successful.

### Lecture 11 - Searching for Data

* what we see as a dev runnign queries towards an elasticsearch cluster. we usually have aclient app sending search queries over http. the cluster based on the index and query we specify in the http request does the search. when  results are ready the cluster responds with the results.
* most users see elasticsearch cluster as a black box on whichthey run queries. they dont care how elasticsearch gather the results internally.
* Suppose we have a 3 node cluster with 1 index distributed across 3 shards (A,B,C). each shard has 2 replicas. a replication group consists of one primary and 2 replicas
* a client sends a search request to a cluster which ends up ona node waving primary shard B. this becomes the coordinating node for this query and sends queries to the other nodes,a ssembles results and sends them back to client
* any node in cluster may act as coordinating node and receive HTTP requests. as coordinating node contains serchable shard it will perform the query itself. the ccordinating node then broadcasts the request to every shard in the index primary and replicas. how the coordinating node decides which shards will get the request is out of scope. when the other nodes reply. coordinating node merges the results and sorts them returning them to client. if we serach by ID the request is routed to the apropriate shard and not broadcasted

### Lecture 12 - Distributing documents across shards

* we saw how data is stored on1 or more nodes with sharding
* how elasticsearch knows on which shard t store a new document and how it finds it when retreived by id? docs should be distributed evenly across shards
* determining on which shard a doc will be or has been stored is called ROUTING.
* Routing is done  automatically. elasticsearch uses a simple routing formula. 
	* by default routing value is equal to a given documents ID 
	* the routing vak is hashed to generate a num to be used in the division
	* the shard num is the modulo of the hash divided by the number of shards
	* `shard = hash(routing) % total_primary_shards`
	* custom routing is also accepted (but it needs thoght to spread evenly)
* num of shards cannot changed after index creation. the reason is the routing formula. 
* for this reason we should never change routing formula after index creation

### Lecture 13 - Wrap Up

## Section 3 - Installing ElasticSearch & Kibana

* Although we would prefer to install them on docker containers we will follow the course for learning purpose

### Lecture 14 - Instaling ElasticSearch on Mac/Linux

* [Java Download](http://www.oracle.com/technetwork/java/javase/downloads/index.html)
* [Elasticsearch Download](https://www.elastic.co/downloads/elasticsearch)
* Elastic is not installed but executed. its dependency is Apache Lucene and is packaged as .jar file
* >= java8 is needed (we have it)
* we download elasticsearch as a tarball and extract it (tar -zxf)
* lib folder contains jars, config contains config files (elastic, jvm, log4j), bin folder contains binaries (win, linux,mac)
* we start it localy by entering the bin direcory and runing `./elasticsearch` in the console
* we can shutdown the node with Ctrl+C
* we can test that elasticsearch runs by opening another terminal use curl to localhost:9200 `curl http://localhost:9200`. elastic search default port is 9200. we get back a JSON reply

```
{
  "name" : "MtGZ0aF",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "J92dI1TrSK-hKOGpSiBd6g",
  "version" : {
    "number" : "6.2.3",
    "build_hash" : "c59ff00",
    "build_date" : "2018-03-13T10:06:29.741383Z",
    "build_snapshot" : false,
    "lucene_version" : "7.2.1",
    "minimum_wire_compatibility_version" : "5.6.0",
    "minimum_index_compatibility_version" : "5.0.0"
  },
  "tagline" : "You Know, for Search"
}
```

### Lecture 17 - Configuring Elasticsearch

* 

### Lecture 18 - Installing Kibana on Mac/Linux

* [Kibana Download](https://www.elastic.co/downloads/kibana) 

### Lecture 20 - Configuring Kibana

* [Kibana Config Options](https://www.elastic.co/guide/en/kibana/current/settings.html)


### Lecture 22 - intro to Kibana and dev tools