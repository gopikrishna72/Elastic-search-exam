# Indexing Data

# Use the Data Visualizer to upload a text file into Elasticsearch

This is a bit of point and click.

https://www.elastic.co/blog/importing-csv-and-log-data-into-elasticsearch-with-file-data-visualizer

You have to make sure that Machine Learning is switch on (default) or this functionality will be missing.

In elasticsearch.yml: `xpack.ml.enabled=true`



1. Download all USGS magnitude 2.5+ earthquakes from the past month as a CSV.

    https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/2.5_month.csv

2. Import into an index called `usgs`

3. Visualise the data

    3.1 Kibana > Analytics > Maps

    3.2 Add Layer > Documents > Index > `usgs`

    3.3 Calendar Icon > Commonly Used > Last 1 Year

You should now see lots of dots where the earth quakes have happend.




# Define an index that satisfies a given set of requirements

## part 1

:question: Create an index with the following requirements:

- **Name**: accounts-raw
- **Shards**: 1
- **Replicas**: 0

<details>
  <summary>View Solution (click to reveal)</summary>

```json
PUT accounts-raw
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0
  }
}
```

Check details:

```json
GET accounts-raw/_settings

// Output 

{
  "accounts-raw" : {
    "settings" : {
      "index" : {
        "creation_date" : "1618482826116",
        "number_of_shards" : "1",
        "number_of_replicas" : "0",
        "uuid" : "uHiOrOZsQUC7oAyeIzZKUg",
        "version" : {
          "created" : "7020099"
        },
        "provided_name" : "accounts-raw"
      }
    }
  }
}
```
</details>
<hr>

## part 2

:question: Download the following data in bulk insert it into the `accounts-raw` index

https://github.com/elastic/elasticsearch/blob/master/docs/src/test/resources/accounts.json
 
<details>
  <summary>View Solution (click to reveal)</summary>

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/docs-bulk.html

Download the json

```bash
$ wget https://github.com/elastic/elasticsearch/raw/master/docs/src/test/resources/accounts.json
```

Bulk insert the data
```bash
$ curl -u "elastic:Password01" -s -H "Content-Type: application/x-ndjson" -XPUT localhost:9200/accounts-raw/_bulk --data-binary "@accounts.json"; echo
```

Check data was imported

```json
GET accounts-raw/_count

// Output 

{
  "count" : 1000,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  }
}
```

You could also do this in the Development GUI

```json
PUT /accounts-raw/_bulk
## then paste the json contents here
```
</details>
<hr>

# Perform index, create, read, update, and delete operations on the documents of an index

## Index a document

:question: Index the following document to the `accounts-raw` index
```json
{
"account_number":10000,
"balance":1000000,
"firstname":"George",
"lastname":"Cross",
"age":92,
"gender":"M",
"address":"1 Dog Lane",
"employer":"Wheatens",
"email":"george@wheatens.org",
"city":"London",
"state":"UK"
}
```

<details>
  <summary>View Solution (click to reveal)</summary>

```json
PUT accounts-raw/_doc/10000
{"account_number":10000,"balance":1000000,"firstname":"George","lastname":"Cross","age":92,"gender":"M","address":"1 Dog Lane","employer":"Wheatens","email":"george@wheatens.com","city":"London","state":"UK"}
```

or

```json
PUT accounts-raw/_bulk
{"index":{"_id":"10000"}}
{"account_number":10000,"balance":1000000,"firstname":"George","lastname":"Cross","age":92,"gender":"M","address":"1 Dog Lane","employer":"Wheatens","email":"george@wheatens.com","city":"London","state":"UK"}
```
Check the index count was increased
```json
GET accounts-raw/_count

// Output 

{
  "count" : 1001,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  }
}
```

Read back that document that was indexed
```json
GET accounts-raw/_doc/10000

// Output 

{
  "_index" : "accounts-raw",
  "_type" : "_doc",
  "_id" : "10000",
  "_version" : 1,
  "_seq_no" : 1000,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "account_number" : 10000,
    "balance" : 1000000,
    "firstname" : "George",
    "lastname" : "Cross",
    "age" : 92,
    "gender" : "M",
    "address" : "1 Dog Lane",
    "employer" : "Wheatens",
    "email" : "george@wheatens.com",
    "city" : "London",
    "state" : "UK"
  }
}
```
</details>
<hr>

## Create a document

:question: Create the following document to the `accounts-raw` index

```json
"account_number":10001,
"balance":1000001,
"firstname":"Millie",
"lastname":"Cross",
"age":92,
"gender":"F",
"address":"1 Dog Lane",
"employer":"Wheatens",
"email":"millie@wheatens.com",
"city":"London",
"state":"UK"
```

<details>
  <summary>View Solution (click to reveal)</summary>

```json
PUT accounts-raw/_doc/10001
{"account_number":10001,"balance":1000001,"firstname":"Millie","lastname":"Cross","age":92,"gender":"F","address":"1 Dog Lane","employer":"Wheatens","email":"millie@wheatens.com","city":"London","state":"UK"}
```

```json
PUT accounts-raw/_bulk
{"create":{"_id":"10001"}}
{"account_number":10001,"balance":1000001,"firstname":"Millie","lastname":"Cross","age":92,"gender":"F","address":"1 Dog Lane","employer":"Wheatens","email":"millie@wheatens.com","city":"London","state":"UK"}
```

```json
// Output 

{
  "took" : 41,
  "errors" : false,
  "items" : [
    {
      "create" : {
        "_index" : "accounts-raw",
        "_type" : "_doc",
        "_id" : "10001",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 1,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 1001,
        "_primary_term" : 1,
        "status" : 201
      }
    }
  ]
}
```

Check document count has increased

```json
GET accounts-raw/_count

// Output 

{
  "count" : 1002,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  }
}
```

Read back that document just created

```json
GET accounts-raw/_doc/10001

// Output 

{
  "_index" : "accounts-raw",
  "_type" : "_doc",
  "_id" : "10001",
  "_version" : 1,
  "_seq_no" : 1001,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "account_number" : 10001,
    "balance" : 1000001,
    "firstname" : "Millie",
    "lastname" : "Cross",
    "age" : 92,
    "gender" : "F",
    "address" : "1 Dog Lane",
    "employer" : "Wheatens",
    "email" : "millie@wheatens.com",
    "city" : "London",
    "state" : "UK"
  }
}
```
</details>
<hr>

## Update a document

:question: Opps, we made a mistake, account holder `Millie Cross` is not `92` years of `age` but actually `84`.

:question: Update the document in the `accounts-raw` index

<details>
  <summary>View Solution (click to reveal)</summary>

**Hint**: The `update` action requires the extra `"doc"` field to be defined.


```json
POST accounts-raw/_update/10001
{ "doc": { "age":84}}

GET accounts-raw/_doc/10001?filter_path=*.age

// Output 

{
  "_source" : {
    "age" : 84
  }
}
```

```json
PUT accounts-raw/_bulk
{"update":{"_id":"10001"}}
{"doc": {"age":84}}

// Output 

{
  "took" : 43,
  "errors" : false,
  "items" : [
    {
      "update" : {
        "_index" : "accounts-raw",
        "_type" : "_doc",
        "_id" : "10001",
        "_version" : 2,
        "result" : "updated",
        "_shards" : {
          "total" : 1,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 1002,
        "_primary_term" : 1,
        "status" : 200
      }
    }
  ]
}
```

Read back the document to check it has been updated

```json
GET accounts-raw/_doc/10001

// Output 

{
  "_index" : "accounts-raw",
  "_type" : "_doc",
  "_id" : "10001",
  "_version" : 2,
  "_seq_no" : 1002,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "account_number" : 10001,
    "balance" : 1000001,
    "firstname" : "Millie",
    "lastname" : "Cross",
    "age" : 84,
    "gender" : "F",
    "address" : "1 Dog Lane",
    "employer" : "Wheatens",
    "email" : "millie@wheatens.com",
    "city" : "London",
    "state" : "UK"
  }
}
```
</details>
<hr>

## Delete a document

:question: Account holders Millie and George Cross have moved to another bank, remove their accounts.

:question: Delete the following documents from the `accounts-raw` index

- **id**: 10000
- **id**: 10001

<details>
  <summary>View Solution (click to reveal)</summary>

```json
DELETE accounts-raw/_doc/10001
DELETE accounts-raw/_doc/10000?filter_path=result

// Output 

{
  "result" : "deleted"
}
```

```json
PUT accounts-raw/_bulk
{"delete":{"_id":"10000"}}
{"delete":{"_id":"10001"}}

// Output 

{
  "took" : 67,
  "errors" : false,
  "items" : [
    {
      "delete" : {
        "_index" : "accounts-raw",
        "_type" : "_doc",
        "_id" : "10000",
        "_version" : 2,
        "result" : "deleted",
        "_shards" : {
          "total" : 1,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 1003,
        "_primary_term" : 1,
        "status" : 200
      }
    },
    {
      "delete" : {
        "_index" : "accounts-raw",
        "_type" : "_doc",
        "_id" : "10001",
        "_version" : 3,
        "result" : "deleted",
        "_shards" : {
          "total" : 1,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 1004,
        "_primary_term" : 1,
        "status" : 200
      }
    }
  ]
}
```

Get document count from index to check is has reduced.

```json 
GET accounts-raw/_count

// Output 

{
  "count" : 1000,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  }
}
```

</details>
<hr>

# Define and use index aliases

## part 1

:question: Define an index alias for `accounts-raw` called `accounts-all`

<details>
  <summary>View Solution (click to reveal)</summary>

```json
POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "accounts-raw",
        "alias": "accounts-all"
      }
    }
  ]
}
```

Check that the document count matches

```json
GET accounts-all/_count
```
</details>
<hr>

## part 2

:question: Define an index alias for `accounts-raw` called `accounts-male`

:question: Apply a filter to only show the male account owners.

<details>
  <summary>View Solution (click to reveal)</summary>

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/indices-aliases.html

1. check that the feild you want to filter is a keyword

```json
GET accounts-raw/_mapping/field/gender

// Output 

{
  "accounts-raw" : {
    "mappings" : {
      "gender" : {
        "full_name" : "gender",
        "mapping" : {
          "gender" : {
            "type" : "text",
            "fields" : {
              "keyword" : {
                "type" : "keyword",
                "ignore_above" : 256
              }
            }
          }
        }
      }
    }
  }
}
```

2. apply the alias with the filter

**Hint**: you need to use the `.keyword` field here, or you will get zero results.

```json
POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "accounts-raw",
        "alias": "accounts-male",
        "filter": {
          "term": {
            "gender.keyword": "M"
          }
        }
      }
    }
  ]
}
```

3. test

```json
GET accounts-male/_count

// Output 

{
  "count" : 507,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  }
}
```

4. :question: BONUS: Run a query to do the same on `accounts-raw` index

Extra bonus: only print the total hits

```json
POST accounts-raw/_search?filter_path=hits.total.value
{
  "query": {
    "match": {
      "gender": "M"
    }
  }
}

// Output 

{
  "hits" : {
    "total" : {
      "value" : 507
    }
  }
}
```
</details>
<hr>

# Define and use an index template for a given pattern that satisfies a given set of requirements

## Part 1

:question: Create an index template called `accounts-tmpl` with the following requirements:

    "account_number" : integer,
    "balance" : long,
    "firstname" : keyword,
    "lastname" : keyword,
    "age" : integer,
    "gender" : keyword,
    "address" : text,
    "employer" : text / keyword,
    "email" : keyword,
    "city" : keyword,
    "state" : keyword


<details>
  <summary>View Solution (click to reveal)</summary>

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/indices-templates.html

> Templates are only applied at index creation time. Changing a template will have no impact on existing indices. When using the create index API, the settings/mappings defined as part of the create index call will take precedence over any matching settings/mappings defined in the template.

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/mapping.html

> text, keyword, date, long, double, boolean or ip.

> object or nested.

> geo_point, geo_shape, or completion.

### Text vs Keyword

- text - a string broken up into individual words, that are searchable.
- keyword - a fixed string (not broken up) such as email addresses.  Only searchable by their _exact_ value.  Usually used in `filters`.

> Loosely: Text is a bunch of text,  Keyword is a single 'word' used as a 'key'


```json
PUT _template/accounts-tmpl
{
  "index_patterns": ["accounts-*"],
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0
  },
  "mappings": {
    "_source": {
      "enabled": false
    },
    "properties": {
    "account_number" : { "type": "integer" },
    "balance" :  { "type": "integer" },
    "firstname" :  { "type": "keyword" },
    "lastname" :  { "type": "keyword" },
    "age" :  { "type": "integer" },
    "gender" :  { "type": "keyword" },
    "address" :  { "type": "text" },
    "employer" :  { "type": "keyword" },
    "email" :  { "type": "keyword" },
    "city" :  { "type": "keyword" },
    "state" :  { "type": "keyword" }
    }
  }
}
```

:warning: `"_source": { "enabled": false },` will BREAK any re_indexing you may want to do later on.

</details>
<hr>

## Part 2

:question: Import the account data into a new index called `accounts-new`

<details>
  <summary>View Solution (click to reveal)</summary>

```bash
$ curl -u "elastic:Password01" -s -H "Content-Type: application/x-ndjson" -XPUT localhost:9200/accounts-new/_bulk --data-binary "@accounts.json"; echo
```

Check the mappings of this new index

```json
GET accounts-new\_mapping

// Output 

{
  "accounts-new" : {
    "mappings" : {
      "_source" : {
        "enabled" : false
      },
      "properties" : {
        "account_number" : {
          "type" : "integer"
        },
        "address" : {
          "type" : "text"
        },
        "age" : {
          "type" : "integer"
        },
        "balance" : {
          "type" : "integer"
        },
        "city" : {
          "type" : "keyword"
        },
        "email" : {
          "type" : "keyword"
        },
        "employer" : {
          "type" : "keyword"
        },
        "firstname" : {
          "type" : "keyword"
        },
        "gender" : {
          "type" : "keyword"
        },
        "lastname" : {
          "type" : "keyword"
        },
        "state" : {
          "type" : "keyword"
        }
      }
    }
  }
}
```

### Test
How many account holders work for `Zork`

```json
POST accounts-new\_search?filter_path=hits.total.value
{
  "query": {
    "match": {
      "employer": "Zork"
    }
  }
}

// Output 

{
  "hits" : {
    "total" : {
      "value" : 1
    }
  }
}
```
</details>
<hr>

# Define and use a dynamic template that satisfies a given set of requirements

:question: Create a new index with a dynamic mapping `accounts-fullname` where the `firstname` and `lastname` fields are combined to create the `fullname` field.

:anger: This does not work. #FIXME:

<details>
  <summary>View Solution (click to reveal)</summary>

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/dynamic-templates.html

> Dynamic templates allow you to define custom mappings that can be applied to dynamically added fields based on:
> - the datatype detected by Elasticsearch, with match_mapping_type.
> - the name of the field, with match and unmatch or match_pattern.
> - the full dotted path to the field, with path_match and path_unmatch.

```json
PUT accounts_fullname
{
  "mappings": {
    "dynamic_templates": [
      {
        "full_name": {
          "path_match":   "*name",
          "mapping": {
            "type":       "text",
            "copy_to":    "full_name"
          }
        }
      }
    ]
  }
}
```

Okay, to this doesn't work, nor does the example on the website (below)

```json
PUT my_index
{
  "mappings": {
    "dynamic_templates": [
      {
        "full_name": {
          "path_match":   "name.*",
          "path_unmatch": "*.middle",
          "mapping": {
            "type":       "text",
            "copy_to":    "full_name"
          }
        }
      }
    ]
  }
}

PUT my_index/_doc/1
{
  "name": {
    "first":  "John",
    "middle": "Winston",
    "last":   "Lennon"
  }
}

GET my_index/_doc/1

POST my_index/_search
{
  "query": {
    "match_all" :{}
  
  }
}
```
you will find full_name field is not created?!?


#FIXME: find an example that actually works!

</details>
<hr>

# Use the Reindex API and Update By Query API to reindex and/or update documents

## Part 1
:question: Reindex the `accounts-raw` index into `accounts-2021`.

:question: Then reindex `accounts-2021` into `accounts-female` index where only the female account holders are present.

<details>
  <summary>View Solution (click to reveal)</summary>

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/docs-update-by-query.html

> :warning: 
Reindex requires _source to be enabled for all documents in the source index.

:warning: check your templates, to make sure they are not forcing `"_source": { "enabled": false },` as this will break reindexing.

```json
POST _reindex
{
  "source": { "index": "accounts-raw"  },
  "dest":   { "index": "accounts-2021" }
}

GET accounts-2021/_count?filter_path=count

// Output 

{
  "count" : 1000
}
```

reindex into `accounts-female`

```json
POST _reindex
{
  "source": { "index": "accounts-raw",
    "query": {
      "term": {
        "gender.keyword": "F"
      }
    }
  },
  "dest":   { "index": "accounts-female" }
}
```

Check
```json
GET accounts-female/_count?filter_path=count

// Output 

{
  "count" : 493
}
```

Check again
```json
GET /accounts-female/_search?filter_path=*.*.*.gender

// Output 

{
  "hits" : {
    "hits" : [
      {
        "_source" : {
          "gender" : "F"
        }
      },
      {
        "_source" : {
          "gender" : "F"
        }
      },
    ...
```
</details>

## Part 2
:question: Give all female account holders in `accounts-2021` a 25% bonus increase on their balance :)

<details>
  <summary>View Solution (click to reveal)</summary>

Get two example docs
```json
GET /accounts-raw/_search?q=gender:F&size=2
```

Note down those ids and get the balances
```json
GET /accounts-raw/_doc/_mget?filter_path=*.*.balance
{
    "ids" : ["13", "25"]
}

//Output

{
  "docs" : [
    {
      "_source" : {
        "balance" : 32838
      }
    },
    {
      "_source" : {
        "balance" : 40540
      }
    }
  ]
}
```

Update the accounts - take note of the number of `updated` docs
```json
POST accounts-2021/_update_by_query
{
  "script": {
    "source": "ctx._source.balance=ctx._source.balance*1.25",
    "lang": "painless"
  },
  "query": {
    "term": {
      "gender": "F"
    }
  }
}


// Output

{
  "took" : 369,
  "timed_out" : false,
  "total" : 493,
  "updated" : 493,
  "deleted" : 0,
  "batches" : 1,
  "version_conflicts" : 0,
  "noops" : 0,
  "retries" : {
    "bulk" : 0,
    "search" : 0
  },
  "throttled_millis" : 0,
  "requests_per_second" : -1.0,
  "throttled_until_millis" : 0,
  "failures" : [ ]
}
```

Get those same ids again to check the balances have increased
```json
GET /accounts-2021/_doc/_mget?filter_path=*.*.balance
{
    "ids" : ["13", "25"]
}

// Output

{
  "docs" : [
    {
      "_source" : {
        "balance" : 41047.5
      }
    },
    {
      "_source" : {
        "balance" : 50675.0
      }
    }
  ]
}
```

</details>
<hr>

# Define and use an ingest pipeline that satisfies a given set of requirements, including the use of Painless to modify documents

:question: Apply a pipeline called `accounts-ingest` to the data in `accounts-2021` with the following requirements:

- Add a `tag` called `pipeline_ingest` to show that the document was ingested via the pipeline 
- Combine the `firstname` and `lastname ` fields into a new field called `full_name`
- Increase the `balance` of holders that are `female` and `39 years old and over` by 5%.


**Hint**: Use the simulate API to test your code

<details>
  <summary>View Solution (click to reveal)</summary>

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/ingest.html

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/ingest-apis.html

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/accessing-data-in-pipelines.html

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/ingest-processors.html

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/script-processor.html

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/set-processor.html

https://www.elastic.co/guide/en/elasticsearch/reference/7.2/append-processor.html


> :warning: The value of ctx is read-only in if conditions.

> :question: Thus use a script to do more complex work

## Simulate first

```json
POST _ingest/pipeline/_simulate
{
  "pipeline": {
    "processors": [
      {
        "append": {
          "field": "tags",
          "value": ["pipeline_ingest"]
        }
      },
      {
        "set": {
          "tag": "set full_name",
          "field": "full_name",
          "value": "{{firstname}} {{lastname}}"
        }
      },
      {
        "script": {
          "tag": "39s and over female bonus",
          "if": """
            if (ctx.age >= 39) { 
              if (ctx.gender=="F") { 
                return true 
              }
            } 
            return false
          """,
          "lang": "painless",
          "source": """
            ctx.balance = ctx.balance*1.05
          """
        }
      }
    ]
  },
  "docs": [
    {
      "_source": {
        "account_number": 10000,
        "balance": 1000000,
        "firstname": "George",
        "lastname": "Cross",
        "age": 92,
        "gender": "M",
        "address": "1 Dog Lane",
        "employer": "Wheatens",
        "email": "george@wheatens.com",
        "city": "London",
        "state": "UK"
      }
    },
    {
      "_source": {
        "account_number": 10001,
        "balance": 1000001,
        "firstname": "Millie",
        "lastname": "Cross",
        "age": 84,
        "gender": "F",
        "address": "1 Dog Lane",
        "employer": "Wheatens",
        "email": "millie@wheatens.com",
        "city": "London",
        "state": "UK"
      }
    }
  ]
}
```

Here you will get a lot of output, make sure it matches what you expect to see.


Now copy the working pipeline

```json
PUT _ingest/pipeline/accounts-ingest
{
  "description" : "pipeline to account ingest",
  "processors": [
      {
        "append": {
          "field": "tags",
          "value": ["pipeline_ingest"]
        }
      },
      {
        "set": {
          "tag": "set full_name",
          "field": "full_name",
          "value": "{{firstname}} {{lastname}}"
        }
      },
      {
        "script": {
          "tag": "39s and over female bonus",
          "if": """
            if (ctx.age >= 39) { 
              if (ctx.gender=="F") { 
                return true 
              }
            } 
            return false
          """,
          "lang": "painless",
          "source": """
            ctx.balance = ctx.balance*1.05
          """
        }
      }
    ]
}
```

Then reindex the data with that new pipeline

```json
POST accounts-2021/_update_by_query?pipeline=accounts-ingest

// Output - check the `updated` field

{
  "took" : 651,
  "timed_out" : false,
  "total" : 1000,
  "updated" : 1000,
  "deleted" : 0,
  "batches" : 1,
  "version_conflicts" : 0,
  "noops" : 0,
  "retries" : {
    "bulk" : 0,
    "search" : 0
  },
  "throttled_millis" : 0,
  "requests_per_second" : -1.0,
  "throttled_until_millis" : 0,
  "failures" : [ ]
}
```

### Check some docs

Pick a doc id

```json
POST accounts-raw/_search?filter_path=*.*._id
{
  "size": 1, 
  "query": { 
    "bool": { 
      "must": [
        { "match": { "gender.keyword":   "F"}}
      ],
      "filter": [ 
        { "range": { "age": { "gte": "39" }}}
      ]
    }
  }
}

// Output 

{
  "hits" : {
    "hits" : [
      {
        "_id" : "25"
      }
    ]
  }
}
```


Original data
```json
GET accounts-raw/_doc/25?filter_path=*.balance,*.firstname,*.lastname

// Output 

{
  "_source" : {
    "balance" : 40540,
    "firstname" : "Virginia",
    "lastname" : "Ayala",
    "age" : 39
  }
}
```

Updated data
```json
GET accounts-2021/_doc/25?filter_path=*.balance,*.full_name,*.tags

// Output 

{
  "_source" : {
    "tags" : [
      "pipeline_ingest"
    ],
    "full_name" : "Virginia Ayala",
    "balance" : 42567.0
  }
}
```

```
40540 * 1.05 = 42567
```

</details>
<hr>

# Define an Index Lifecycle Management policy for a time-series index

https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started-index-lifecycle-management.html


1 .define an ILM policy to the following requirements
- ILM policy name `timeseries_policy`

Rollover at:
- Max size of 50GB
- 30 days

Delete at:
- 90 days



```json
PUT _ilm/policy/timeseries_policy
{
  "policy": {
    "phases": {
      "hot": {                                
        "actions": {
          "rollover": {
            "max_primary_shard_size": "50GB", 
            "max_age": "30d"
          }
        }
      },
      "delete": {
        "min_age": "90d",                     
        "actions": {
          "delete": {}                        
        }
      }
    }
  }
}
```

## index creation

2. Create a index template with the following requirements

- index pattern `timeseries-*`
- ILM policy `timeseries_policy`
- 1x shard, 0x replicas

```json
PUT _index_template/timeseries_template
{
  "index_patterns": ["timeseries-*"],                 
  "template": {
    "settings": {
      "number_of_shards": 1,
      "number_of_replicas": 0,
      "index.lifecycle.name": "timeseries_policy",      
      "index.lifecycle.rollover_alias": "timeseries"    
    }
  }
}
```

### Bootstrap index
```json
PUT timeseries-000001
{
  "aliases": {
    "timeseries": {
      "is_write_index": true
    }
  }
}
```

### Add a document
write a document to the alias

```json
POST timeseries/_doc
{
  "message": "logged the request",
  "@timestamp": "1591890611"
}
```

## Force a rollover

```json
POST timeseries/_rollover
```

## Check index alias write status


```json
GET _alias/timeseries

// output

{
  "timeseries-000002" : {
    "aliases" : {
      "timeseries" : {
        "is_write_index" : true
      }
    }
  },
  "timeseries-000001" : {
    "aliases" : {
      "timeseries" : {
        "is_write_index" : false
      }
    }
  }
}

```

### View document count

```json
GET _cat/indices/time*?v

// output

health status index             uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   timeseries-000002 bmzwZVAfToqwdbB58JBgQg   1   0          0            0       208b           208b
green  open   timeseries-000001 0Lt9rjDxTAOi2DigXL7cXw   1   0          1            0      4.6kb          4.6kb

```



# Define an index template that creates a new data stream

## What is a datastream
https://www.elastic.co/guide/en/elasticsearch/reference/current/data-streams.html

> A data stream lets you store append-only time series data across multiple indices while giving you a single named resource for requests. _Data streams are well-suited for logs, events, metrics, and other continuously generated data._

:warning: That doesn't need updating - like alarms that open/close or support tickets, etc.  Use an index and alias for that.

> ### Datastream Write index
>
> The most recently created backing index is the data stream’s write index. The stream adds new documents to this index _only_.
>
> You cannot add new documents to other backing indices, even by sending requests directly to the index.
>
> You also cannot perform operations on a write index that may hinder indexing, such as:
>
> * Clone
> * Delete
> * Freeze
> * Shrink
> * Split

> ### There are Append-only
> Data streams are designed for use cases where existing data is rarely, if ever, updated. You cannot send update or deletion requests for existing documents directly to a data stream. Instead, use the update by query and delete by query APIs.
> 
> If needed, you can update or delete documents by submitting requests directly to the document’s backing index.
> 
> If you frequently update or delete existing documents, use an index alias and index template instead of a data stream. You can still use ILM to manage indices for the alias.

https://www.elastic.co/guide/en/elasticsearch/reference/current/set-up-a-data-stream.html

> To set up a data stream, follow these steps:
> 
> Step 1. Create an index lifecycle policy
> 
> Step 2. Create component templates
> 
> Step 3. Create an index template
> 
> Step 4. Create the data stream
> 
> Step 5. Secure the data stream


## 1. Create a ILM policy

```json
PUT _ilm/policy/ds_policy
{
  "policy": {
    "phases": {
      "hot": {                                
        "actions": {
          "rollover": {
            "max_primary_shard_size": "50GB", 
            "max_age": "30d"
          }
        }
      },
      "delete": {
        "min_age": "90d",                     
        "actions": {
          "delete": {}                        
        }
      }
    }
  }
}
```

## 2. Create the component templates

Use a wild card to later extract other data from the message field.
### Mappings
```json
# Creates a component template for mappings
PUT _component_template/my-mappings
{
  "template": {
    "mappings": {
      "properties": {
        "@timestamp": {
          "type": "date",
          "format": "date_optional_time||epoch_millis"
        },
        "message": {
          "type": "wildcard"
        }
      }
    }
  },
  "_meta": {
    "description": "Mappings for @timestamp and message fields",
    "my-custom-meta-field": "More arbitrary metadata"
  }
}
```

### The component template
```json
# Creates a component template for index settings
PUT _component_template/my-settings
{
  "template": {
    "settings": {
      "index.lifecycle.name": "ds-policy"
    }
  },
  "_meta": {
    "description": "Settings for ILM",
    "my-custom-meta-field": "More arbitrary metadata"
  }
}
```

## 3. Create the combined index template based on the components

```json
PUT _index_template/my-component-index-template
{
  "index_patterns": ["my-data-stream*"],
  "data_stream": { },
  "composed_of": [ 
    "my-mappings", 
    "my-settings" 
  ],
  "priority": 500,
  "_meta": {
    "description": "Template for my time series data",
    "my-custom-meta-field": "More arbitrary metadata"
  }
}
```

## 4. Create the data stream
By adding a document to it

```json
POST my-data-stream/_doc
{
  "@timestamp": "2099-05-06T16:21:15.000Z",
  "message": "192.0.2.42 - - [06/May/2099:16:21:15 +0000] \"GET /images/bg.jpg HTTP/1.0\" 200 24736"
}

// output

{
  "_index" : ".ds-my-data-stream-2021.06.27-000001",
  "_type" : "_doc",
  "_id" : "fMjrTnoBw1F50El3ljBq",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
```

### View doc count

```json
GET _cat/indices/.ds-my-data-stream-2021.06.27-000001?v

// output

health status index                                uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   .ds-my-data-stream-2021.06.27-000001 none_iX6QbasrQSlT7eTZg   1   1          1            0        4kb            4kb
```





## 5. Secure a datastream

Usual privileges apply.

Aliases, filtered aliases and all the user privileges can be used (eg. from Kibana admin)
