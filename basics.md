# 1. Basic concepts

## What are indexes, mappings and types?

> **TL;DR**
> Indexes are like databases. Each index has one or more mappings. Mappings define how the data is organized within the index. Each mapping defines a data type. The data types have a set of fields consisting of what type of data will be stored in the mapping.

Indexes, mappings and types are abstractions used when talking about data definition in Elasticsearch. Indexes hold a set of documents that can be retrieved, modified, filetered and searched. The data stored in an index may have different formats and definitions. Thus, Elasticsearch offers the possibility to define different data types. The data types are defined by mappings. Each mapping defines a different data type as it describes what fields  each data type needs.

A simple example how to use indexes, mappings and data types to model data in Elasticsearch:

```
Museum [Index]
  [Mapping]
  Paitings [Data type]
    - (string) title 
    - (string) artist 
    - (date) date 
  
  [Mapping]
  Employees [Data type]
   - (string) name
   - (string) gender
   - (integer) salary  

```

The index is the datbase that will store data relative to museums. Each museum has two different data types that must be stored: the collection of paintings and the current museum's employees. Each data types may have different set of information that must be stored. Thus, mappings are used to define what information should each data type store. 

To create the same database index using the Elasticsearch API, we could use the following PUT request:

```
curl -X PUT localhost:9200/museum -d ' 
{
  "mappings": {
    "paitings": { 
      "properties": { 
        "title":    { "type": "string" }, 
        "artist":   { "type": "string" }, 
        "date":     { "type": "date" }  
      }
    },
    "employee": { 
      "properties": { 
        "name":     { "type": "string" }, 
        "gender":   { "type": "string" }, 
        "salary":   { type":  "integer" },
      }
    }
  }
}
'
```


## What are shards?