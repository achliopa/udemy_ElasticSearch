# Udemy Lecture: Complete Guide to Elasticsearch
[course](https://www.udemy.com/elasticsearch-complete-guide/learn/v4/overview)
[repo](https://github.com/codingexplained/complete-guide-to-elasticsearch)

## IMPORTANT UPDATE: since version 7 types will be removed and default type should be replaced with the *_doc* type 

## Section 1 - Getting Started

### Lecture 2 - Intro to Elasticsearch

* Elasticsearch is an analytics and full text search engine. it enables search functionality in application. 
* We can query structured data like numbers.
* Data is stored as JSON objects.so we can search anything that can be added to a JSON object.
* Elastic search is highly scalable and offers near realtime performance.
* A tool named Kibana is often used to visualize data to add BI
* We can perform powerful analytic queries on an Elastic cluster.
* Elastic search is  able to find,retrieve and analyze data.
* Elastic stores data in a way optimized for searching.
* A common pattern is to store data in a DB and ingest these data to Elastic.
* Elasticsearch focuses on searching data. It persists data on a nonstandard way. so that we can restore data in cluster from another data source. it offers replication but does not support transactions
* It is written in Java on Apache Lucene.
* It provides a REST API for JSON.
* It is distributed by nature and scalable.

### Lecture 3 - Overview of the ELK Stack (ELK+)

* ELK stands for Elasticsearch, Logstash, Kibana
* Elastic is the search tool
* Logstash is a tool to get data into the Elasticsearch cluster (originally built to add logs to Elasticsearch). It is a data processing pipeline that can ingest data from varius sources. It can transform and filter data before stashing them to a datastore.
* THe datastore naturally is Elasticsearch but MongoDB, Redis can be used.
* A common flow for Logstash is to get logs, parse, filter and save them to a store.
* Kibana is an analytics and visualization platform. It supports a number of diagrams and graphs out of the box. it works well with Elasticsearch aggregations. find trends in data, ploting
* Elastic Stack is a superset of ELK stack as it adds Beats and x-pack.
* Beats is a way to get hold on data to stash in elastic. It contains a number of data shippers (called beats). they can be installed on servers and send data to logstash for processing or directly to an elastic cluster.
* There is a Filebeat for handling logfiles and a metricbeat for collectig CPU or memory, a Packetbeat for collecting uptime data or we can build a custom beat
* X-pack adds security capbilies to the elastic cluster and offers alerting, monitoring etc. we can integrate with LDAP and add login to our Kibana dashboard. manage users and roles and give permissions on files, documents, indexes or fields. and alerts in unusual behaviour. add reporting.
* X-Pack is clustrer manager

## Section 2 - Architecture of Elasticsearch

### Lecture 5 - Nodes & Clusters

* a node is a server that stores data and is part of a cluster
* a cluster is a collection of nodes. each node contains a part of the cluster's data
* the collection of nodes contains cluster's entire data set
* each node:
	* participates in the indexing and search capabilitites of the cluster
	* participates in a query by searching the data it stores
	* supports searching its data and indexing new data, or manipulating existing data
	* handles HTTP requests for clients that want to send request to the cluster through the HTTP REST API the cluster exposes
	* knows about every node and can forward requests to a given node using transport layer, while HTTP layer is used for external clients
* a node receives the request and is responsible to coordinate the rest of work
* each node may be assigned to be the master node. master node coordinates changes to the cluster (adding/removing nodes, create remove indices)
* master is the only who can update the state of the cluster
* clusters and nodes are identified by unique names. the default name for the clusetr is *elasticsearch* the default name for nodes is a UUID. 
* names of nodes are important as this is how you can identify which physical or VMs correspond to each elasticsearch node
* by default nodes join a cluster named *easticsearch* but we can configure them to join a specific cluster with custom name. But if we start a number of nodes on our network they will join a cluster called elasticsearch and if no cluster with this name exists it will be formed. 
* its good to change the default name in a production environment, so that no nodes accidentaly join the prod cluster, while performing maintenance or developing on same network
* all in all its not good idea our dev machine to operate on the production network
* the cluster can have from 1 to n nodes
* elasticsearch has innate scalability and can handle massive amounts of data

### Lecture 6 - Indices & Documents

* Each data item we store in the cluster is called a document.
* a document is a basic unit of info that can be indexed
* documents are JSON objects similar to rows in a Relational DB
* if we would like to store a person, we could add an object with name and birthday property
* where are these JSON objects stored? data are stored across all nodes in the cluster
* how documents are organized? docs are stored within something called indices
* index is a collection of objects with similar characteristics (e.g an index for product data, one for customer data, one for orders). an index can have arbitrary number of docs. 
* docs have ids assigned. automatically by elasticsearch or by us when adding them to an index. a doc is uniquely identified by the index it belongs and its id
* indices are identified by names (all lowercase)
* docs are added to indices, indices are collection of docs. docs are JSON obj

### Lecture 7 - A word on types

* [index vs type](https://www.elastic.co/blog/index-vs-type)
* Apart from inidces elasticsearch uses types. types are phased out and are obsolete. tutor added a type default. so when he specs the word default within an index he refers to a type 

### Lecture 8 - Sharding

* elasticsearch is scalable due to its distributed architecture.
* one reason for this is sharding.
* suppose we have an index with lots of docs totalling 1TB of data. we have 2 nodes in the cluster each with 512GB available for storing data. the entire index won't fit in any of nodes. we have to split index somehow to avoid running out of disk space
* in cases when the size of an index exceeds the HW limits, sharding come to the rescue
* Sharding divides indices into smaller pieces called shards
* shard will contain a subset of an index data and is fully functional and independent, like an independent index
* when an index is sharded, a document in that index will be stored within one of the shards
* shards can be hosted on any node in the cluster
* an indexes shards wont necessarily be distributed across multiple physical or virtual machines. this depends on the number of nodes in our cluster.
* sharding is important because: it allows to split and scale volumes of data. in growing amounts of data we dont have bottleneck as we can tweak the number of shards  for this index
* with shards operations can be distributed across multiple nodes and parallelized (faster) transparent from client
* we can specify the number of shards at index creation time (if not the default is 5)
* if index has been created we cannot change the number of shards. we make a new index with the num of shards we want aand move the data to the new index

### Lecture 9 - Replication

* replication = fault tolerance & failover
* elasticsearch natively supports replication of the shards. shards are copied
* when a shard is replicated it is  referred as replica shard or just replica
* shards that have been replicated are referred as primary shards
* a primary shard and its replicas are referred as a replication group
* replication gives HA (high availability) in case nodes or shards fail
* replica shards are never allocated to the same node as primary shards
* if an entire node fails we will have at least one replica of any shard svailable.
* a side benefit of replication is increased performance for search queries. queries can be executed on all replicas n parallel. replicas are part of clusters search capabilities
* num of replicas is defined when creating an index (default is 1). by default a cluster with more than 1 node will have 5 primary shards and 5 replicas (10 shards per index)

### Lecture 10 - Keeping replicas synchronized

* how replicas are updated when data is changed or removed? replicas need to be kept in sync to avoid trouble of data incosistency between replicas
* to keep replicas in sync elasticsearch uses a model named primary-backup for data replication
* primary shard in a replication group acts as an enty pont for indexing operations. 	
* all operations that affect the index (ceate update remove  data) are sent to the primary shard
* primary shard then is responsible for validating operations and ensuring that all is good. (checking if request is structurally invalid e.g trying to add a number to an object field)
* when the operation is accepted by the primary shard it will be performed locally on the primary shard itself.
* when operation completes it will be forwarded to each of the replica shards in the group
* if the hard has multiple replicas the operation will be performed in parallel on each one
* when the operation has completed successfully on every replica and responded to the primary shard, the primary shard will respond to the client that the operation was successful.

### Lecture 11 - Searching for Data

* what we see as a dev running queries towards an elasticsearch cluster. we usually have a client app sending search queries over http. the cluster based on the index and query we specify in the http request does the search. when  results are ready the cluster responds with the results.
* most users see elasticsearch cluster as a black box on which they run queries. they dont care how elasticsearch gather the results internally.
* Suppose we have a 3 node cluster with 1 index distributed across 3 shards (A,B,C). each shard has 2 replicas. a replication group consists of one primary and 2 replicas
* a client sends a search request to a cluster which ends up on a node having primary shard B. this becomes the coordinating node for this query and sends queries to the other nodes, assembles results and sends them back to client
* any node in cluster may act as coordinating node and receive HTTP requests. as coordinating node contains a serchable shard it will perform the query itself. the cordinating node then broadcasts the request to every shard in the index primary and replicas. how the coordinating node decides which shards will get the request is out of our scope. when the other nodes reply. coordinating node merges the results and sorts them returning them to client. if we search by ID the request is routed to the apropriate shard and not broadcasted

### Lecture 12 - Distributing documents across shards

* we saw how data is stored on 1 or more nodes with sharding
* how elasticsearch knows on which shard to store a new document and how it finds it when retreived by id? docs should be distributed evenly across shards
* determining on which shard a doc will be or has been stored is called ROUTING.
* Routing is done  automatically. elasticsearch uses a simple routing formula. 
	* by default routing value is equal to a given documents ID 
	* the routing value is hashed to generate a num to be used in the division
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
* at least java8 is needed (we have it)
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

* we go to config folder on elasticsearch installation and open elasticsearch.yml file
* default options are great so we dont need to tweak anything but we will go through it for learning
* we can change the cluster name `cluster.name: <my-application>` by default the cluster will be named elasticsearch. use _dev and_prod differentiators in the name if needed
* we can specify the name of te node it will be started up on the machine `node.name: <node-1>` the installation he have setup is a single node. to start more nodes we have to repeat the installation process on another machine. what we get from this is per-node configuration. nodes might scale or replaced so we need proper node naming
* we can specify the network host and port to which nodes listen for request

```
# Set the bind address to a specific IP (IPv4 or IPv6):
network.host: 192.168.0.1
# Set a custom port for HTTP:
http.port: 9200
```

* we have discovery section with an option specifying the unicast hosts. these are the nodes the node we configure will try to contact. when starting the node needs to contact a node in a cluster or form a cluster of its own. the ips we specify there are the ones the node will try to contact when joining a cluster

```
# Pass an initial list of hosts to perform discovery when new node is started:
# The default list of hosts is ["127.0.0.1", "[::1]"]
#
#discovery.zen.ping.unicast.hosts: ["host1", "host2"]
#
# Prevent the "split brain" by configuring the majority of nodes (total number of master-eligible nodes / 2 + 1):
#
#discovery.zen.minimum_master_nodes: 
#
# For more information, consult the zen discovery module documentation.
```

* we dont have to specify all nodes there, this is unmaintanable. just one node is enought o retrieve the current cluster state. after retrieving the cluster state from a node it contacts the master node to join the cluster. dnt change minimum master nodes as it messes things up

* always restart the node after changing the configuration

### Lecture 18 - Installing Kibana on Mac/Linux

* [Kibana Download](https://www.elastic.co/downloads/kibana)
* if we are using docker, kibana is bundled on port 5601 at elasticsearch image
* we go to the link and download for our os (linux 64)
* we extract the tar (tar -zxf) and enter its dir
* to start kiban we need to run the kibana script in bin dir (./bin/kibana)
* we can hit its websearver in chrome at `http://localhost:5601`
* when we start kibana elasticsearch cluster must be running
* we stop kibana with Ctrl+C

### Lecture 20 - Configuring Kibana

* [Kibana Config Options](https://www.elastic.co/guide/en/kibana/current/settings.html)
* kibana config file is stored in config directory and is named kibana.yml
* the first option we might need to change is host and port of kibana server (defaults localhost:5601)

```
# Kibana is served by a back end server. This setting specifies the port to use.
#server.port: 5601

# Specifies the address to which the Kibana server will bind. IP addresses and host names are both valid values.
# The default is 'localhost', which usually means remote machines will not be able to connect.
# To allow connections from remote users, set this parameter to a non-loopback address.
#server.host: "localhost"
```

* we can change the serve name 

```
# The Kibana server's name.  This is used for display purposes.
#server.name: "your-hostname"
```

* we can change the elasticsearch url. kibana needs to communicate with an elasticsearch cluster

```
# The URL of the Elasticsearch instance to use for all your queries.
#elasticsearch.url: "http://localhost:9200"
``` 

* kibana creates an index in elasticsearch for storing data. we can customize its name there

```
# Kibana uses an index in Elasticsearch to store saved searches, visualizations and
# dashboards. Kibana creates a new index if the index doesn't already exist.
#kibana.index: ".kibana"
```

* other options include SSL, timeouts, logging
* restart kibana after changing config file so that changes take effect

### Lecture 21 - Kibana requires data to be available

* Kibana on recent versions requires data to be present in the cluster before being able to configurew an index pattern . We need to import data in teh cluster before working with kibana. (Lecture 32 & 33) . We will import data follwing LEcture 32 before going back to next lecture
* THIS IS SO WRONG. There is no need to do it. I dont know why the tutor says so

### Lecture 22 - Intro to Kibana and dev tools

* in the home page of kibana at localhost:5601 we are prometed to set up index patterns
* here we can enter a pattern for which indeices we want to include in kibana ui
* we insert * to incude all existing indices, kibana finds our avaialble indices (product from lecture 33)
* we uncheck time-based events (not relevant in our application) in advances options and click create
* our aim is to use the console in the dev tools to visualize our queries to the elasticsearch cluster. the API is the same HTTP RESTful and we can use also POSTMAN to insert our queries
* a default query is avaialble for us to test the console

* to send queries to the cluster we use HTTP requests. the request is composed by the http verb the request URI and request body. as the API is RESTful the verb is important (GET,POST,PUT,DELETE)
* in kibana devtools we start in console by writing the verb
* then we add the request URI (relative path excluding the http://host:port). the uri is structured /<index>/<type>/<api> so a rule is 

```
<HTTP verb> /<index>/<type>/<api>
```

* type is usually default (types are obsolete)
* api starts with _ (e.g _search _bulk)
* the console lets us cp the query to use it in cURL (wrench icon)
* if kibana runs on docker the command generated by the devtools console uses elasticsearch as hostname, because kibana runs in a different container in the virtual network, but if i run the command from my machine so outside the docker network the port 5601 is publixhed so i have to use localhost instead
* to add a request body (JSON in dev tools i just add my JSON script directly undter the URI in next line
* the generated cURL command uses -H flag to specify content-type json and -d flag to paste the JSON code of the body 

```
curl -XGET "http://localhost:9200/product/default/_search" -H 'Content-Type: application/json' -d'
{
  
}'
```

## Section 4 - Managing Documents

### Lecture 23 - Creating an Index

* we will create an index to add documents to it. it will be named products
* the command in kibana console is PUT /product?pretty` pretty is a flag to make the JSON pretty, this is relevant for other tools to structure the response in a human friendly way. so a general rule to create an index is PUT /<index>?pretty
* cluster replies 

```
{
  "acknowledged": true,
  "shards_acknowledged": true
}
```

* this is a simple command using the default replicas and shards

### Lecture 24 - Adding Documents

* the VERB for adding documents is `POST /<index>/<type>` in our case `POST /product/default`
* the request body must be a valid JSON object. any object is accepted

```
{"name": "Processing Events with Logstash",
  "instructor": {
    "firstname": "Bo",
    "lastname": "Andersen"
  }
}
```

* the reply from the cluster is

```
{
  "_index": "product",
  "_type": "default",
  "_id": "THgAy2IBN9aihGl6IhMT",
  "_version": 1,
  "result": "created",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 201,
  "_primary_term": 1
}
```
* an id is generated by elastisearch for the newly added document
* we can specify the id that we want when adding the document by chenging the URI to `PUT /<index>/<type>/<id>` see that it is not a POST but a PUT verb to comply with REST API. the request JSON body is the document object like before

```
# request
PUT /product/default/1
{"name": "Complete Guide to Elasticsearch",
  "instructor": {
    "firstname": "Bo",
    "lastname": "Andersen"
  }
}
# reply
{
  "_index": "product",
  "_type": "default",
  "_id": "1",
  "_version": 1,
  "result": "created",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 196,
  "_primary_term": 1
}
```

* if we add a document in a nonexisting index or type elasticsearch will create them using default values. if we want control over the config of the index. we must create it apriori

### Lecture 25 - Retieve Documents By ID

* we use GET and the URI is `GET /<index>/<type>/<id>`. if the doc is fount the reply contains "found: true. 
* document metafields added by elastic start with _ . the document itseldf is found in source:

```
# request
GET /product/default/1
# reply
{
  "_index": "product",
  "_type": "default",
  "_id": "1",
  "_version": 2,
  "found": true,
  "_source": {
    "name": "Complete Guide to Elasticsearch",
    "instructor": {
      "firstname": "Bo",
      "lastname": "Andersen"
    }
  }
}
```

### Lecture 26 - Replacing Documents

* replacing an existing document (if we now its ID) is very easy. is like adding a new document by id, say we want to add a price to the product#1

```
# request
PUT /product/default/1
{"name": "Complete Guide to Elasticsearch",
  "instructor": {
    "firstname": "Bo",
    "lastname": "Andersen"
  },
  "price": 10.99
}
# reply
{
  "_index": "product",
  "_type": "default",
  "_id": "1",
  "_version": 3,
  "result": "updated",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 203,
  "_primary_term": 1
}
```

* we see that result type is "updated" while when we create for first time is "created". if i now retrieve the document the _version metadata is 2 (it counts the number of times doc has changed)

### Lecture 27 - Updating Documents

* instead of replacing alll the doc we can patch it. we do it with a POST request to the update API using the doc URI `POST /<index>/<type>/<id>/_update` in the request JSON object we add the doc property with an object that contains the changes we want to apply on the doc

```
# request
POST /product/default/1/_update
{
"doc": {
    "price": 95,
    "tags": ["Elasticsearch"]
  }
}
# reply
{
  "_index": "product",
  "_type": "default",
  "_id": "1",
  "_version": 4,
  "result": "updated",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 204,
  "_primary_term": 1
}
```

* i verify the changes by retrieving the doc
* documents in elasticsearch are immutable, they cannot be changed once indexed. even if it looks the doc was partially updated this is not the case. internally th Update API retrieves,changes and re-indexes the document. it will retrieve the doc and replace the existing. all this is done in the shard

### Lecture 28 - Srcipted Updates

* scripting allows us to do multiple actions at once in a query.
* say I want to change the price of the course again. i can do so ,with the same URI but istead of using "doc" property I will use "script". the value of this property can contain various logic. but we will stick in updating the product price. we can access the document object on a variable called "ctx". this var contains the document fields we saw in teh query results like the _id and _source. so to access the price i type ctx._source.price and increase its val with += like in programming lang.

```
POST /product/default/1/_update
{
"script": "ctx._source.price += 10"
}
```

* if we retrieve th doc again we see the change 
* scripting can be very powerful and used to do many things.
* [scripting documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting.html)

### Lecture 29 - Upserts

* upsert refers to when we use the update api to update a doc but if it does not exist is created. we delete our test record  with `DELETE /product/default/1`
* what the script below does is it will increase the value if the doc exists . if the doc does not exist it will create a doc with the param passed in the "upser" object
```
# request
POST /product/default/1/_update
{
  "script": "ctx._source.price +=5",
  "upsert": {
    "price": 100
  }
}
# reply 1st run
{
  "_index": "product",
  "_type": "default",
  "_id": "1",
  "_version": 1,
  "result": "created",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 207,
  "_primary_term": 1
}
# reply 2nd run
{
  "_index": "product",
  "_type": "default",
  "_id": "1",
  "_version": 2,
  "result": "updated",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 208,
  "_primary_term": 1
}
``` 

### Lecture 30 - Deleting documents

* we saw delete queries in the previous lec. the sysntax is simple `DELETE /<index>/<type>/<id>`

```
# request
DELETE /product/default/1
# reply
{
  "_index": "product",
  "_type": "default",
  "_id": "1",
  "_version": 3,
  "result": "deleted",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 209,
  "_primary_term": 1
}
```

* if we want to delete multiple docs that meet certain conditions. updating multiple documents is not possible but deleting multiple documents is possible. we add 3 documents fo testing
* i want to delete with some criteria. we use an API named delete by query, which is avaialble at index level. the URI is `POST /<index>/_delete_by_query` followed by a JSON object with the query property. out test query will be a match query whoius value will  be an object with key balue pair specifing the query

```
# request
POST /product/_delete_by_query
{
"query": {
    "match": {
      "category": "book"
    }
  }
}
# reply
{
  "took": 46,
  "timed_out": false,
  "total": 1,
  "deleted": 1,
  "batches": 1,
  "version_conflicts": 0,
  "noops": 0,
  "retries": {
    "bulk": 0,
    "search": 0
  },
  "throttled_millis": 0,
  "requests_per_second": -1,
  "throttled_until_millis": 0,
  "failures": []
}
```

### Lecture 31 - Deleting Indices

* deleting an index is easy  `DELETE /<index>`

```
# request
DELETE /product
# reply
{
  "acknowledged": true
}
```

### Lecture 32 - Batch processing

* To import test data we will use batch processing, this means we will add many documents in a single request. each request has overhead in data and time
* Batch processing is done by using a special format for the request body using whats called the Bulk API.
* Bulk API is used to add, update or delete documents
* what we have to do is to send a POST request to /product/default/_bulk
* in Kibana (localhost:5601) we go to Dev Tools -> Console we enter

```
POST /product/default/_bulk
{"index": {"_id":"100"}}
{"price":100}
{"index": {"_id":"101"}}
{"price":101}
```

* the operation is  "index" to add a document to the index and for key we add a JSON object
* within the JSON object we add an "_id" key with the ID of the document we want to add and givie it a value of 100
* this was the action part. we need to add a document for the id of 100 we added. our document will contain only a price of 100. this is a JS literal
* we need to add multiple objects so that the BulkAPI makes sense.
* we cp the action and doc part of the query by increasing the value from 100 to 101
* this request will index 2 documents with IDs of 100 and 101
* we run the query and get the following response

```
{
  "took": 604,
  "errors": false,
  "items": [
    {
      "index": {
        "_index": "product",
        "_type": "default",
        "_id": "100",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 0,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "product",
        "_type": "default",
        "_id": "101",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 1,
        "_primary_term": 1,
        "status": 201
      }
    }
  ]
}
```

* the response is similar like running a single query (status per action).
* in BulkAPI it might happen that 1 action fails and the rest pass. ther error property is global though. si we check the error property and then search in the reply what failed (status)
* we will now see how to update and deletein bulk API
* our query updates the doc with id 100 price to 1000 and deletes the second document with id 101

```
POST /product/default/_bulk
{"update": {"_id":"100"}}
{"doc": {"price": 1000}}
{"delete": {"_id":"101"}}
```
* the reply is 

```
{
  "took": 31,
  "errors": false,
  "items": [
    {
      "update": {
        "_index": "product",
        "_type": "default",
        "_id": "100",
        "_version": 2,
        "result": "updated",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 2,
        "_primary_term": 1,
        "status": 200
      }
    },
    {
      "delete": {
        "_index": "product",
        "_type": "default",
        "_id": "101",
        "_version": 2,
        "result": "deleted",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 3,
        "_primary_term": 1,
        "status": 200
      }
    }
  ]
}
```

* to verify price we retieve the first document with 

```
GET /product/default/100
```

* the reply is OK, if we search for 101 the result is not found

```
{
  "_index": "product",
  "_type": "default",
  "_id": "100",
  "_version": 2,
  "found": true,
  "_source": {
    "price": 1000
  }
}
```

* we delete the 100 as well to prepare for next lecture

### Lecture 33 - Importing Test  Data and cURL

* we have a test JSON document consisting of 1000 docs representing foods and drinks products with price. 
* we can cp the contents of the file in kibanas dev tools console but we will try cURL
* we go tto the dir where the test file resides
* we will read the data from the file and send its contents to elasticsearch without any processing
* the complete command is `curl -H "Content-Type: application/json" -XPOST "http://localhost:9200/product/default/_bulk?pretty" --data-binary "@test-data.json"`
* the above dommand parsed data from a file and uses post command setting the header content-type param

* the import is successful
* [cURL download](https://curl.haxx.se/download.html)

### Lecture 34 - Exploring the Cluster

* [Cluster API Documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster.html)
* [node commands documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/cat-nodes.html)
* the cluster API replies verbose JSON with lots of detail
* we get the clusters health with `GET /_cat/health?v` v is the verbose flag

```
GET /_cat/health?v
# reply
epoch      timestamp cluster       status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
1523906221 22:17:01  elasticsearch yellow          1         1      6   6    0    0        5             0                  -                 54.5%
```

* the result shows we have a single node cluster and 6 primary shards (5 for the index we created and 1 for .kibana index) and 6 replica shards as default replicas is 1. replicas are unassigned because we have a single node. priamry and replica shards are never in same node. that is the reson the cluster status is yellow (risk of data loss). status red is BAD. it means that some of the data are unavailable (some node is down and we dont have enough replicas)
* to get info about the node i use `GET /_cat/nodes?v`

```
GET /_cat/nodes?v
# reply
ip        heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
127.0.0.1           35          95   1    0.33    0.50     0.53 mdi       *      MtGZ0aF
```

* we se the load and name and role of our node as well its ip
* in the cluster api we can specify what column we want returned
* we now want info on indices `GET /_cat/indices?v`

```
GET /_cat/indices?v
# reply
health status index   uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   product R5K7ozuLQEmRSu0O_UoRhQ   5   1       1000            0    503.8kb        503.8kb
green  open   .kibana x1uJQoGjTwyyRkjs5YbDJQ   1   0          2            1     13.1kb         13.1kb
```
* info we get is health, storage usednum of docs
* to see shards allocation we use `GET /_cat/allocation?v` with info on disk usage
* for shards `GET /_cat/shards?v` for doc distribution etc

## Section 5

### Lecture 35 - Introduction to Mapping

* mappings define how documents and theirs fields should be stored and indexed
* the reson for that is to store and index data in a way that is appropriate for how we want to search our data
* a couple of examples of what mappings can be used is to define which fields should be treated as full text fields, which fields contain numbers, dates or geo locations
* we can also specify date formats for date fields or specify analyzers for full text fields
* mappings in elasticsearch is like a schema for tables in relational dbs
* for simple use cases mappings are not needed, but for great control over how elasticsearch handles our data we need to know about mappings

### Lecture 36 - Dynamic Mapping

* [config date detection](https://www.elastic.co/guide/en/elasticsearch/reference/current/dynamic-field-mapping.html#date-detection)
* mapping can be defined explicitly when we tell elasticsearch what to do with our data when we add new documents or update new ones. docs are immutable so to update them we replace existing ones
* Dynamic mapping is an alternative way to explicitly define document field data types
* It is used by elasticsearch when we add new docs. it will add default data types for fileds that dont have mappings, by inspecting the types of values for a docs fields
* if a field in a doc we add has a value of formatted data elasticsearch will add the "date" data type. we can add rules and defaults for dynamic mapping
* to see the mappings elastic has added with default mapping for the docs we have added we use the mapping API `GET /<index>/<type>/_mapping`

```
GET /product/default/_mapping
# reply
 {
  "product": {
    "mappings": {
      "default": {
        "properties": {
          "created": {
            "type": "date",
            "format": "yyyy/MM/dd HH:mm:ss||yyyy/MM/dd||epoch_millis"
          },
          "description": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "in_stock": {
            "type": "long"
          },
          "is_active": {
            "type": "boolean"
          },
          "name": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "price": {
            "type": "long"
          },
          "sold": {
            "type": "long"
          },
          "tags": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          }
        }
      }
    }
  }
}
```

* we see that it does date detection based on formats. we can configure or disable date detection (see link)
* for ints elastic uses long type as it does not num size we intend to use
* for text fields apart from text type which is norla elastic adds a fields property of type keyword. text is used for full type searches and keyword for exact matches, agregations eta ll

### Lecture 37 - Meta Fields

* we saw that docs stored in elastic have meta-data associated with them. these fields are called meta-fields. there are 10 meta fields
* _index contains the name of the index to which the document belongs. is added by elastic autom and is used by elastic intrnally when querying docs in index. but can explicitly used in search queries to search for docs in multiple indexes
 * _id stored the id of the doc. we dont query it explicitly but is used in queries when we query by id
* _source contains the original JSON object that was passed in elastic when doc was indexed. we cannot  search it but we can retrieve it.
* _field_names contains the names of every field that contains non-null val. this is used with query named exists. which matches docs that contain a non-null val for a given field
* _routing sotres the value used to route a doc into a shard
* _version stores the internall version of the doc. it increases on any update
* _meta may be used to store custom data that is left untouched by elastic e.g app specific data

 ### Lecture 38 - Field Data Types

* 4 categories of field data types: Core Data Types, Complex Data Types, Geo Data Types, Specialized Data Types
* Core Data Types:
  * Text Data Type (text): used to index full-text value such as desc. Values are analyzed. Full-text fields are rarely used for sorting and aggregating documens due to their nature. for this keyword are used. they are sotred in a way optimal ofr full-text searches
  * Keyword Data Type (keyword): Used for structured data (tags, categories, email-addr...) Not analyzed. Typically used for filtering and aggregations. they are stored as is
  * Numeric Data Types: basic numeric data types, most of which are found in various programming langs. (... byte ... scaled_float, half_float ...) byte(-127,128) scalled _float is a float storeda s a long in elastic (possible through the use of scaling factor). integers are easier to compress. long is trasformed back to float at index time
  * Date data type (date): represents dates as either as tring, long or it. the date format may be configured. default type is string or timestamp in ms. internally dates are stored as long (timestamp) in ms since epoch
  * Boolean data Type (boolean): sores boolean vals
  * Binary Data Type (binary): acceptsa Base65 encoded binary val. not stored by default. so we cannot search it or retrieve it outside of _source. this can be configured with store option
  * Range Data Types: used for range  values such as date ranges or numeric ranges (e.g 10-20) integer_range, float_range, long_range, double_range, date_range. we define alower and upper boundary when indexing the doc. using "gt, "gte","lt","lte" (e.g. {"gte": 10, "lte": 20})
* Complex Data Types: these datatypes are not primitive, but more complex (arrays , objects)
  * Object Data Type: added as JSON objects. stored as key-value pairs internally where structure gets flattened out but key naming keeps hierarchy. may contain nested objects

  ```
  # index time
  {
    "name": {
    "firstname": "Bo",
    "lastname": "Andersen"
    },
    "profession": "Software Engineer"
  }
  # Stored
  {
    "name.firstname": "Bo",
    "name.lastname": "Andersen",
    "profession": "Software ENgineer"
  }
  ```
  * Array Data Type: not an actual data type, because each field may contain multiple values by default. any field in a doc can contain 0:n values by deafult like an array of nums, strngs, objects etc. this holds without us having to explicitly declare it. we can have nested arrays. but they are flattened when indexed. in arrays all vals have to be of same datatype. if we dont expricitly se type. the type of first element will be used to set the data type. if we storearray of objects we cannot query them individualy. objecs get flattened and Apache Lucene has no concept of inner objects. association between obkject vals is lost.

  ```
  {
    "persons": [
      {"name": "Bo Andersen", "age":28},
      {"name": "Mika Hakinen", "age":20}
    ]
  }
  # becomes
  {
    "persons.name": ["Bo Andersen","Mika Hakinen"],
    "persons.age": [20,28],
  }
  ```

  * Nested Data Type (nested): solves the problem of object arrays and querying them individualy. is a specialized version of the object data type. enables arrays of objects to be qualified independently of each other. object are stored as hifdden docs. we need to use "nested" queries to use it
* Geo Data Types: data types that are used for geographical data (e.g lat long pair)
  * Geo-point Data Type (geo_point): accepts latitude longitude paires. used for various geographical operations. it accepts 4 formats for coordinates
  * Geo-Shape Data Type (geo_shape): used for more geographical shapes like polygons, circles etc. (point, linestring,multipoint, multilinestring, multipolygon, geometrycollection, envelope). used to stoe shapeof geo datas, borders. shapes are represented with GeoJSON
* Specialized Data Types: data types with very specific purpose (e.g storing IP addr)
  * IP Data Type (ip): Used for storing IPv4 and IPv6 addresses, query them with CIDR notation
  * Completion Data Type (completion): used to provide auto-completion ("search as you type") functionality. opimized for quick lookups. using suggesters. is very fast
  * Attachment Data Type (attachment): requires the Ingest Attachment Processor Plugin. Used to make text from various document formats searchable (e.g PPT, PDF, RTF). Uses Apache Tika internally for text recognition `sudo bin/elasticsearch-plugin install ingest-attachment`. Tika can be used at application level (get full text search it outside elastic)

### Lecture 39 - Adding Mappings to existing Indices

* we have in our cluster an index *products* and some documents indexed in it. 
* we will add a new field in the mapping (aka schema). a discount field of double data type
* the syntax is `PUT /<index>/<type>/_mapping` and the JSON req body contains a "properties" JSON obect with our fields to be added and their types declared as a mapping obejct containing the type

```
PUT /product/default/_mapping
 {
  "properties": {
    "discount": {
      "type": "double"
    }
  }
}
# reply
{
  "acknowledged": true
}
```

* to verify we inspect the mappings like we saw before. 
* now docs with discount field can be added

### Lecture 40 - Changing Existing Mappings

* if we try changin existing mappings by using the previous lectures query to change the type from double to integer we get an error (illegal argument exception)
* existing field mappings cannot be updated
* we must delete the index, create a new mappings and reindex the data into the new index
* if we could update mappings we would invalidate any docs already added to the index
* we wipe out the index `DELETE /<index>`
* we recreate the index AND its mapping explicitlyin one query 

```
PUT /product
{
"mappings": {
    "default": {
      "dynamic": false,
      "properties": {
        "in_stock": {
          "type": "integer"
        },
        "is_active": {
          "type": "boolean"
        },
        "price": {
          "type": "integer"
        },
        "sold": {
          "type": "long"
        }
      }
    }
  }
}
# reply
{
  "acknowledged": true,
  "shards_acknowledged": true,
  "index": "product"
}
``` 

* at this point we can add the test data but only the simple ones will pass ans we are missing field mappings for the compolex ones. 
* we will complete mappings before indexing the test data
* the EXCEPTIONS to the NO MAPPING UPDATE rule are objects where we can addproperties and text fields whjere we can add the keyword type

### Lecture 41 - Mapping Parameters

* [Mapping Parameters](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-params.html)
* [Custom Date Formats](http://www.joda.org/joda-time/apidocs/org/joda/time/format/DateTimeFormat.html)
* [Biolt In Date Formats](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-date-format.html#built-in-date-formats)
* we will look at parameters used to configure field mappings
* coerce: can be used to disable coercion (automatically cleaning up values, or trasformation to the correct type if data coming are another type). the result of disabling coerce is that elastic will reject docs with fields that dont match the type
copy_to: eanblaes us to create custom field from other fields. copies field values into a given field. it copie values not terms that are output fromt he analyzer

```
{
  "first_name": {
    "type": "text",
    "copy_to": "full_name"
  },
  "last_name": {
    "type": "text",
    "copy_to": "full_name"
  },
  "full_name": {
    "type": "text"
  }
}
```

* copied values will not show up in the _source meta field
* dynamic: enables or disables adding fields to documents or inner objects dynamically. it eanbles/disables dynamic mapping for new fields as you ve seen. by default elastic uses dynamic mapping for new fields. we can disable enable selectively in document or inner objects. disabling dynamic mapping forbids adding new fields if they are mapped before. if it sets false it ignores it , if it set strict it rejects the doument and does not index it.
* properties: this param contains the field mappings, aither at the top level of documents or with inner objects. it is used as a wrapper at any level to frap field mappings
* it is also used when defining fields in an object or nested field.
* norms: it defines whether or not to disable storing norms (used for relevance scores).this is because elastic doens just sees if document matches but it works out how well a document matches. so it stores some info to calculate relevance scores. we can disable/enable that with the norm param. for fields  that are used for aggregations or filtering this might be acceptble, we cannot reenable norms without re recreating the index

```
{
  "properties": {
    "full_name": {
      "type": "text",
      "norms": false
    }
  }
}
```

* format: defines the format for date fields `yyyy-MM-dd` 'epoch_millis' `epoch _second` ... custom formats are accepted.  the default format is *strict_date_optional_time || epoch_millis*
* null_value: replace NULL values with the specified value

```
{
  "properties": {
    "discount": {
      "type": "integer",
      "null_value": )
    }
  }
}
```

* fields: it is used to index fields in different ways

### Lecture 42 - Adding Multi-Field Mappings

* we will use the *fields* param to add more keyword mappings to text fields and also the *properties* param. we, ve seen this multi mapping struct when we insert unmapped data and elastic replies with the dynamic mapping map.

```
PUT product/default/_mapping
{
  "properties": {
    "description": {
      "type": "text"
    },
    "name": {
      "type": "text",
      "fields": {
        "keyword": {
          "type": "keyword"
        }
      }
    },
    "tags": {
      "type": "text",
      "fields": {
        "keyword": {
          "type": "keyword"
        }
      }
    }
  }
}
# reply 
{
  "acknowledged": true
}
```

* we retrieve the complete mapping to check status

```
GET /product/default/_mapping
# reply
{
  "product": {
    "mappings": {
      "default": {
        "dynamic": "false",
        "properties": {
          "description": {
            "type": "text"
          },
          "in_stock": {
            "type": "integer"
          },
          "is_active": {
            "type": "boolean"
          },
          "name": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword"
              }
            }
          },
          "price": {
            "type": "integer"
          },
          "sold": {
            "type": "long"
          },
          "tags": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword"
              }
            }
          }
        }
      }
    }
  }
}
```

### Lecture 44 - Defining Custom Date Formats

* [built-in date formats](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-date-format.html#built-in-date-formats)
* [custom date formats](http://www.joda.org/joda-time/apidocs/org/joda/time/format/DateTimeFormat.html)
* we will now add a mapping for the created field which is a date.

```
PUT product/default/_mapping
{
  "properties": {
    "created": {
      "type": "date",
      "format": "strict_date_optional_time||epoch_milis"
    }
  }
}
```

* we spec it a a date type and pass the accepted date format. 
* strict date means a complete date with leading zeros.
* optional time means time is not required.
* || epoch_millis means an alternative timestamp format is accepted. millis epoch is UNIX timnestamp * 1000. no slashes allowed.
* this format is the implicit one if we do dynamic mapping. muchmore formats available
* always check the test data for format. change the data or change the format to parse them. our date data have / . we define a custom format `"format": "yyyy/MM/dd HH:mm:ss||yyyy/MM/dd"`
* we pass the query to mapping API with the custom format and retrieve mapping to verify
* we are ready to pass in the test data and use all their fields with our now complete map. we use the curl command like before

```
curl -H "Content-Type: application/json" -XPOST "http://localhost:9200/product/default/_bulk?pretty" --data-binary "@test-data.json"
```
* we successfully populate our index with docs

### Lecture 45 - Picking up new fields without dynamic mapping

* we added all fields except from discount field. we want to add this in mapping
* before that we will try to index a doc with a description and discount(not yet mapped). it is indexed successfully

```
POST /product/default/2000
{
  "description": "Test",
  "discount": 20
}
# reply
{
  "_index": "product",
  "_type": "default",
  "_id": "2000",
  "_version": 1,
  "result": "created",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 196,
  "_primary_term": 1
}
```

* next we add the discount proerty to the mapping. its added OK

```
PUT /product/default/_mapping
{
  "properties": {
    "discount": {
      "type": "integer"
    }
  }
}
# reply
{
  "acknowledged": true
}
```

* we search for the test doc to see it is indexed correctly using the _search API. SUCCESS

```
GET /product/default/_search
{
  "query": {
    "match": {
      "description": "Test"
    }
  }
}
# reply
{
  "took": 36,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 8.108471,
    "hits": [
      {
        "_index": "product",
        "_type": "default",
        "_id": "2000",
        "_score": 8.108471,
        "_source": {
          "description": "Test",
          "discount": 20
        }
      }
    ]
  }
}
```

* we will now try to search the test doc using the discount field in our query. It fails to return the doc. this si because we disabled dynamic mapping. so it ignored fields for whose there is no mappings. test doc therefore was store without the discount param then not yet mapped. the values exist under the _source meta field but they were not indexed/searchable

```
GET /product/default/_search
{
  "query": {
    "term": {
      "discount": 20
    }
  }
}
# reply
{
  "took": 8,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 0,
    "max_score": null,
    "hits": []
  }
}
```

* one solution is to start over and add mapping before adding the doc. or use an API named *update by query*. this API updates documents, without changing the source. it touches the documens and reindexes them. this is useful to pick up new mappings

```
POST /product/_update_by_query?conflicts=proceed
# reply
{
  "took": 279,
  "timed_out": false,
  "total": 1001,
  "updated": 1001,
  "deleted": 0,
  "batches": 2,
  "version_conflicts": 0,
  "noops": 0,
  "retries": {
    "bulk": 0,
    "search": 0
  },
  "throttled_millis": 0,
  "requests_per_second": -1,
  "throttled_until_millis": 0,
  "failures": []
}
```

* we set conflict=proceed because we dont care. conflict is when a doc is added while we run the query. but if it is changed it will pick up the map so no problem, we ignore it. this query updates ALL doc in the index. 
* if we rerun the query with the discount we get the doc back. SUCCESS
* we delete the test doc `DELETE /product/default/2000`

## Section 6 - Analysis & Analyzers

### Lecture 47 - Intro to the analysis process

* what it means that a text is analyzed? it means that when indexing a document its full text fields are run through an analysis procees. full text fields are fields of type "text". analyzing means tokenizing the text in terms, locating text etc
* the analysis process means normaizing and tokenizing text to make it easier to search
* we have full control over which analyzer to use. standard analyzer is enough for most cases. results of analysis is what is stored in the inverted index. when we search through text we search through the results of the analysis process (inverted index) not in the actual text

### Lecture 48 - A closer look at analyzers

* the conversion we talked before is done by the nalyzer. the analyzer consists of 3 things: character filters, token filters, tokenizer
* the analyzer is a package of these 3 building blocks each one changing the input stream
* when being indexed the document goes through the following flow:
  * serial or more character filters can be added. a character filter receives the original text and can transform the value by adding removing or changing characters e.g <strong>Two</strong>words! => Two words!
  * after the tokenizer splits the text into individual tokens which are usually words. e.g for a sentence of 10 words we would get an array of 10 words. Two words! => [Two,words]. an analyzer may have only one tokenizer. by default a tokenizer name standrd is used. this uses a unicode text segmentation algorithm, it splits by whitespace and removes special symbols. tokenizers store the psosition of tokes. start position and offset of the words they represent. this matching is useful when highlighting the found words. also are used for fuzzy phrase searches and  proximity searches
  * then is passes from serial or more token filters. a token filter may add remove or change tokens. like character filters but working on token stream. many of them exist. simplest one is lowercase filter [Two,words] => [two,words]. another filter removes stub words. little sgnificance in terms of relevance. other filter is synonym. gives similar words same meaning semantically
  * in elastic search if no special analyzer is specified,. the default is the standard analyzer.
  with the standard analyzer there is no character filter. text goes straight to tokenizer. the tokenizer used is one named standard (filters out symbols, split by whitespace)  I'm in the mood for drinking semi-dry red wine! => [I'm, in, the, mood, for, drinking, semi, red, wine]. this array of tokens is sent ot a chain of token filters. first one is standard. it does nothing. just a placeholder for future filters, stub token filter is added but disabled. the only token filter used wihth thes default standard analyzer is the lowercase
  * there s an analyze API to test the results of applying character filters, tokenizers, and token filters as a whole

### Lecture 49 - Using the Analyze API

* we use the analyzer API on the test sentence of the prev lecture. er use the POST _analyze URI pasing a JSON object whre we spec the tokenizer and pass in the text to the analyzer

```
POST _analyze
  {
    "tokenizer": "standard",
    "text": "I'm in the mood for drinking semi-dry red wine"
  }
# reply
{
  "tokens": [
    {
      "token": "I'm",
      "start_offset": 0,
      "end_offset": 3,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "in",
      "start_offset": 4,
      "end_offset": 6,
      "type": "<ALPHANUM>",
      "position": 1
    },
...
    {
      "token": "red",
      "start_offset": 38,
      "end_offset": 41,
      "type": "<ALPHANUM>",
      "position": 8
    },
    {
      "token": "wine",
      "start_offset": 42,
      "end_offset": 46,
      "type": "<ALPHANUM>",
      "position": 9
    }
  ]
}
```

* now we will apply a tokefilter (lowercase) to the tokens generated. token filters come as an array

``` 
POST _analyze
{
  "filter": ["lowercase"],
  "text": "I'm in the mood for drinking semi-dry red wine"
}
# reply
{
  "tokens": [
    {
      "token": "I'm",
      "start_offset": 0,
      "end_offset": 3,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "in",
      "start_offset": 4,
      "end_offset": 6,
      "type": "<ALPHANUM>",
      "position": 1
    },
...
    {
      "token": "red",
      "start_offset": 38,
      "end_offset": 41,
      "type": "<ALPHANUM>",
      "position": 8
    },
    {
      "token": "wine",
      "start_offset": 42,
      "end_offset": 46,
      "type": "<ALPHANUM>",
      "position": 9
    }
  ]
}
```

* we see that even when we dont define tokenizer the text was tokenized before the filter was applied. this is because standard tokenizer comes as default. if we want to add a character filter in the JSON obejct we pass "char_filter": []
* instead of configuring the discrete steps of the analyzer i can pass a predeifined analyzer ans an object param

```
POST _analyze
{
  "analyzer": "standard",
  "text": "I'm in the mood for drinking semi-dry red wine"
}
# result is the same
```

### Lecture 50 - Understanding the Inverted Index

* now that we have seen how the results of the analysis are produced we will see what happens to the results after being created
* the results are stored to what is called inverrted index. it purpose is to to store text in a way that allows very fast and efficient full text searches
* full text queries are performed on the inverted index and on the original text
* a cluster will have at least one inverted index. there is one inverted index foer each full text field per index
* an inverted index consists of all the unique terms that appear in any indexed document for the same full text field
* an inverted index is a map between terms and which docs contain them.
* terms in inverted index are sorted
* the search query is split in terms which are mapped to the documents using the inverted index map to find the best match
* the inverted index contains meta data for internal usage lie num of documents containg each term
* we can apply stemming and synonims in inv index

### Lecture 51 - Overview of Character Filters

* we will see the available character filters for us to use in our analyzers and what they do. 
* three built-in character filters available
  * HTML Strip Character Filter "filter_strip". strips out HTML elements *<strong>* and decodes HTML entities *&amp;*
  * Mapping Character Filter "mapping": Replaces values bassed on a map of keys and values I broke my leg _sad_ but who cares _happy_ => I broke my leg :-( but who cares :-)
  * Pattern Replace "pattern_replace": Uses a regular expression to match characters and replaces them with the specified replacement. Capture groups may be used. Pattern: ([a-zA-Z0-9]+)(-?) Replacement: $1 "9x4343-5s-jf4" => "9x43435sjf4"

### Lecture 52 - Overview of Tokenizers

* we will see the tokenizers tha elasticsearch provides by default
* 3 main groups: 
  * word oriented tokenizers: typically used for tokenizing full text into individual words
    * Standard Tokenizer "standard": divides text into terms on word boundaries and removes most symbols. Usually the best choice
    * Letter Tokenizer "letter": divides text into terms when encountering a character thyat is not a letter. "I'm in funky-buddha" => [I,m,in,funky,buddha]
    * Lowercase Tokenizer "lowercase": works like the *letter* tokenizer but also lowercases all terms generated
    * Whitespace tokenizer "whitespace": divides text into terms when encountering whitespace chars "Funky-budda in Bob's house" => [Funky-buddha,in,Bob's,house!]
    * UAX URL Email Tokenizer "uax_url_email": like the *standard* tokenizer, but treats URLs and email addresses as single tokens "http://localhost as@as.com" => [http://localhost,as@as.com]
  * Partial word tokenizers: they break up text or words into small fragments. Used for partial word matching
    * N-Gram Tokenizer "ngram": breaks text into words when encountering certain characters and then emits N-grams of the specified length "Red Wine" => [Re,Red,ed,wi,win,wine,in,ine,ne]. N--gram is like a sliding window that moves accross a word. the N-gram is configurable. here min length is 2 and max 10
    * Edge N-Gram Tokenizer "edge_ngram": Breaks text into words when encountering certain characters and then emits N-grams of each word beginiing from the start of the word "Red Wine" => [Re,Red,wi,win,wine]. like previous but N-grams always start from first char. Used in autocompletion like Google. not as good as suggestors
  * Structured Text tokenizers: used for structured text like email addresses, zip codes, identifiers
    * Keyword Tokenizer "keyword": no-op tokenizer which outputs the exact same text as a single term "I'm in the mood for love" => [I'm in the mood for love]. doesnt do anything. Only useful when we dont want to tokenize (elastic requires one tokenizer for full text fields). or when we want just o apply filters
    * Pattern Tokenizer "pattern": uses a regular expression to split text into terms when matching a word separator. Alternatively captures matched text as terms "I,like,red,wine!" => [I,like,red,wine!]. commas are uses to separate terms
    * Path Tokenizer "path_hierarchy": Splits hierarchical values (e.g file system paths) and emits a term for each component of the tree /path/to/dir => [/path,/path/to,/path/to/dir]. dlimiter configurable but defaults is slash

### Lecture 53 - Overview of Token Filters

* [full-list of available token filters](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-tokenfilters.html)
* many avaialble as buil-in . here we mention a few. for the xamples we assume standard tokenizer is applied
  * Standard Token Filter "standard": doesn't do anything. acts as a placeholder for future versions [I'm,in,the,mood] => [I'm,in,the,mood]
  * Lowercase Token Filter "lowercase": normalizes terms to lowercase. [I'm,in,Paris] => [i'm,in,paris]
  * Uppercase Token Filter "uppercase": normalizes terms to Uppercase [I'm,in,Paris] => [I'M,IN,PARIS]
  * NGram Token Filter "nGram": Emits N-Grams of the specified length based on the provided terms [Red,wine] => [R,Re,e,ed,d,w,wi,i,in,ne,e] default min=1 max=2 ngram
  * Edge NGram Token Filter "edgeNGram": emits N-grams of each term beginning from the start of the term [Red,wine] => [R,Re,w,wi]
  * Stop Token Filter "stop": removes stop words [I'm,in,the,forest] => [I'm,forest]
  * Word Delimiter Token Filter "word_delimiter": splits words into subwords and performs transformations on subword groups [Wi-Fi,PowerShell,Andy's] => [Wi,Fi,Power,Cell,Andy] uses a number of rules enabled and disabled at please
  * Stemmer Token Filter "stemmer": stems words for the specified language [I'm,in ,the,mood,for,drinking] => [I'm,in ,the,mood,for,drink] stems workd to their base form
  * Keyword Marker Token Filter "keyword_marker": protects words from being modified by stemmers. Protected terms: [drinking] [I'm,in ,the,mood,for,drinking] => [I'm,in ,the,mood,for,drinking]
  * Snowball Token Filter "snowball": stems words based on the snowball algorithm, [I'm,in ,the,mood,for,drinking] => [I'm,in ,the,mood,for,drink]
  * Synonym Token Filter "synonym": Adds or replaces tokens based on a  a synonym configuration file [I,am,very,happy] => [I,am,very,happy/delighted]. we can supply a config file for synoniums. synonyms get injected to the terms
* Other token filters: Trim,Length. Truncate

### Lecture 54 - Overview of Built-in Analyzers

* [Language Analyzers](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-lang-analyzer.html) 
* [Built-in Analyzers](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-analyzers.html)
* we will have a look at the default analyzers. they are the default orchestrators of the parts we ve seen so far.
* Standard Analyzer "standard": Divides text into terms using Unicode Text Segmentation (e.g word boundaries), Removes punctuation, lowercases terms, and optioanlly removes stop words "I'm inthe moof for funky-buddha!" => [i'n,in,the,mood,for,funky,buddha]
* Simple Analyzer "simple": Divides text into terms when encountering a character that is not a letter. Also lowercases all terms "I'm in the mood for Post-Rock" => [i,m,in,the,mood,for,post,rock]
* Stop Analyzer "stop": like the *simple* analyzer but also removes stop words "I'm in the mood for Post-Rock!" => [i,m,mood,post,rock]
* Language Analyzers "(english, ...)": language-specific analyzers, e.g for English or Spanish. "I'm in the mood for drinking!" => [i'm,mood,drink] easy to enable stemming, synonyms etc without explicitly defining them. here it has stop token filter, stemmer token filter
* Keyword Analyzer "keyword": no-op analyzer that returns the input as a single term "I'm in the mood fro drinking!" => [I'm in the mood fro drinking!]
* Pattern Analyzer "pattern": uses a regular expression to match token separators and splits text into terms where matches occur "I,like,red,wine!" => [I,like,red,wine!] loweracase token filtern and pattern tokenizer an optional stop token filter
* Whitespace Analyzer "whitespace": Breaks text into terms when encountering a whitespace character "I'm in the high-mood for drinking!" => [I'm,in,the,high-mood,for,drinking!] wrapper of whitespace tokenizer

### Lecture 55 - Configuring Built-In Analyzers and Token Filters

* some analyzers can be configured through parameters
* we create a new index for testing and configure it in the same time. in its settings we add a custom analyzer and custom token by extending existing ones. we can do the same for tokenizers and char filters
```
PUT /existing_analyzer_config
{
  "settings": {
    "analysis": {
      "analyzer": {
        "english_stop" : {
          "type": "standard",
          "stopwords": "_english_"
        }
      },
      "filter": {
        "my_stemmer": {
          "type": "stemmer",
          "name": "english"
        }
      }
    }
  }
}
# reply
{
  "acknowledged": true,
  "shards_acknowledged": true,
  "index": "existing_analyzer_config"
}
```

* we use the newly created custom analyzers and token filters using the _analyze API
on a test string

```
POST /existing_analyzer_config/_analyze
{
  "analyzer": "english_stop",
  "text": "I'm in the mood for drinking semi-dry red wine!"
}
# reply
{
  "tokens": [
    {
      "token": "i'm",
      "start_offset": 0,
      "end_offset": 3,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "mood",
      "start_offset": 11,
      "end_offset": 15,
      "type": "<ALPHANUM>",
      "position": 3
    },
    {
      "token": "drinking",
      "start_offset": 20,
      "end_offset": 28,
      "type": "<ALPHANUM>",
      "position": 5
    },
    {
      "token": "semi",
      "start_offset": 29,
      "end_offset": 33,
      "type": "<ALPHANUM>",
      "position": 6
    },
    {
      "token": "dry",
      "start_offset": 34,
      "end_offset": 37,
      "type": "<ALPHANUM>",
      "position": 7
    },
    {
      "token": "red",
      "start_offset": 38,
      "end_offset": 41,
      "type": "<ALPHANUM>",
      "position": 8
    },
    {
      "token": "wine",
      "start_offset": 42,
      "end_offset": 46,
      "type": "<ALPHANUM>",
      "position": 9
    }
  ]
}
```

* or we use the xustom token filter instead (with the standard tokenizer)

```
POST /existing_analyzer_config/_analyze
{
  "tokenizer": "standard",
  "filter": ["my_stemmer"],
  "text": "I'm in the mood for drinking semi-dry red wine!"
}
# reply
{
  "tokens": [
    {
      "token": "I'm",
      "start_offset": 0,
      "end_offset": 3,
      "type": "<ALPHANUM>",
      "position": 0
    },
...
    {
      "token": "drink",
      "start_offset": 20,
      "end_offset": 28,
      "type": "<ALPHANUM>",
      "position": 5
    },
    {
      "token": "semi",
      "start_offset": 29,
      "end_offset": 33,
      "type": "<ALPHANUM>",
      "position": 6
    },
    {
      "token": "dry",
      "start_offset": 34,
      "end_offset": 37,
      "type": "<ALPHANUM>",
      "position": 7
    },
    {
      "token": "red",
      "start_offset": 38,
      "end_offset": 41,
      "type": "<ALPHANUM>",
      "position": 8
    },
    {
      "token": "wine",
      "start_offset": 42,
      "end_offset": 46,
      "type": "<ALPHANUM>",
      "position": 9
    }
  ]
}
```

### Lecture 56 - Creating Custom Analyzers

* we ve seen how to extend default analyzers and their steps and apply them using the _analyze API. we will now learn how to create them from scratch
* we use the settings object from previous lecture with the two custom extended analyzer.filter on a new test index, while adding our new custom anal;yzer "my_analyzer" from scratch after the "english_stop" we defined before
* the type is "custom" and add its parts composed by default parts.

```
PUT /analyzers_test
{
  "settings": {
    "analysis": {
      "analyzer": {
        "english_stop" : {
          "type": "standard",
          "stopwords": "_english_"
        },
        "my_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "char_filter": [
            "html_strip"
          ],
          "filter": [
            "standard",
            "lowercase",
            "trim",
            "my_stemmer"
          ]
        }
      },
      "filter": {
        "my_stemmer": {
          "type": "stemmer",
          "name": "english"
        }
      }
    }
  }
}
# reply
{
  "acknowledged": true,
  "shards_acknowledged": true,
  "index": "analyzers_test"
}
```

* we test our newly created analyzer like before on a test string using the _analyze API

```
POST /analyzers_test/_analyze
{
  "analyzer": "my_analyzer",
  "text": "I'm in the mood for drinking <span>semi-dry</span> red wine!"
}
#reply
{
  "tokens": [
    {
      "token": "i'm",
      "start_offset": 0,
      "end_offset": 3,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "in",
      "start_offset": 4,
      "end_offset": 6,
      "type": "<ALPHANUM>",
      "position": 1
    },
    {
      "token": "the",
      "start_offset": 7,
      "end_offset": 10,
      "type": "<ALPHANUM>",
      "position": 2
    },
    {
      "token": "mood",
      "start_offset": 11,
      "end_offset": 15,
      "type": "<ALPHANUM>",
      "position": 3
    },
    {
      "token": "for",
      "start_offset": 16,
      "end_offset": 19,
      "type": "<ALPHANUM>",
      "position": 4
    },
    {
      "token": "drink",
      "start_offset": 20,
      "end_offset": 28,
      "type": "<ALPHANUM>",
      "position": 5
    },
    {
      "token": "semi",
      "start_offset": 35,
      "end_offset": 39,
      "type": "<ALPHANUM>",
      "position": 6
    },
    {
      "token": "dry",
      "start_offset": 40,
      "end_offset": 50,
      "type": "<ALPHANUM>",
      "position": 7
    },
    {
      "token": "red",
      "start_offset": 51,
      "end_offset": 54,
      "type": "<ALPHANUM>",
      "position": 8
    },
    {
      "token": "wine",
      "start_offset": 55,
      "end_offset": 59,
      "type": "<ALPHANUM>",
      "position": 9
    }
  ]
}
```

### Lecture 57 - Using analyzers in mappings

* we ve seen how to use the _analyze API to apply analyzers on test strings. now we will learn how to add them in field mappings to use them in the actual indexing of our docs
* it is simple as all it takes is add the anlyzer parameter in our field mapping
* we use our test index again

```
PUT /analyzers_test/default/_mapping
{
  "properties": {
    "description": {
      "type": "text",
      "analyzer": "my_analyzer"
    },
    "teser": {
      "type": "text",
      "analyzer": "standard"
    }
  }
}
# reply
{
  "acknowledged": true
}
```

* we will now add documents for testing with the POST command to index

```
POST /analyzers_test/default/1
{
  "description": "drinking",
  "teaser": "drinking"
}
#reply
{
  "_index": "analyzers_test",
  "_type": "default",
  "_id": "1",
  "_version": 1,
  "result": "created",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 0,
  "_primary_term": 1
}
```

* we will now query the data based on "drinking" term in "teaser" filed using the _search API. Success I get a result

```
GET /analyzers_test/default/_search
{
  "query": {
    "term": {
      "teaser": {
        "value": "drinking"
      }
    }
  }
}
# reply
{
  "took": 2,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 0.2876821,
    "hits": [
      {
        "_index": "analyzers_test",
        "_type": "default",
        "_id": "1",
        "_score": 0.2876821,
        "_source": {
          "description": "drinking",
          "teaser": "drinking"
        }
      }
    ]
  }
}
```

* if i search the same term in the description field i get NO results as drinking was stemmed to drink from the applyed filter.

### Lecture 58 - Adding analyzer definitions to existing indices

* up to now we added analyzers to new indices . now we will learn how to aadd them to existing indices. analyzers term applies to its parts (filters,tokenizers etc)
* we need to CLOSE the index before adding analyzers and OPEN it again afterwards. this is a special case
* Closing an index means it will block read/write operations. we do this with _close API `POST /<index>/_close`

```
POST /analyzers_test/_close
# reply
{
  "acknowledged": true
}
```

* we add the analyzer using the _settings API (similar to when we added analyzers at index creation)

```
PUT /analyzers_test/_settings
{
  "analysis": {
    "analyzer": {
      "french_stop": {
        "type": "standard",
        "stopwords": "_french"
      }
    }
  }
}
# reply
{
  "acknowledged": true
}
```

* we reopen the index with the _ open API `POST /analyzers_test/_open`
* note that we didnt have to do this if we were using the standard analyzers. also we cannot add the  newly added anlyzer to existing mappings. we would have to recreate the index then. for new fields this is not an issue

### Lecture 59 - A word onm Stop Words

* it used to be common practice to remove stop words but not anymore. the relevance algo of elasticsearch is eveloved to include stopwords so it is recommended not to remove them

## Section 7 - Introduction to Searching

### Lecture 61 - Search Methods

* the first method is to use the QUERY DSL (domain spec lang) utilizing the _search API on the index `GET /<index>/<type>/_search` followed by a JSON query

```
GET /<index>/<type>/_search
{
  "query": {
    "<query type>": {
      "<field>": {
        "value": "<some value>"
      }
    }
  }
}
```
* we can shorten the query by ommiting the "value" term `"<field>": "<some value>"`
* another way is using the request URI embedding the search query in the search URI `GET /<index>/<type>/_search?q=<field>:<search val>` . these are refered to as query string queries
* Query DSL supports query string syntax

```
GET /<index>/<type>/_search
{
  "query": {
    "query_string": {
      "query":  "<>field>: <searchvalue>"
    }
  }
}
```

### Lecture 62 - Searching with the Request URI

* [Query String Doc](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-query-string-query.html)

```
GET /product/default/_search?q=*
GET /product/default/_search?q=name:lobster
GET /product/default/_search?q=tags:Meat
```

* = is followed by search query, * means all, key value pairs. field:searchval means search by value
* results are sorted by relevance
* we can chain search queries using boolean operators

```
GET /product/default/_search?q=tags:Meat AND name:Tuna
```

* in cURL spaces must be substituted with %20

### Lecture 63 - Introducting the Query DSL

* we spec the query in an JSON object istead of Request URI. there are 2 types of queries
  * Leaf Queries e.g category = 'Fruit'
  * Compound Queries e.g category = 'Fruit' OR category = 'Vegetable'. vompound queries consiste of multiple leaf queries or compound queries (recursive by nature)

  ```
  GET /product/default/_search
{
  "query": {
    "match_all": {}
  }
}
  ```

  * "match_all": {} means bring all 

### Lecture 64 - Understanding Query Results

```
{
  "took": 1,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 1000,
    "max_score": 1,
    "hits": [
      {
        "_index": "product",
        "_type": "default",
        "_id": "14",
        "_score": 1,
        "_source": {
          "name": "Nori Sea Weed - Gold Label",
          "price": 177,
          "in_stock": 21,
          "sold": 411,
          "tags": [],
          "description": "Nulla ac enim. In tempor, turpis nec euismod scelerisque, quam turpis adipiscing lorem, vitae mattis nibh ligula nec sem. Duis aliquam convallis nunc. Proin at turpis a pede posuere nonummy. Integer non velit. Donec diam neque, vestibulum eget, vulputate ut, ultrices vel, augue. Vestibulum ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia Curae; Donec pharetra, magna vestibulum aliquet ultrices, erat tortor sollicitudin mi, sit amet lobortis sapien sapien non mi. Integer ac neque. Duis bibendum.",
          "is_active": true,
          "created": "2007/02/16"
        }
      },

      ......
    ]
```

* "took": counts the miliseconds it took to execute
* "timed_out": boolean if query timed out
* "_shards": total number of shards that were searched, how many successful and how many with error
* "hits": are the search results
* "max score": is the maximum score achieved
* "hits": [] contains the actual results (matched documents). by default it brings 10 documents per time (configurable)
* the matched document object is the document. it contains
  * the index it belongs, its type, its id, the score it achieved an the _source object with the fields
* hits are sorted per score



* ### Lecture 65 - Understanding Relevance Scores

* elastic returns how well a result matches. this is unique feat in contrast with DBs. Find docs matching the query + How well do the docs match
* relevance score is a float 0.0- 1.0. the algorithm is customizable
* elastic used TF/IDF (Term Frequency/Inverse Term Frequence) algorithm
* now it uses Okapi BM25 algorithm
* Term Frequency (TF): How many times does the term appear in teh filed for a given document
* Inverse Document Frequency (IDF): How often the term appear within the index (across all documents). if it is  a common word e.g stop word) then it has a high IDF score so lower weight (smaller fraction) so not so relevant
* Field-length norm: How long is the field. the smaller the more significant the term (fewer terms)
* the above 3 params are calculated and stored at index time.
* Okapi BM25 algo is better at handling stop words. in the past it was common to remove stop words. now not so common because BM25 is good at it. the relevance of stopwords is suppressed by BM25 but not ignored. this is done by NonLinear Term Frequency Saturation. in BM25() tf() is suppressed while in TF/IDF it increases in a 1- e^x rate. this is done as BM25 applies a limit on how much a term can be boosted by the tf()
* In BM25 Field-length norm also improves each field is treated separately
* BM25 is configurable with 2 params
* its possible to alter the scores. also possible to alter how scores are calculated (advanced stuff)
* we can get an insight on the scoring with the explain param in the JSON query object and get the explanation field in the reply with insight

```
GET /product/default/_search
{
  "explain": true,
  "query": {
    "term": {
      "name": "lobster"
    }
  }
}
# reply
...
"_explanation": {
          "value": 5.6265006,
          "description": "weight(name:lobster in 1) [PerFieldSimilarity], result of:",
          "details": [
            {
              "value": 5.6265006,
              "description": "score(doc=1,freq=1.0 = termFreq=1.0\n), product of:",
              "details": [
                {
                  "value": 4.8777385,
                  "description": "idf, computed as log(1 + (docCount - docFreq + 0.5) / (docFreq + 0.5)) from:",
                  "details": [
                    {
                      "value": 1,
                      "description": "docFreq",
                      "details": []
                    },
                    {
                      "value": 196,
                      "description": "docCount",
                      "details": []
                    }
                  ]
                },
                {
                  "value": 1.153506,
                  "description": "tfNorm, computed as (freq * (k1 + 1)) / (freq + k1 * (1 - b + b * fieldLength / avgFieldLength)) from:",
                  "details": [
                    {
                      "value": 1,
                      "description": "termFreq=1.0",
                      "details": []
                    },
                    {
                      "value": 1.2,
                      "description": "parameter k1",
                      "details": []
                    },
                    {
                      "value": 0.75,
                      "description": "parameter b",
                      "details": []
                    },
                    {
                      "value": 2.9642856,
                      "description": "avgFieldLength",
                      "details": []
                    },
                    {
                      "value": 2,
                      "description": "fieldLength",
                      "details": []
                    }
                  ]
                }
              ]
            }
          ]
        }
      },
      {
        "_shard": "[product][2]",
        "_node": "MtGZ0aFdRtakBTIApOpKiA",
        "_index": "product",
        "_type": "default",
        "_id": "55",
        "_score": 4.4460244,
        "_source": {
          "name": "Lobster - Baby Boiled",
          "price": 134,
          "in_stock": 41,
          "sold": 207,
          "tags": [
            "Meat"
          ],
          "description": "Nulla tellus. In sagittis dui vel nisl. Duis ac nibh. Fusce lacus purus, aliquet at, feugiat non, pretium quis, lectus. Suspendisse potenti. In eleifend quam a odio.",
          "is_active": false,
          "created": "2016/01/19"
        },
...
```

* the value of the term apearing in docs refers to the shard not the complete index

### Lecture 66 - Debugging Unexpected Search Reults

* at some point we will not be able to find a document with our query. it is important to know how to search why the result was not returned. elasticsearch provides the _explain API for this reason. the syntax is

```
GET /<index>/<type>/<id>/_explain
{
  "query": {
    "term": {
      "FIELD": {
        "value": "VALUE"
      }
    }
  }
}
```

* example

```
GET /product/default/1/_explain
{
  "query": {
    "term": {
      "name": "lobster"
    }
  }
}
# reply
{
  "_index": "product",
  "_type": "default",
  "_id": "1",
  "matched": false,
  "explanation": {
    "value": 0,
    "description": "no matching term",
    "details": []
  }
}
```

* this si a good whey to debug if we want to know why a given doc didnt match the query

### Lecture 67 - Query Contexts

* a query can be executed in 2 contexts. query context and filter context.
* when we query in the query context we ask "how well documents match the query"
* in filter context we ask if documents match. doc that dont match wont be returned. in filter context no relevance scores are calculated. its a boolean evaluation. filetring is applied on dates, ranges,status etx. not full text

### Lecture 68 - Full-Text Queries vs Term Level Queries

* we can use 2 types of queries when searching for data. term level and dull -text

* we run a term query with the same term once capitalized and once lowercase

```
GET /product/default/_search
{
  "query": {
    "term": {
      "name": "lobster"
    }
  }
}

GET /product/default/_search
{
  "query": {
    "term": {
      "name": "Lobster"
    }
  }
}
```

* in the first i get 5 hits all with name containing "Lobster" capitalized. the second query gets 0 hits evenif we search with capitalized Lobster
* if i run a full text query with the capitalized version of lobster i get 5 hits

```
GET /product/default/_search
{
  "query": {
    "match": {
      "name": "Lobster"
    }
  }
}
```

* term level queries search for ecact values (terms) in the inverted index where we now by default terms are lowercased
* in full text query the query passes from analysis process like doc itself (using the same analyzer used for the field) so is itself also lowercase before hitting the inverted index
* term level queries are better for numbers dates etc. not sentences

## Section 8 - Term Level Queries

### Lecture 69 - Introduction to Term Level Queries

* used to query structured data like dates and nums, status checks or Enums or keywords, return exact matches.

### Lecture 70 - Searching for a Term

* we will look for all docs that have is_active == true

```
GET /product/default/_search
{
  "query": {
    "term": {
      "is_active": true
    }
  }
}
# OR another syntax
GET /product/default/_search
{
  "query": {
    "term": {
      "is_active": {
        "value": true
      }
    }
  }
}
# i get 487 hits
```

### Lecture 71 - Searching for Multiple Terms

* instead of providing 1 val we provide multiple ones, query returns if it matches any of the provided vals (they are ORed) also instead of "term" wei use "terms"

```
GET /product/default/_search
{
  "query": {
    "terms": {
      "tags.keyword": [
        "Soup",
        "Cake"
      ]
    }
  }
}
```

### Lecture 72 - Retrieving Docs based on IDs

* same like specing an array of terms we can spec an array of IDs (if e.g we store IDs in  DB or if we use same IDs with a DB). instead of "terms" we use "ids"

```
GET /product/default/_search
{
  "query": {
    "ids": {
      "values": [1,2,3]
    }
  }
}
# we get 3 hits
```

### Lecture 73 - Matching Documents w/ Range Vals

* say we want to find the products that are nearly exhasted. "in_stock" between 1 and 5. instead of "term" we use "range" then the "FIELD" and a set of values limiting the range with "gt" "lt"

```
GET /product/default/_search
{
  "query": {
    "range": {
      "in_stock": {
        "gte": 1,
        "lte": 5
      }
    }
  }
}
# result 96 hits
```

* we can use range for dates

```
GET /product/default/_search
{
  "query": {
    "range": {
      "created": {
        "gte": "2010/01/01",
        "lte": "2010/12/31"
      }
    }
  }
}
# result 57 hits
```

* we can apply format in date range query

```
GET /product/default/_search
{
  "query": {
    "range": {
      "created": {
        "gte": "01-01-2010",
        "lte": "31-12-2010",
        "format": "dd-MM-yyyy"
      }
    }
  }
}
```

### Lecture 74 - Working with Relative dates (date math)

* [date and time units](https://www.elastic.co/guide/en/elasticsearch/reference/current/common-options.html#date-math)
* date math or relative dates can be applied in other areas of elasticsearch
* the relative date string starts with the anchor date (now or specific date) next we use the pipe symbol || and then the operation on the anchor date e/g -1y,+2m,-4w.
* the below example returns docs with created date after 1/1/2009 we can stack multiple operands. -1w-1d. 
```
GET /product/default/_search
{
  "query": {
    "range": {
      "created": {
        "gte": "2010/01/01||-1y"
      }
    }
  }
}
# result 57 hits
```

* we can round values in the relative date string with /. we can round for day moth etc. rounding direction for gt and lte is up and for gte and lt is down
* rounding can be placed before or after math depending when we want to round off ` "gte": "2010/01/01||-1y/M"` ` "gte": "2010/01/01||/M-1y"`
* when we set the anchor to now we dont need the 2 pipes. `"gte": "now/M-1y"`

### Lecture 75 - Matching documents with non-null values

* now we will learn how to search for fields that have at least on non-null val at a given field
* a non-null value is a value that is not null (?!?!)
* empty srting matches the exist query an empty array not
* to do this type of query we use the "exists" param

```
GET /product/default/_search
{
  "query": {
    "exists": {
      "field": "tags"
    }
  }
}
# result 316 hits
```

* the above query searches for docs that have at least one tag keyword (non empty array)
* elastic uses the "_field" metafield to search for existence. if we specify a null val at map the doc will match a exists query.

### Lecture 76 - Matching Based on Prefixes

* we can search for terms that begin with a certain prefix.
* the query type that allows us to do that is called "prefix"

```
GET /product/default/_search
{
  "query": {
    "prefix": {
      "tags.keyword": "Vege"
    }
  }
}
# result 34 hits most of which "Vegetable"
```

### Lecture 77 - Searching with Wildcards

* elasticsearch supports a variety of dynamic queries one of which is the wildcard query. this query allows us to use wildcards in a query * and ?. * matches any char sequence including no chars. ? matches any single char

```
GET /product/default/_search
{
  "query": {
    "wildcard": {
      "tags.keyword": "Veg*abl?"
    }
  }
}
# result 34 hits (Vegetable)
```

* wildcard queries are slow especially if wildcards are put in the beginning of the query term

### Lecture 78 - Searching with RegEx

* same drill. we use "regexp" query type and use regex in our query term

```
GET /product/default/_search
{
  "query": {
    "regexp": {
      "tags.keyword": "Veget[a-zA-Z]+ble"
    }
  }
}
# result 34 hits
```

* elastic uses lucene regex engine with is not 100% compatible. regex is applied to the terms in the field and not the complete field
* regex query performance depends on the pattern used (limit use of wildcard pattterns)

## Section 9 - Full Text Queries

### Lecture 79 - Introduction to full-text queries

* full-text queries are used for blocked posts, articles etc
* for this section he uses new test data that he imports in recipe index using dynamic mapping
* we import them ourself with curl

```
curl -H "Content-Type: application/json" -XPOST "http://localhost:9200/recipe/default/_bulk?pretty" --data-binary "@test-data2.json"
```

* we check the mapping created with `GET /recipe/default/_mapping`
* we get a sample doc with 

```
GET /recipe/default/1
# reply
{
  "_index": "recipe",
  "_type": "default",
  "_id": "1",
  "_version": 1,
  "found": true,
  "_source": {
    "title": "Fast and Easy Pasta With Blistered Cherry Tomato Sauce",
    "description": "Cherry tomatoes are almost always sweeter, riper, and higher in pectin than larger tomatoes at the supermarket. All of these factors mean that cherry tomatoes are fantastic for making a rich, thick, flavorful sauce. Even better: It takes only four ingredients and about 10 minutes, start to finish—less time than it takes to cook the pasta you're gonna serve it with.",
    "preparation_time_minutes": 12,
    "servings": {
      "min": 4,
      "max": 6
    },
    "steps": [
      "Place pasta in a large skillet or sauté pan and cover with water and a big pinch of salt. Bring to a boil over high heat, stirring occasionally. Boil until just shy of al dente, about 1 minute less than the package instructions recommend.",
      "Meanwhile, heat garlic and 4 tablespoons (60ml) olive oil in a 12-inch skillet over medium heat, stirring frequently, until garlic is softened but not browned, about 3 minutes. Add tomatoes and cook, stirring, until tomatoes begin to burst. You can help them along by pressing on them with the back of a wooden spoon as they soften.",
      "Continue to cook until sauce is rich and creamy, about 5 minutes longer. Stir in basil and season to taste with salt and pepper.",
      "When pasta is cooked, drain, reserving 1 cup of pasta water. Add pasta to sauce and increase heat to medium-high. Cook, stirring and tossing constantly and adding reserved pasta water as necessary to adjust consistency to a nice, creamy flow. Remove from heat, stir in remaining 2 tablespoons (30ml) olive oil, and grate in a generous shower of Parmesan cheese. Serve immediately, passing extra Parmesan at the table."
    ],
    "ingredients": [
      {
        "name": "Dry pasta",
        "quantity": "450g"
      },
      {
        "name": "Kosher salt"
      },
      {
        "name": "Cloves garlic",
        "quantity": "4"
      },
      {
        "name": "Extra-virgin olive oil",
        "quantity": "90ml"
      },
      {
        "name": "Cherry tomatoes",
        "quantity": "750g"
      },
      {
        "name": "Fresh basil leaves",
        "quantity": "30g"
      },
      {
        "name": "Freshly ground black pepper"
      },
      {
        "name": "Parmesan cheese"
      }
    ],
    "created": "2017/03/29",
    "ratings": [
      4.5,
      5,
      3,
      4.5
    ]
  }
}
```

### Lecture 80 - Flexible Matching with the Match Query

* better matches text fields and human writen queries

```
GET /recipe/default/_search
{
  "query": {
    "match": {
      "title": "recipes with pasta or spagetti"
    }
  }
}
# results are sorted by relevance score 16 hits

  "took": 8,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 16,
    "max_score": 1.7715604,
    "hits": [
      {
        "_index": "recipe",
        "_type": "default",
        "_id": "2",
        "_score": 1.7715604,
        "_source": {
          "title": "Pasta With Butternut Squash and Sage Brown Butter",
          "description": "Brown butter-based pasta sauces are some of the simplest things around. They're emulsions made with a flavorful fat and pasta cooking water that coats the pasta in a thin, creamy sheen of flavor. Throw in some sautéed squash and some sage and you've got yourself a great 30-minute meal. It's a classic fall and winter dish that can be made right on the stovetop.",
          "preparation_time_minutes": 30,
          "servings": {
            "min": 4,
            "max": 6
          },
          "steps": [
            "Heat olive oil in a large stainless steel or cast-iron skillet over high heat until very lightly smoking. Immediately add squash, season with salt and pepper, and cook, stirring and tossing occasionally, until well-browned and squash is tender, about 5 minutes. Add butter and shallots and continue cooking, stirring frequently, until butter is lightly browned and smells nutty, about 1 minute longer. Add sage and stir to combine (sage should crackle and let off a great aroma). Remove from heat and stir in lemon juice. Set aside.",
            "In a medium saucepan, combine pasta with enough room temperature or hot water to cover by about 2 inches. Season with salt. Set over high heat and bring to a boil while stirring frequently. Cook, stirring frequently, until pasta is just shy of al dente, about 2 minutes less than the package directions. Drain pasta, reserving a couple cups of the starchy cooking liquid.",
            "Add pasta to skillet with squash along with a splash of pasta water. Bring to a simmer over high heat and cook until the pasta is perfectly al dente, stirring and tossing constantly and adding a splash of water as needed to keep the sauce loose and shiny. Off heat, stir in Parmigiano-Reggiano. Adjust seasoning with salt and pepper and texture with more pasta water as needed. Serve immediately, topped with more cheese at the table."
          ],
          "ingredients": [
            {
              "name": "extra-virgin olive oil",
              "quantity": "30ml"
            },
            {
              "name": "Butternut squash",
              "quantity": "450g"
            },
            {
              "name": "Kosher salt"
            },
            {
              "name": "Freshly ground black pepper"
            },
            {
              "name": "Unsalted butter",
              "quantity": "30g"
            },
            {
              "name": "Shallot",
              "quantity": "1"
            },
            {
              "name": "Fresh sage leaves",
              "quantity": "15g"
            },
            {
              "name": "Lemon juice",
              "quantity": "15ml"
            },
            {
              "name": "Penne",
              "quantity": "450g"
            },
            {
              "name": "Grated fresh Parmigiano-Reggiano cheese",
              "quantity": "30g"
            }
          ],
          "created": "2014/12/19",
          "ratings": [
            3,
            4,
            3.5,
            2
          ]
        }
      },
    ............
```

* our best match does not contain the term spagetti. thi is because analyde result terms are ORed
* we ususally leave the boolean operator as is but we can change it

```
GET /recipe/default/_search
{
  "query": {
    "match": {
      "title": {
        "query": "Recipes with pasta or spagηetti",
        "operator": "and"
      }
    }
  }
}
# results 0 hits (this is a strickt query)
```

* order of terms in our query doesnt matter

### Lecture 81 - Matching Phrases

* if i want to match a phrase aka words in a specific order i use "match_phrase" query type instead of "match"

```
GET /recipe/default/_search
{
  "query": {
    "match_phrase": {
      "title": "spaghetti puttanesca"
    }
  }
}
# 1 hit with title "Spaghetti Puttanesca (Pasta or Spaghetti With Capers, Olives, and Anchovies)"
```
* if i swap the order of workds in my phrase i get 0 hits. order matters in phrase

### Lecture 82 - Searching Multiple Fields

* [multi match fields](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-multi-match-query.html#multi-match-types)
* is easy using a query type "multi_match". the syntax is an objecct with a query string to be searchin in multiple fields (specsd in an array). it is like match searching in both fields. the highest relevance score between both fields is used.  so if one field contain both terms and the other none scores higher that if each term appears in each fields. this is paramtertical

```
GET /recipe/default/_search
{
  "query": {
    "multi_match": {
      "query": "pasta",
      "fields": ["title","description"]
    }
  }
}
```

## Section 10 - Adding Boolean Logic to Queries (Compound Queries)

### Lecture 83 - Intro to Compound Queries

* bollean/compound queries wrap other queries (leaf queries) to construct more complex ones

### Lecture 84 - Queying w/ Boolean Logic

* a query can be run in a query or filter context. query = relevance, sorted results. filter = bollean logic yes or no
* we use context, in its object we use "bool" query type, in its object we use the operand in our case AND named "must" an in it an array of the individual queres. these are specd by their type and object. e.g match or range

```
GET /recipe/default/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "ingredients.name": "parmesan"
          }
        },
        {
          "range": {
            "preparation_time_minutes": {
              "lte": 15
            }
          }
        }
      ]
    }
  }
}
# results in 3 hits
```

* relevance scores are calced for each one and then combined, using query context for a range query is overkill as there is match or no match. elastic sets score 1 in range, but filter is faster. we refactor the query. see that range moces out of the must array  into a sepaerate object and is wrapped in a filter context. result is the same

```
GET /recipe/default/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "ingredients.name": "parmesan"
          }
        }
      ],
      "filter": {
        "range": {
          "preparation_time_minutes": {
            "lte": 15
          }
        }
      }
    }
  }
}
```

* i can cascade multiple bollean operations in my complex boll query. i use "must_not" aka NAND as an array passing unwanted match queries to excluse
* i add one more operand the "should" caluse with its query array. should is more relaxed than must. it enhances the results but do not confine them (us e the score of this query to score higher but it do not deduct score if not found)
* there is also minimu should which ake more strict the should clause

### Lecture 85 - Debugging bool queries with named queries

* we can use the _explain API we saw before to debug complex queries. to use it  we need to name our individual queries using the "_name" param to THE QUERIES in the compound

```
GET /recipe/default/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "ingredients.name": {
              "query": "parmesan",
              "_name": "parmesan_must"
            }
          }
        }
      ],
      "must_not": [
        {
          "match": {
            "ingredients.name": {
              "query": "tuna",
              "_name": "tuna_must_not"
            }
          }
        }
      ],
      "should": [
        {
          "match": {
            "ingredients.name": {
              "query": "parsley",
              "_name": "parsley_should"
            }
          }
        }
      ], 
      "filter": {
        "range": {
          "preparation_time_minutes": {
            "lte": 15,
            "_name": "range_filter"
          }
        }
      }
    }
  }
}
```

* in the results for each doc retrieved i have the matched queries array for the queries that matched for this doc

### Lecture 86 - How the "match" Query Works

* match query by default uses the or operand internaly for the terms produced by the string we pass. this can be chaned to and as we ve seen. so internally it is a compound query, doing this as part of the analysis

```
GET /recipe/default/_search
{
  "query": {
    "match": {
      "title": "pasta carbonara"
    }
  }
}
```
* the above full text match query is a bool query with OR on the 2 terms produced by the phrase. the clauses it uses are the "should" of "term" queries. it is equivalent to the following one

```
GET /recipe/default/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "term": {
            "title": {
              "value": "pasta"
            }
          }
        },
        {
          "term": {
            "title": {
              "value": "carbonara"
            }
          }
        }
      ]
    }
  }
}
```

* if i use the "and" operator is equivalent to the "must" query type
* match query is analyzed but term query not. so what about capitalization and other default filters (the terms we used were the wequivalent of the analyzer products as they appear in the inverted index). if we capitalize the terms in the term queries  there no equality with the match query

## Section 11 - Joining Queries

### Lecture 87 - Intro to the Section

* use join datatype
* [Join Data Type](https://www.elastic.co/guide/en/elasticsearch/reference/master/parent-join.html)

### Lecture 88 - Querying Nested Objects

* nested is used for arrays of objects when we want to maintain the relationship between object properties. if a filed is defined as an array of objects an object is not independent because the object are mixed together when store in elastisearch.
* we createe a new index named Departmentwhich contains a mapping for two two fields name and employees. employees is of "nested" as it will contain an array of objects. object fields are not mapped as  they will be dynamic mapped by elastic when we add docs.

```
PUT /department
{
  "mappings": {
    "default": {
      "properties": {
        "name": {
          "type": "text"
        },
        "employees": {
          "type": "nested"
        }
      }
    }
  }
}
```

* next we add a doc in the new index with a number of employees. each object in the array contains name,age,gender,position

```
POST /department/default/1
{
  "name": "Development",
  "employees": [
    {
      "name": "Eric Green",
      "age": 39,
      "gender": "M",
      "position": "Big Data Specialist"
    },
    {
      "name": "James Taylor",
      "age": 27,
      "gender": "M",
      "position": "Software Developer"
    },
    {
      "name": "Gary Jenkins",
      "age": 21,
      "gender": "M",
      "position": "Intern"
    },
    {
      "name": "Julie Powell",
      "age": 26,
      "gender": "F",
      "position": "Intern"
    },
    {
      "name": "Benjamin Smith",
      "age": 46,
      "gender": "M",
      "position": "Senior Software Engineer"
    }
  ]
}
```

* we add a second doc and now we are ready to query the nested field (employees). we want to know which departments have female interns. as these are 2 queries we wrap them in a bool query. so with what we knbow we form a boolean query

```
GET /department/default/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "employees.position": "intern"
          }
        },
        {
          "term": {
            "employees.gender.keyword": {
              "value": "F"
            }
          }
        }
      ]
    }
  }
}
```

* we get 0 hits. nested types are not queried this way. they need a speacial query type named "nested". "nested" wraps a "path" property . this sis the path to the array of objects our wrapped query will be performed on. as our nested query will be perfomed on each object in the array. as 1to1 relationship only nested query type can reconstruct it and search it

```
GET /department/default/_search
{
  "query": {
    "nested": {
      "path": "employees",
      "query": {
        "bool": {
          "must": [
            {
              "match": {
                "employees.position": "intern"
              }
            },
            {
              "term": {
                "employees.gender.keyword": {
                  "value": "F"
                }
              }
            }
          ]
        }
      }
    }
  }
}
```

## Section 12 - Controlling Query Results

### Lecture 89 - Specifying the result format

* we will see how to format the result in YAML instead of JSON, and how to make results pretty in case of JSON
* to get results in yaml we set in usi the param format=yaml

```
GET /recipe/default/_search?format=yaml
{
  "query": {
    "match": {
      "title": "pasta"
    }
  }
}
# result
---
took: 2
timed_out: false
_shards:
  total: 5
  successful: 5
  skipped: 0
  failed: 0
hits:
  total: 9
  max_score: 1.2756016
  hits:
.............
```
* making JSON pretty is irrelevant for kibana de tools console as it does prettygy JSON. but in files or raw format we need to pretify JSON. we do this with ?pretty query param in URI. if we cp the query in curl and remove the generated ?pretty flag our result is compressed. prettifying JSON consumes resources

```
#result
{"took":2,"timed_out":false,"_shards":{"total":5,"successful":5,"skipped":0,"failed":0},"hits":{"total":9,"max_score":1.2756016,"hits":[{"_index":"recipe","_type":"default","_id":"2","_score":1.2756016,"_source":{"title":"Pasta With Butternut Squash and Sage Brown Butter","description":"Brown butter-based pasta sauces are some of the simplest things around. They're emulsions made with a flavorful fat and pasta cooking water that coats the pasta in a thin, creamy sheen of flavor. Throw in some saut\u00e9ed squash and some sage and you've got yourself a great 30-minute meal. It's a classic fall and winter dish that can be made right on the stovetop.","preparation_time_minutes":30,"servings":{"min":4,"max":6},"steps":["Heat olive oil in a large 
```

### Lecture 90 - Source Filtering

* we will learn to control which parts of the "_source" field are returned for each of the matches
* by default the whole content gets returned
* the rationale for limiting whats returned is to reduce traffic in network
* in apps with a lot of documents or long textfields it can make a difference.
* we can disable the complete "_source" altogether (if we want just ids) and then retrieve docs from other data store
* we can disable "_source" by preceding the "query" with "_source": false

```
GET /recipe/default/_search
{
  "_source": false, 
  "query": {
    "match": {
      "title": "pasta"
    }
  }
}
```

* return  only one field from "_source" `"_source`: "created"
* for nested fields we use "<parent>.<child>"" `"_source": "ingredients.name"`
* we can use wildcards * to get all fiends fom an object `"_source": "ingredients.*"
`
* if i want to get back a number of fields i set them in an array `"_source": ["ungredients.*", "created"]`
* i can include and exclude by using the following syntax

```
"_source": {
  "includes": "ingredients.*",
  "excludes": "ingredients.name"
}
```
* its like selecting which columns t rget back in a relational db query

### Lecture 91 - Specifying the result size

* we can define the number of hits to be returned in the URI on in the Request DSL body
* URI `GET /<index>/<type>/_search?size=<max hits>`

```
GET /recipe/default/_search?size=2
{
  "_source": false, 
  "query": {
    "match": {
      "title": "pasta"
    }
  }
}
# reply
{
  "took": 1,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 9,
    "max_score": 1.2756016,
    "hits": [
      {
        "_index": "recipe",
        "_type": "default",
        "_id": "2",
        "_score": 1.2756016
      },
      {
        "_index": "recipe",
        "_type": "default",
        "_id": "8",
        "_score": 1.2540693
      }
    ]
  }
}
```

* the hit number is the total but hte hits array contains the
* if i want to set the size limit in body (DSL query)

```
GET /recipe/default/_search
{
  "size": 2,
  "_source": false, 
  "query": {
    "match": {
      "title": "pasta"
    }
  }
}
```

* the default size is 10

### Lecture 92 - Specifying an  offset

* offset allows s to paginate the results returned. this is useful when total hits exceed the default 10 or the size we specify.
* size + offset do pagination. offset is set with "from" se either in URI or in body like the "size"

```
GET /recipe/default/_search
{
  "size": 2,
  "from": 4, 
  "_source": false, 
  ...
```

### Lecture 93 - Pagination

* in our elasticsearch client source code we need to calculate total_page = ceil(total_hits/page_size)
* offset calculation: from (page_size * (page_number -1))
* queries are stateless. if new data arrive our next page will contain tehm

### Lecture 94 - Sorting Results

* the way to sort results is by adding teh "sort" param in the query body. its value can be an array of the fields we wanto sort by

```
GET /recipe/default/_search
{
  "_source": false, 
  "query": {
    "match_all": {}
  },
  "sort": [
    "preparation_time_minutes"
  ]
}
```

* the hits contain the sort field with the sorted field val in ascending order (default)
* if we want better control on sort we pass order criteria as objects int he array

```
  "sort": [
      {"created": "desc"}
  ]
```

* in sorting dates are used as timestamps
* if i want to sort on multile fields i put more elements in the sort array

### Lecture 95 - Sorting with multi-value fields

* we can sort on fields containing multiple vals

```
GET /recipe/default/_search
{
  "_source": "ratings", 
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "ratings": {
        "order": "desc",
        "mode": "avg"
      }
    }
  ]
}
```

* in the above example the ratings field is an array of floats. we sort on it in the standard way. however in the ratings field in the sort array we dond pass a simple value e.g desc but an object with two params. the order (asc, desc) and the mode (min,max,avg). the mode is what operates on the mutiple field values. produces a value on which we sort

### Lecture 96 - Filters

* we will talk onthe second context of elasticearch search API. the filter context. as we said before filter contex is boolean mathes or no (no relevance scores used). they are used on numbers and dates  eywords, enums. rather than text
* in the past fitler context was used at the same level as query (root of search JSON body). in latestt version it is used as a query type usualy in a boolean query (composed query). it fits in a boolean query as it returns a boolean

```
GET /recipe/default/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "title": "pasta"
          }
        }
      ],
      "filter": [
        {
          "range": {
            "preparation_time_minutes": {
              "lte": 15
            }
          }
        }
      ]
    }
  }
}
```

* filters are more efficient, elasticsearch caches frequently used filters

## Section 13 - Aggregations

### Lecture 97 - Intro to Aggregations

* agreggations are more powerfull than DB
* say we have an index of orders with fields product and amount sold per order. 
* we can aggregate the total amount of porduct A units sold in all orders
* in this section we work with order index. we populate it with test data.

```
curl -H "Content-Type: application/json" -XPOST "http://localhost:9200/order/default/_bulk?pretty" --data-binary "@orders-bulk.json"
```

* the course has order doc lines as nested object. we dont. it has to do withdefault mapping. maybe we need to do explicit mapping insted, we need to enter "type": "nested" in lines

### Lecture 98 - Metric Aggregations

* [all metric aggregations](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-metrics.html)
* Single Value & Multi-Value Numeric Metric Aggregations
* sum aggregations sumps up thenumbers from fields in retreived docs from query

```
GET /order/default/_search
{
  "size": 0,
  "aggs": {
    "total_sales": {
      "sum": {
        "field": "total_amount"
      }
    }
  }
}
# result
{
  "took": 43,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 1000,
    "max_score": 0,
    "hits": []
  },
  "aggregations": {
    "total_sales": {
      "value": 109209.60997009277
    }
  }
}
```

* what we did is we run a global query in the JSON body se speced size=0 (w dont care to get back the docs) and an aggregations "aggss" object where we set the query field name we want to use for the results. the operation ("sum" = add) and the doc field which values we want to add
* what we get back is a query results with "agggregations" property wehre we can find the result with the name we speced in the query body
* we can apply other operands "avg" "min" "max"
* if we want to count the distinc values of a field in the retrieved docs we use the "cardinality" aggrgation type. with that we can count the disctinct slasemen ids in all orders. this gives back aproximates as accurate results consume many resources
* the "value_count" aggregation type counts all the values of the field uses in an aggregation (it should be equal to total_hits for non-null fields)
* all the above were single-value aggregations
* a lulti-val is "stats" which calculates amultitude of aggregations

```
GET /order/default/_search
{
  "size": 0,
  "aggs": {
    "amount_stats": {
      "stats": {
        "field": "total_amount"
      }
    }
  }
}
# reply
..........
"aggregations": {
    "amount_stats": {
      "count": 1000,
      "min": 10.270000457763672,
      "max": 281.7699890136719,
      "avg": 109.20960997009277,
      "sum": 109209.60997009277
    }
  }
```

### Lecture 99 - Bucket Aggregations

* bucket aggregations create buckets of documents, buckets have criteria to determine if a doc will be contained in them
* the "terms" aggregation creates a bucket for each unique value (e.g group orders depending on the value of status field)

```
GET /order/default/_search
{
  "size": 0,
  "aggs": {
    "status_terms": {
      "terms": {
        "field": "status",
        "missing": "N/A",
        "order": {
          "_term": "asc"
        }
      }
    }
  }
}
```

* this query results in aggregations contains a bucket list with buckets weach one has a key (status distinct term) and the doc count
* elastic ueses the top terms for buckets (size?) and the rest are noncategorized so the doc count of uncategorized docs is set in theother doc counter
* we can add a "missing" param with a name for a bucket that will contain docs with no value
* we can set a "min_doc_count" param with the min num of docs a bucket must have to be present in the bucket list
* calculate doc counts are approximate
* we canset an order int he bucket list

### Lecture 100 - Document counts are approximate

* the reson that document counts are approximate is due to the distributed nature of elasticsearch cluster. an index is distributed in multiple shards. 
* in term aggregation. master node prompts others for their top unique terms. master node tkes the results and calculates the sums. it might be that terms are not evenly distributed accross shards
* the problem is bigger when size param is low. default size is 10
* if all index is in one shard the results are acuratet
* "doc_count_error_upper_bound" gives the error margin

### Lecture 101 - Nested Aggregations

* Bucket aggregations can have nested aggregations (sub aggregations)
* we use the "aggs" key in the buckets. backets are actually sub indexes so we can apply aggregations in them
* this is recursive. we can have bucket aggregations in bucket aggregations and so one.

```
GET /order/default/_search
{
  "size": 0,
  "aggs": {
    "status_terms": {
      "terms": {
        "field": "status"
      },
      "aggs": {
        "status_stats": {
          "stats": {
            "field": "total_amount"
          }
        }
      }
    }
  }
}
```

* the above creates a stat aggregation in each bucket. the numbers are relevant to the bucket they are contained
* metric aggregations cannot have sub aggregations

### Lecture 102 - Filtering out Documents

* the document which the aggregation uses depends on the context the aggregation is defined
* we can filter the docs on which the aggregation is applied regardles of the parent query using a filter
* a filter uses a query clause (term query or match query)

```
GET /order/default/_search
{
  "size": 0,
  "aggs": {
    "low_value": {
      "filter": {
        "range": {
          "total_amount": {
            "lte": 50
          }
        }
      },
      "aggs": {
        "avg_amount": {
          "avg": {
            "field": "total_amount"
          }
        }
      }
    }
  }
}
# results
{
  "took": 7,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 1000,
    "max_score": 0,
    "hits": []
  },
  "aggregations": {
    "low_value": {
      "doc_count": 164,
      "avg_amount": {
        "value": 32.59371952894257
      }
    }
  }
}
```

* we see that query returns 1000 docs. but the filter we apply limits the number the aggregation runs on to 164.

### Lecture 103 - Defining bucket rules with filters

* we will see "filters" aggregagations to define rules on which bucket docs are placed into. it defines query names based on which docs are place on buckets. the synstax is weird
* we have "filters" aggrgation with a "filters" clause in it. in it we define the bucket names with a query in them to filter the docs that go in the bucket

```
GET /recipe/default/_search
{
  "size": 0,
  "aggs": {
    "my_filter": {
      "filters": {
        "filters": {
          "pasta": {
            "match": { 
              "title": "pasta"
            }
          },
          "spaghetti": {
            "match": {
              "title": "spaghetti"
            }
          }
        }
      },
      "aggs": {
        "avg_rating": {
          "avg": {
            "field": "ratings"
          }
        }
      }
    }
  }
}
# result
{
  "took": 2,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 21,
    "max_score": 0,
    "hits": []
  },
  "aggregations": {
    "my_filter": {
      "buckets": {
        "pasta": {
          "doc_count": 9,
          "avg_rating": {
            "value": 3.4125
          }
        },
        "spaghetti": {
          "doc_count": 4,
          "avg_rating": {
            "value": 2.3684210526315788
          }
        }
      }
    }
  }
}
```

* of course we can add subaggregations in the buckets

### Lecture 104 - Range Aggregations

* there are 2 range aggregations "range" and "date_range" optimized for dates
* they define ranges each of which represents a bucket of docs (a range rule per bucket)
* the syntax is: "range" is the aggregation type. then the field it is applied. the with "ranges" we have an array or range rules that set the buckets

```
GET /order/default/_search
{
  "size": 0,
  "aggs": {
    "amount_distribution": {
      "range": {
        "field": "total_amount",
        "ranges": [
          {
            "to": 50
          },
          {
            "from": 50,
            "to": 100
          },
          {
            "from": 100
          }
        ]
      }
    }
  }
}
# reply
{
  "took": 5,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 1000,
    "max_score": 0,
    "hits": []
  },
  "aggregations": {
    "amount_distribution": {
      "buckets": [
        {
          "key": "*-50.0",
          "to": 50,
          "doc_count": 164
        },
        {
          "key": "50.0-100.0",
          "from": 50,
          "to": 100,
          "doc_count": 347
        },
        {
          "key": "100.0-*",
          "from": 100,
          "doc_count": 489
        }
      ]
    }
  }
}
```

* from is included and to is excluded
* for date range instead of "range" we use "date_range", also in from and to we use dates (date math is allowed)
* we can pass a date format in the date range aggregation, format is defined at the top under the field

```
GET /order/default/_search
{
  "size": 0,
  "aggs": {
    "purchased_ranges": {
      "date_range": {
        "field": "purchased_at",
        "format": "yyyy-MM-dd", 
        "ranges": [
          {
            "from": "2016-01-01",
            "to": "2016-01-01||+6M"
          },
          {
            "from": "2016-01-01||+6M",
            "to": "2016-01-01||+1y"
          }
        ]
      },
      "aggs": {
        "bucket_stats": {
          "stats": {
            "field": "total_amount" 
          }
        }
      }
    }
  }
}
```

### Lecture 105 - Histograms

* histogram is a good way to define range for buckets if we dont know the values beforehand. with the "histogram" aggregation type we set an interval and elasticsearch distributes dynamicaly the docs in buckets in ranges set by intervals. 

```
GET /order/default/_search
{
  "size": 0,
  "query": {
    "range": {
      "total_amount": {
        "gte": 100
      }
    }
  }, 
  "aggs": {
    "amount_distribution": {
      "histogram": {
        "field": "total_amount",
        "interval": 25,
        "min_doc_count": 0,
        "extended_bounds": {
          "min": 0,
          "max": 500
        }
      }
    }
  }
}
```

* docs are rounded to nearest bucket
* with "min_doc_count" we can set a minimum  count per bucket (eg to prevent empty buckets)
* with extended bounds we explicitly define the start anbd end of buckets even if buckets are empty (this contradicts with min_doc_count)
* date histogram is supported , intervals are specd in date units
* offset is supported and format as well
```
GET /order/default/_search
{
  "size": 0,
  "aggs": {
    "orders_over_time": {
      "date_histogram": {
        "field": "purchased_at",
        "interval": "month"
      }
    }
  }
}
```

### Lecture 106 - Global Aggregation

* with aggregation type "global" we break out of query limitation so that even if query limits the results on which aggregation will be applied, global apllies the aggregation to the whole index

```
GET /order/default/_search
{
  "query": {
    "range": {
      "total_amount": {
        "gte": 100
      }
    }
  },
  "size": 0,
  "aggs": {
    "all_orders": {
      "global": {},
      "aggs": {
        "stats_amount": {
          "stats": {
            "field": "total_amount"
          }
        }
      }
    }
  }
}
```

* global creates a global bucket on which we run the sub aggregations
* there is no point to place a global aggregation as a sub-aggregation

### Lecture 107 - Missing field values

* what if we aggregate on a field and some docs dont have this field

```
GET /order/default/_search
{
  "size": 0,
  "aggs": {
    "orders_without_status": {
      "missing": {
        "field": "status"
      }
    }
  }
}
# returns docs with not status (missing or null)
```

* i can add subagreggations to teh missing bucket

### Lecture 108 - Aggregating nested objects

* we try to issue an aggregation on a nested object field of the docs . the result is null

```
GET /department/default/_search
{
  "size": 0,
  "aggs": {
    "minimum_age": {
      "min": {
        "field": "employees.age"
      }
    }
  }
}
```

* like nested queries we need to set the path t o the nested object field
* we formulte anested aggregation

```
GET /department/default/_search
{
  "size": 0,
  "aggs": {
    "employees": {
      "nested": {
        "path": "employees"
      }
    }
  }
}
```

* we get a bucket of 15 docs. so we can run sub aggregation on this

```
GET /department/default/_search
{
  "size": 0,
  "aggs": {
    "employees": {
      "nested": {
        "path": "employees"
      },
      "aggs": {
        "minimum_age": {
          "min": {
            "field": "employees.age"
          }
        }
      }
    }
  }
}
```

## Section 14 - Improving Search Results


### Lecture 111 - Proximity searches

* match phrase queries are quite strict. they require the terms in the phrase we spec out to appear in the field and to be in the order we spec out and in the distance we spec out (no in between words). this is because the phrase we use in the query passes from the smae analyzer like the text we apply it to. n inverted index terms have index and this index is used in the match phrase query
* if we want to relax out query and let in between words in the text inverted index deviating from our exact query phrase we use the param "slop" and a number which  sets the allowed distance(in between terms) between the terms produced by our query . this makes the query a proximity query. we can say we search for the terms produced in our phrase within a proximity of one to the other speced by slop
. if our defined slop distance distances the terms so much that they exceed the length of the searched text the inverted order of terms returns a match

### Lecture 112 - Affecting relevance scoring with proximity

*the proximity in which terms appear affects the relevance. the closest the proximity the highest the relevance
* we want to create a complex query where we promote terms in close proximity but we dont retrict to it. match query is a full text so it is not wxclusive rather thanrelevance driven. in match query the default operator between terms is or

```
GET /proximity/default/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "title": {
              "query": "spicy sauce"
            }
          }
        }
      ],
      "should": [
        {
          "match_phrase": {
            "title": {
              "query": "spicy sauce",
              "slop": 5
            }
          }
        }
      ]
    }
  }
```

### Lecture 113 - Fuzzy match query (handling typos)

* the easiest way to handle user induces typos in qurty phrase is to add parameter "fuzziness" to the match query 

```
GET /product/default/_search
{
  "query": {
    "match": {
      "name": {
        "query": "l0bster",
        "fuzziness": "auto"
      }
    }
  }
}
```

* fuziness is defined with a number or auto. the number sets the number of characters that can deviate in our phrase from the inverted index terms (edit distance). max edit distance allowed is 
* auto fuziness works as follows: term size 1-2 => edit distance - 0, term size 3-5 => max edit distance = 1, term size >5 => max edit distance = 22
* in match queries of multiple terms the fuziness edit distanceis shared among words. also to use it we need the operator to be and
* edit distace is the levenstein distance. another scientist added trasposition to the allowed edits (swaping letter order). we can disable transposition with `"fuzzy_transpositions": false`

### Lecture 114 - Fuzzy Query

* instead of adding fuzziness to a  match query there is a dedicaed query type called "fyzzy"

```
GET /product/default/_search
{
  "query": {
    "match": {
      "name": {
        "query": "LOBSTER",
        "fuzziness": "auto"
      }
    }
  }
}
# returns 5 results
GET /product/default/_search
{
  "query": {
    "fuzzy": {
      "name": {
        "query": "LOBSTER",
        "fuzziness": "auto"
      }
    }
  }
}
# returns 0 results
```

* fuzzy query is a term level query so not analyzed so not lowercased bya filter
* other than that it works the same way

### Lecture 115 - Adding Synonyms

* we add an index called synonyms to the cluster with a custom analyzer and custom totken filter, of type synonyms passing a synonyms list. 
* the synonyms array has the syntax ["ayful => teriible", "awesome => great, super"]. this means replace term awful with terrible in inverted index
* query go through same analisys so both awful and terrible will give a match. however term level queries dont use the analyzer
* awesome produces 2 terms in inverted index so all 3 are treated the same after the analysis (i can use duble words in synonyms array). both synonyms are places in the same position in teh inverted index
* the reason fro keeping both terms in same position is to perform proximity searches if required
* synonims multiplicity can be opossite. one on right multiple on left "elasicsearch,logstash,kibana => elk", or i can put multiple words in teh array iwithout assignemtn to put them in same position in inverted index "weird,strange"
* fitler order matters,  in the example lowercase filter is applied before synonyms filter. so our term assignemtns must be all lower case other wise we have bproblem
* synonim filter must be placed before stemming and as early as possible in fitler array (except lowercase)
* we can use _analyze API to test synonyms

### Lecture - Adding Synonyms from File

* we use "synonyms_path" param passing a path to the file
* we can use relative path or absolute

```
PUT /synonyms
{
  "settings": {
    "analysis": {
      "filter": {
        "synonym_test": {
          "type": "synonym",
          "synonyms_path": "analysis/synonyms.txt"
        }
      },
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "synonym_test"
          ]
        }
      }
    }
  },
  "mappings": {
    "default": {
      "properties": {
        "description": {
          "type": "text",
          "analyzer": "my_analyzer"
        }
      }
    }
  }
}
```

* our file is a simple txt file with 1 rule per line , no quotation marks comments with #

`````
# synonyms.txt content

awful => terrible
awesome => great,super
elasticsearch, logstash, kibana => elk
weird, strange
`````

* if we add rules to the txt file and restart node then the existing documents are not updated .as indexing has already taken place. it affects new documents or updted old oones, this can create inconsistencies as the docs inverted intex does not contain the synonims while the query does as it goes throug the doc
* the solution is _update_by_query API as it reindexes the docs

### Lecture 117 - Highlighting matches in fields

* [Highlighting](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-highlighting.html)
* elasticsearch can highlight matches mimicing popular search engines using a highlighter
* test doc added to new index

```
POST /highlighting/default/1
{
  "description": "Let me tell you a story about Elasticsearch. It's a full-text search engine that is built on Apache Lucene. It's really easy to use, but also packs lots of advanced features that you can use to tweak its searching capabilities. Lots of well-known and established companies use Elasticsearch, and so should you!"
}
```

* our query is formed, and we spec that we don't want the actual doc returned. but we add a param "highlight" with its syntax

```
GET /highlighting/default/_search
{
  "_source": false,
  "query": {
    "match": {
      "description": "Elasticsearch story"
    }
  },
  "highlight": {
    "fields": {
      "description": { }
    }
  }
}
# result
{
  "took": 4,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 0.68324494,
    "hits": [
      {
        "_index": "highlighting",
        "_type": "default",
        "_id": "1",
        "_score": 0.68324494,
        "highlight": {
          "description": [
            "Let me tell you a <em>story</em> about <em>Elasticsearch</em>.",
            "Lots of well-known and established companies use <em>Elasticsearch</em>, and so should you!"
          ]
        }
      }
    ]
  }
}
```

* we see that reply includes a "higlight" object with the terms emphasized with HTML tags. these are text fragments
* in indexing elasticsearch uses the analyzer for text and stores with the term also the start offset and the end offset in teh invertedf index. these offsets are used by highlight, synonyms also are taken into consideration when highlighting
* we can customize the tags with the "pre_tags": ["<strong>"] "post_tags": ["</strong>"]

### Lecture 118 - Stemming

* highlighter highlights stem words as well
* we add a stemmer filter in the analyzer and run some queries
