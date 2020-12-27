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
    "index.lifecycle.name": "policy-prd-xxx"
    
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
