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

#### Is there any official tool maintaining Elasticsearch backups?

Yes there is. Elastic Curator (https://github.com/elastic/curator) is a neat tool that helps maintaining Elasticsearch clusters' indexes. It includes features that help to create and maintain backups such as taking indexes snapshots, deleting indexes and snapshots.

Curator provides a CLI interface and HTTP API..