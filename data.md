# 2. Data


// note: in the beginning, include a short introduction to the chapter. Provide an overview about the things talked about in the chapter and discuss why they are relevant.


* (refer sharding data from previous chapter) Though, sharding data is not enough in case something catastrophic happens to the cluster and some data - or even all data - is damaged or deleted. (e.g.) When for some reason data is deleted or the cluster enters in a faulty mode, it is desirable to have data backups and a process to restore the cluster state and data into a new cluster.


## How to backup Elasticsearch data?

**TL;DR**
```
Elasticsearch includes the snapshot and restore modules for creating and managing backups. With those modules, you can create snapshots of individual or a set of indexes and store them in several different places, such as shared filesystem, Amazon S3 and others. The backups can be created and maintained through the API or through the Elastic Curator tool.
```

#### What are Elasticsearch repositories and snapshots?

Repositories and snapshots are essential concepts for understanding Elasticsearch backups. 

Repositories are remote archives where the backups are stored. Elasticsearch supports multiple types of repositories like:

 - **Shared remote file system** 
 - **Amazon S3**
 - **Azure Cloud**
 - **Hadoop Distributed File System**


A shapshot is a copy of individual or a set of indexes. The snapshots are the data that can be stored into the repositories for backup purposes. It is possible to take snapshots of individual indexes or the entire ES cluster. 

#### How to configure a repository?

#### How to create snapshots?

#### How to monitor the backups?

#### Is there any official tool for maintaining Elasticsearch backups?

Yes there is. Elastic Curator (https://github.com/elastic/curator) is a neat tool that helps maintaining Elasticsearch clusters' indexes. It includes features that help to create and maintain backups such as taking indexes snapshots, deleting indexes and snapshots.

Curator provides a CLI interface and HTTP API..


## How to recover from cluster red status when shards are unassigned?

**TL;DR**
```
First, inspect which shards are having problems and which indexes are affected. Once you know the problematic index, stop the data flow in Elasticsearch, delete the faulty index and then restore it back with from backup snapshots. Restart the data flow again and rejoice with a green state cluster.
```

Your Elasticssearch cluster will turn into red state if there is at least one primary shards missing, as well as all of its replicas. When a cluster is in red state it means that there is data missing. As a side effect the cluster will not accept any new to store any data in the shards that are faulty. 

When the cluster gets into red state, we need to first inspect which indexes are having problems and then take action to recover the faulty indexes. The problem is often that a particular index has unassigned shards.

```
	GET _cluster/health?level=indices
```

```
{
   "cluster_name": "elasticsearch_cluster01",
   "status": "red",
   "timed_out": false,
   "number_of_nodes": 7,
   "number_of_data_nodes": 7,
   "active_primary_shards": 90,
   "active_shards": 180,
   "relocating_shards": 0,
   "initializing_shards": 0,
   "unassigned_shards": 20
   "indices": {
      "index_1": {
         "status": "green",
         "number_of_shards": 5,
         "number_of_replicas": 1,
         "active_primary_shards": 5,
         "active_shards": 10,
         "relocating_shards": 0,
         "initializing_shards": 0,
         "unassigned_shards": 0
      },
      "index_2": {
         "status": "red", 
         "number_of_shards": 5,
         "number_of_replicas": 1,
         "active_primary_shards": 0,
         "active_shards": 0,
         "relocating_shards": 0,
         "initializing_shards": 0,
         "unassigned_shards": 10 
      },
     ....
 }
```

We see that index_2 is missing 10 shards. That means that the data in those shards were lost, unless it was backup. If you have backups, first stop new data from the index to recover to flow into Elasticsearch. After, delete the problematic index using the Curator or the Elasticsearch cluster API. 

```	
	curl -XDELETE 'localhost:9200/index_2?pretty'
```
```
{
  "acknowledged" : true
}
```

Now, you can go ahead and recover the index with Curator or the restore module API. Once the backup is complete, you can restore the data flowing into Elasticsearch and your Elasticsearch cluster should be up and running in green state. Note that recovering faulty shardss may cause permanent data losses due to out of sync backups - or in the worst case, nonexistent backups. 



