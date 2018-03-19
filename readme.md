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