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