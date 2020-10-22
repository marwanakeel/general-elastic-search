# elasticSearch
## Create Policy
```JSON
POST _ilm/policy/policy-prd-xxx

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
