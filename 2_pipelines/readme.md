# Elastic Workshop #2 – Ingest Pipelines

You can find here all Queries in full length for the workshop [Elastic Workshop #2 – Ingest Pipelines](https://cdax.ch/2022/01/30/elastic-workshop-2-ingest-pipelines/)

## Create a pipeline with a set processor

```
PUT _ingest/pipeline/split-city-string-to-array
{
  "description": "Pipeline does various changes on incoming company data",
  "processors": [
   {
     "set": {
       "field": "city_array",
       "copy_from": "city"
       }
     }
  ]
}

PUT companies/_doc/1?pipeline=split-city-string-to-array
{
  "company_name": "Elastic EV", 
  "address": "800 West El Camino Real, Suite 350", 
  "city": "Mountain View, Ca 94040",
  "ticker_symbol": "ESTC", 
  "market_cap": "8B"
}

GET companies/_doc/1

```

## The split processor

```
PUT _ingest/pipeline/split-city-string-to-array
{
  "processors": [
    {
    "set": {
      "field": "city_array",
      "copy_from": "city",
      "override": false
    }
  },
  {
     "split": {
     "field": "city_array",
     "separator": ","
      }
    }
  ]
}

```

## Run a pipeline by _update_by_query

```
POST companies/_update_by_query?pipeline=split-city-string-to-array
{
  "query": {
    "match_all": {}
  }
}

```

## The foreach and the gsub processor

```
PUT _ingest/pipeline/split-city-string-to-array
{
  "processors": [
  {
    "set": {
      "field": "city_array",
      "copy_from": "city",
      "override": false
      }
  },
  {
    "split": {
      "field": "city_array",
      "separator": ","
      }
    },
    {
      "foreach": {
        "field": "city_array",
          "processor": {
            "gsub": {
              "field": "_ingest._value",
              "pattern": "^ ",
              "replacement": ""
           }
         }
       }
     }
   ]
 }
 
```
 
## Run a pipeline by the _simulate API
 
```
POST /_ingest/pipeline/split-city-string-to-array/_simulate
{
  "docs": [
    {
      "_index": "index",
      "_id": "id",
      "_source": {
        "company_name": "Elastic EV", 
        "address": "800 West El Camino Real, Suite 350", 
        "city": "Mountain View, Ca 94040",
        "ticker_symbol": "ESTC", 
        "market_cap": "8B",
        "city_array" : [
          "Mountain View",
          " Ca 94040"
          ]
      }
    }
    ]
}
```

## Setting conditions in processors

```
PUT _ingest/pipeline/split-city-string-to-array
{
  "processors": [
  {
    "set": {
      "field": "city_array",
      "copy_from": "city",
      "override": false
    }
  },
  {
    "split": {
      "if": """
            ctx.city_array instanceof String
            """, 
      "field": "city_array",
      "separator": ","
    }
  },
  {
    "foreach": {
      "field": "city_array",
        "processor": {
          "gsub": {
            "field": "_ingest._value",
            "pattern": "^ ",
            "replacement": ""
         }
       }
     }
   }
  ]
}

POST /_ingest/pipeline/split-city-string-to-array/_simulate
{
  "docs": [
    {
      "_index": "index",
      "_id": "id",
      "_source": {
        "company_name": "Elastic EV", 
        "address": "800 West El Camino Real, Suite 350", 
        "city": "Mountain View, Ca 94040",
        "ticker_symbol": "ESTC", 
        "market_cap": "8B",
        "city_array" : [
          "Mountain View",
          " Ca 94040"
          ]
      }
    }
    ]
}

```

## Handling pipeline failures

```
PUT _ingest/pipeline/split-city-string-to-array
{
  "processors": [
  {
    "set": {
      "field": "city_array",
      "copy_from": "city",
      "override": false
    }
  },
  {
    "split": {
      "ignore_failure": true, 
      "field": "city_array",
      "separator": ","
    }
  },
  {
    "foreach": {
      "field": "city_array",
        "processor": {
          "gsub": {
            "field": "_ingest._value",
            "pattern": "^ ",
            "replacement": ""
         }
       }
     }
   }
  ]
}

POST /_ingest/pipeline/split-city-string-to-array/_simulate
{
  "docs": [
    {
      "_index": "index",
      "_id": "id",
      "_source": {
        "company_name": "Elastic EV", 
        "address": "800 West El Camino Real, Suite 350", 
        "city": "Mountain View, Ca 94040",
        "ticker_symbol": "ESTC", 
        "market_cap": "8B",
        "city_array" : [
          "Mountain View",
          " Ca 94040"
          ]
      }
    }
    ]
}

```

## The script processor

```
PUT _ingest/pipeline/split-city-string-to-array
{
  "processors": [
  {
    "set": {
      "field": "city_array",
      "copy_from": "city",
      "override": false
      }
    },
    {
    "split": {
      "tag": "split", 
      "ignore_failure": true,
      "field": "city_array",
      "separator": ","
      }
    },
    {
      "foreach": {
        "tag": "foreach", 
        "field": "city_array",
          "processor": {
            "gsub": {
              "field": "_ingest._value",
              "pattern": "^ ",
              "replacement": ""
           }
         }
       }
     },
     {
       "script": {
         "tag": "script", 
         "source": """
            ctx['city_name'] = ctx['city_array'].0;
            def split_statezip=ctx['city_array'].1.splitOnToken(' ');
            ctx['state'] = split_statezip[0];
            ctx['zip'] = split_statezip[1];
         """
       }
     }
  ]
}

POST /_ingest/pipeline/split-city-string-to-array/_simulate
{
  "docs": [
    {
      "_index": "index",
      "_id": "id",
      "_source": {
        "company_name": "Elastic EV", 
        "address": "800 West El Camino Real, Suite 350", 
        "city": "Mountain View, Ca 94040",
        "ticker_symbol": "ESTC", 
        "market_cap": "8B",
        "city_array" : [
          "Mountain View",
          " Ca 94040"
          ]
      }
    }
    ]
}

```

## The uppercase processor

```
PUT _ingest/pipeline/split-city-string-to-array
{
  "processors": [
  {
    "set": {
      "field": "city_array",
      "copy_from": "city",
      "override": false
      }
    },
    {
    "split": {
      "tag": "split", 
      "ignore_failure": true, 
      "field": "city_array",
      "separator": ","
      }
    },
    {
      "foreach": {
        "tag": "foreach", 
        "field": "city_array",
          "processor": {
            "gsub": {
              "field": "_ingest._value",
              "pattern": "^ ",
              "replacement": ""
           }
         }
       }
     },
     {
       "script": {
         "tag": "script", 
         "source": """
            ctx['city_name'] = ctx['city_array'].0;
            def split_statezip=ctx['city_array'].1.splitOnToken(' ');
            ctx['state'] = split_statezip[0];
            ctx['zip'] = split_statezip[1];
         """
      }
    },
    {
      "uppercase": {
        "field": "state"
      }
    }
  ]
}

POST /_ingest/pipeline/split-city-string-to-array/_simulate
{
  "docs": [
    {
      "_index": "index",
      "_id": "id",
      "_source": {
        "company_name": "Elastic EV", 
        "address": "800 West El Camino Real, Suite 350", 
        "city": "Mountain View, Ca 94040",
        "ticker_symbol": "ESTC", 
        "market_cap": "8B",
        "city_array" : [
          "Mountain View",
          " Ca 94040"
          ]
      }
    }
    ]
}

```

## The convert processor

```
PUT _ingest/pipeline/split-city-string-to-array
{
  "processors": [
  {
    "set": {
      "field": "city_array",
      "copy_from": "city",
      "override": false
      }
    },
    {
    "split": {
      "tag": "split", 
      "ignore_failure": true, 
      "field": "city_array",
      "separator": ","
      }
    },
    {
      "foreach": {
        "tag": "foreach", 
        "field": "city_array",
          "processor": {
            "gsub": {
              "field": "_ingest._value",
              "pattern": "^ ",
              "replacement": ""
           }
         }
       }
     },
     {
       "script": {
         "tag": "script", 
         "source": """
            ctx['city_name'] = ctx['city_array'].0;
            def split_statezip=ctx['city_array'].1.splitOnToken(' ');
            ctx['state'] = split_statezip[0];
            ctx['zip'] = split_statezip[1];
         """
      }
    },
    {
      "uppercase": {
        "field": "state"
      }
    },
    {
      "convert": {
        "field": "zip",
        "type": "long"
      }
    }
  ]
}

POST /_ingest/pipeline/split-city-string-to-array/_simulate
{
  "docs": [
    {
      "_index": "index",
      "_id": "id",
      "_source": {
        "company_name": "Elastic EV", 
        "address": "800 West El Camino Real, Suite 350", 
        "city": "Mountain View, Ca 94040",
        "ticker_symbol": "ESTC", 
        "market_cap": "8B",
        "city_array" : [
          "Mountain View",
          " Ca 94040"
          ]
      }
    }
    ]
}

```

## The remove processor

```
PUT _ingest/pipeline/split-city-string-to-array
{
  "processors": [
  {
    "set": {
      "field": "city_array",
      "copy_from": "city",
      "override": false
      }
    },
    {
    "split": {
      "tag": "split", 
      "ignore_failure": true, 
      "field": "city_array",
      "separator": ","
      }
    },
    {
      "foreach": {
        "tag": "foreach", 
        "field": "city_array",
          "processor": {
            "gsub": {
              "field": "_ingest._value",
              "pattern": "^ ",
              "replacement": ""
           }
         }
       }
     },
     {
       "script": {
         "tag": "script", 
         "source": """
            ctx['city_name'] = ctx['city_array'].0;
            def split_statezip=ctx['city_array'].1.splitOnToken(' ');
            ctx['state'] = split_statezip[0];
            ctx['zip'] = split_statezip[1];
         """
      }
    },
    {
      "uppercase": {
        "field": "state"
      }
    },
    {
      "convert": {
        "field": "zip",
        "type": "long"
      }
    },
    {
      "remove": {
        "field": ["city_array", "city"]
      }
    }
  ]
}

POST /_ingest/pipeline/split-city-string-to-array/_simulate
{
  "docs": [
    {
      "_index": "index",
      "_id": "id",
      "_source": {
        "company_name": "Elastic EV", 
        "address": "800 West El Camino Real, Suite 350", 
        "city": "Mountain View, Ca 94040",
        "ticker_symbol": "ESTC", 
        "market_cap": "8B",
        "city_array" : [
          "Mountain View",
          " Ca 94040"
          ]
      }
    }
    ]
}

```

# The rename processor

```
PUT _ingest/pipeline/split-city-string-to-array
{
  "processors": [
  {
    "set": {
      "field": "city_array",
      "copy_from": "city",
      "override": false
    }
  },
  {
    "split": {
      "tag": "split", 
      "ignore_failure": true, 
      "field": "city_array",
      "separator": ","
    }
  },
  {
    "foreach": {
      "tag": "foreach", 
      "field": "city_array",
        "processor": {
          "gsub": {
            "field": "_ingest._value",
            "pattern": "^ ",
            "replacement": ""
        }
      }
    }
  },
  {
   "script": {
     "tag": "script",
     "ignore_failure": true, 
     "source": """
          ctx['city_name'] = ctx['city_array'].0;
          def split_statezip=ctx['city_array'].1.splitOnToken(' ');
          ctx['state'] = split_statezip[0];
          ctx['zip'] = split_statezip[1];
        """
    }
  },
  {
    "uppercase": {
    "field": "state"
    }
  },
  {
    "convert": {
      "field": "zip",
      "type": "long"
    }
  },
  {
    "remove": {
      "field": ["city_array", "city"]
    }
  },
  {
    "rename": {
      "field": "city_name",
      "target_field": "city"
    }
  }
  ]
}

```

# The result

```
POST companies/_update_by_query?pipeline=split-city-string-to-array

GET companies/_search
```

