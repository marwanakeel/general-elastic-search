# elasticSearch
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
