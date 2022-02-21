# Elastic Workshop #4 – Scripting Part 2

You can find here all Queries in full length for the workshop [Elastic Workshop #4 – Scripting Part 2](https://cdax.ch/2022/02/13/elasticsearch-workshop-3-scripting-part-2/)

```
PUT companies/_doc/1
{ "ticker_symbol" : "ESTC",
  "market_cap" : 8000000000,
  "market_cap_string" : "8B",
  "share_price" : 82.5 }

GET companies/_doc/1?_source=company_name,market_cap
```

## pipeline-context

```
PUT _ingest/pipeline/calculate_outstanding
{
  "processors": [
    {
      "script": {
        "lang": "painless", 
        "source": """
  double outstanding = ctx.market_cap / ctx.share_price;
  ctx['outstanding'] = (long)outstanding
  """
      }
    }
  ]
}

PUT _ingest/pipeline/calculate_outstanding
{
  "processors": [
    {
      "script": {
        "lang": "painless", 
        "source": """
  double outstanding = ctx['market_cap'] / ctx['share_price'];
  ctx['outstanding'] = (long)outstanding
  """
      }
    }
  ]
}
```

## _update-context

```
POST companies/_update/1
{
  "script" : {
    "lang": "painless",
    "source": """
  double outstanding = ctx._source.market_cap / ctx._source.share_price;
  ctx._source.outstanding = (long)outstanding  
    """
  }
}

GET companies/_doc/1
```

## update_by_query-context

```
POST companies/_update_by_query
{
  "query": {
    "match_all": {}
  }, 
  "script" : {
    "source": """
  double outstanding = ctx._source.market_cap / ctx._source.share_price;
  ctx._source.outstanding_by_query = (long)outstanding      
    """
  }
}

GET companies/_doc/1
```

## reindex-context

```
POST _reindex
{
  "source": {
    "index": "companies"
  },
  "dest": {
    "index": "companies_new"
  },
  "script": {
    "source": """
  double outstanding = ctx._source.market_cap / ctx._source.share_price;
  ctx._source.outstanding_reindexed = (long)outstanding 
    """
  }
}

GET companies_new/_doc/1
```

## runtime_mappings-context

```
GET companies/_search
{
  "query": {
    "match": {
      "ticker_symbol.keyword": "ESTC"
    }
  }, 
  "runtime_mappings": {
    "outstanding": {
      "type": "long",
      "script": {
        "lang": "painless", 
        "source": """
        long result;
        double outstanding = doc.market_cap.value / doc['share_price'].value;
        result = (long)outstanding; 
        emit(result);
        """
      }
    }
  },
  "fields": ["outstanding"]
}
```


## scripted-fields context

```
GET companies/_search
{
  "query": {
    "match_all": {}
  },
  "script_fields": {
    "free_float": {
      "script": {
        "source": """
          double outstanding = doc.market_cap.value / doc['share_price'].value;
          outstanding = (long)outstanding;
        return (outstanding)
        """
      }
    }
  }
}
```
