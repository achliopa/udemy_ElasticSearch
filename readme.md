# Udemy Lecture: Complete Guide to Elasticsearch
[course](https://www.udemy.com/elasticsearch-complete-guide/learn/v4/overview)
[repo](https://github.com/codingexplained/complete-guide-to-elasticsearch)

## IMPORTANT UPDATE: since version 7 types will be removed and default type should be replace with the *_doc* type 

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

```
