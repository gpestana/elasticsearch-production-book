# 2. Data


// note: in the beginning, include a short introduction to the chapter. Provide an overview about the things talked about in the chapter and discuss why they are relevant.



* (refer sharding data from previous chapter) Though, sharding data is not enough in case something catastrophic happens to the cluster and some data - or even all data - is damaged or deleted. (e.g.) When for some reason data is deleted or the cluster enters in a faulty mode, it is helpful to have data backups and a process to restore the cluster state and data into a new cluster.




### How to backup Elasticsearch data?

**TL;DR**
```
Elasticsearch includes the snapshot and restore modules for creating and managing backups. With those modules, you can create snapshots of individual or a set of indexes and store them in several different places, such as shared filesystem, Amazon S3 and others. The backups can be created and maintained through the API or through official tools such as Curator.
```


// ..

There are some officially supported tools such as Elasticsearch curator that wrap the snapshot and restore modules API in easy to use command line tools.