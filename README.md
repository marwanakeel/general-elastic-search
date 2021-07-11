# ElasticSearch
## Create Policy
```JSON
PUT _ilm/policy/policy-prd-xxx
{
    "policy": {
        "phases": {
            "hot": {
                "actions": {}
            },
            "delete": {
                "min_age": "10d",
                "actions": {
                    "delete": {}
                }
            }
        }
    }
}
```
## Create Template
```JSON
PUT _template/template-prd-xxx
{
  "index_patterns": ["INDEX-*"],
  "order": 0,
  "settings": {
    "number_of_shards": "1",
    "number_of_replicas": "1",
    "index.lifecycle.name": "policy-prd-xxx",
    "index.default_pipeline": "xxxx-pipeline"
    
  }  ,
  "aliases" : {
    "als-prd-xxx" : { }
    }
  }
```

## Create Alias
```JSON
POST /_aliases
{

"actions" : [

{ "add" : { "index" : "INDEX-*", "alias" : "als-prd-xxx" } }

]

}
```
## Reindex
```JSON
POST _reindex
    {
      "source": {
        "index": "SRC-*"
      },
      "dest": {
        "index": "DEST-*"
      }
    }
 POST _reindex 
 { 
 "source": { "index": "foo-2019.10.*" }, 
 "dest": { "index": "foo-2019.10" } 
 }
```
## Modify a field based on a condition
```
POST adsl_siebel_history-2021.03.*/_update_by_query?conflicts=proceed
{
  "query": { 
    "bool": {
        "must": {
            "exists": {
                "field": "ip"
            }
        }
    }
  },
  "script" : {
    "source": "ctx._source.join_date_str = '(' + ctx._source.join_date+')';"
  }
}
```
## Add Data
```
POST my-index-000001/_doc/
{
  "@timestamp": "2099-11-15T13:12:00",
  "message": "GET /search HTTP/1.1 200 1070000",
  "user": {
    "id": "kimchy"
  }
}
```
## Rollover steps
```
PUT _index_template/vodadevsize_template
{
  "index_patterns": ["vodadevsize-*"],      
  "template": {
    "settings": {
      "number_of_shards": 1,
      "number_of_replicas": 1,
      "index.lifecycle.name": "vodadevsize_policy",      
      "index.lifecycle.rollover_alias": "vodarollsize"    
    }
  }
}
PUT _ilm/policy/vodadevsize_policy
{
  "policy": {
    "phases": {
      "hot": {                      
        "actions": {
          "rollover": {
            "max_age": "30d",
            "max_size": "1kb"
          }
        }
      },
      "delete": {
        "min_age": "60d",           
        "actions": {
          "delete": {}              
        }
      }
    }
  }
}
PUT vodadevsize-000001
{
  "aliases": {
    "vodarollsize": {
      "is_write_index": true
    }
  }
}
```

## Create Enrich Policy steps
### This will be the index we will create the enrich policy based upon
```
PUT /users/_doc/1?refresh=wait_for
{
  "email": "mardy.brown@asciidocsmith.com",
  "first_name": "Mardy",
  "last_name": "Brown",
  "city": "New Orleans",
  "county": "Orleans",
  "state": "LA",
  "zip": 70116,
  "web": "mardy.asciidocsmith.com"
}
```
### This is the policy itself where we identify the fields that we want to enrich
```
PUT /_enrich/policy/users-policy
{
  "match": {
    "indices": "users",
    "match_field": "email",
    "enrich_fields": ["first_name", "last_name", "city", "zip", "state"]
  }
}
```
### This must be done whenever the index is updated to update the enrich index 
```
POST /_enrich/policy/users-policy/_execute
```
### The below is to create the ingest pipeline and trying to insert an element
```
PUT /_ingest/pipeline/user_lookup
{
  "description" : "Enriching user details to messages",
  "processors" : [
    {
      "enrich" : {
        "policy_name": "users-policy",
        "field" : "email",
        "target_field": "user",
        "max_matches": "1"
      }
    }
  ]
}

PUT /my-user-00001/_doc/my_id3?pipeline=user_lookup
{
  "email": "mardy3.brown@asciidocsmith.com"
}

GET /my-user-00001/_doc/my_id3

```
elasticsearch filter
The elasticsearch filter copies fields from previous log events in Elasticsearch to current events.

The following config shows a complete example of how this filter might be used. Whenever Logstash receives an "end" event, it uses this Elasticsearch filter to find the matching "start" event based on some operation identifier. Then it copies the @timestamp field from the "start" event into a new field on the "end" event. Finally, using a combination of the date filter and the ruby filter, the code in the example calculates the time duration in hours between the two events.
```
      if [type] == "end" {
         elasticsearch {
            hosts => ["es-server"]
            query => "type:start AND operation:%{[opid]}"
            fields => { "@timestamp" => "started" }
         }
         date {
            match => ["[started]", "ISO8601"]
            target => "[started]"
         }
         ruby {
            code => 'event.set("duration_hrs", (event.get("@timestamp") - event.get("started")) / 3600) rescue nil'
        }
      }
```
