# Elastic Workshop #5 – Scripting Part 3

You can find here all Queries in full length for the workshop [Elastic Workshop #5 – Scripting Part 3](https://cdax.ch/2022/02/19/elasticsearch-workshop-5-scripting-part-3/)


## preparation

```
GET _cluster/settings?include_defaults&filter_path=defaults.script.painless.regex
```

/etc/elasticsearch/elasticsearch.yml:
```
script.painless.regex.enabled: true
```

## data

```
PUT companies/_doc/1
{ "ticker_symbol" : "ESTC",
  "market_cap" : "8B",
  "share_price" : 85.41
}
```

## Regexes

```
GET companies/_search
{
  "script_fields": {
    "market_cap_factor": {
      "script": {
        "source": """
        long market_cap_factor;
        if ( doc['market_cap.keyword'].value =~ /B$/){
          market_cap_factor = 1000000000
        }
        return(market_cap_factor);
        """
      }
    }
  }
}
```

## matcher class

```
GET companies/_search
{
  "query": {
    "match_all": {}
  },
  "script_fields": {
    "market_cap_in_billions": {
      "script": {
        "source": """
        String market_cap_string = doc['market_cap.keyword'].value;
        Pattern p = /([A-Za-z]+)$/;
        //def result = p.matcher(market_cap_string).matches();
        //def result = p.matcher(market_cap_string).group(1);
        def result = p.matcher(market_cap_string).replaceAll('');
        return(result)
        """
      }
    }
  }
}

GET companies/_search
{
  "query": {
    "match_all": {}
  },
  "script_fields": {
    "market_cap_in_billions": {
      "script": {
        "source": """
        String market_cap_string = doc['market_cap.keyword'].value;
        Pattern p = /([0-9]+)([A-Za-z]+)$/;
        //def result = p.matcher(market_cap_string).matches();
        //def result = p.matcher(market_cap_string).group(1);
        def market_cap = p.matcher(market_cap_string).replaceAll('$1');
        return(market_cap)
        """
      }
    }
  }
}

```

## examples

```
GET companies/_search
{
  "runtime_mappings": {
    "market_cap": {
      "type": "long",
      "script": {
        "source": """
        long market_cap_long;
        String mc_string = doc['market_cap.keyword'].value;
        String mc_long_as_string = /[BM]$/.matcher(mc_string).replaceAll('');
        long mc_long = (long) Integer.parseInt(mc_long_as_string);
        if ( mc_string =~ /B/){
          market_cap_long = mc_long * 1000000000
        }
        emit(market_cap_long);
        """
      }
    }
  },
  "fields": ["market_cap" ]
}

```

### Script for scripted_fields

```
PUT _scripts/calculate_market_cap_long
{
  "script": {
    "lang": "painless",
    "source": """
        long market_cap;
        Pattern p = /[BM]$/;
        String mc_string = doc['market_cap.keyword'].value;
        String mc_long_as_string = p.matcher(mc_string).replaceAll('');
        long mc_long = (long) Integer.parseInt(mc_long_as_string);
        if ( mc_string =~ /B$/){
          market_cap = mc_long * 1000000000
        }
        return(market_cap)
    """
  }
}

GET companies/_search
{
  "script_fields": {
    "market_cap": {
      "script": {
        "id": "calculate_market_cap_long"
      }
    }
  }
}

```

### Script for update, update_by_api and reindex APIs

```
PUT _scripts/calculate_market_cap_long
{
  "script": {
    "lang": "painless",
    "source": """
        long market_cap;
        Pattern p = /[BM]$/;
        String mc_string = ctx._source['market_cap'];
        String mc_long_as_string = p.matcher(mc_string).replaceAll('');
        long mc_long = (long) Integer.parseInt(mc_long_as_string);
        if ( mc_string =~ /B$/){
          market_cap = mc_long * 1000000000
        }
        ctx._source['market_cap_long'] = market_cap;
    """
  }
}


POST _reindex
{
  "source": {
    "index": "companies"
  },
  "dest": {
    "index": "companies_2"
  },
  "script": {
    "id": "calculate_market_cap_long"
  }
}

POST companies_2/_update_by_query
{
  "script": {
    "id": "calculate_market_cap_long"
  }
}

```

### Script for pipeline

```
PUT _ingest/pipeline/calculate_market_cap_long_pipeline
{
  "processors": [
    {
      "script": {
        "lang": "painless",
        "source": """
          long market_cap;
          Pattern p = /[BM]$/;
          String mc_string = ctx['market_cap'];
          String mc_long_as_string = p.matcher(mc_string).replaceAll('');
          long mc_long = (long) Integer.parseInt(mc_long_as_string);
          if ( mc_string =~ /B$/){
            market_cap = mc_long * 1000000000
          }
          ctx['market_cap_long'] = market_cap;
        """
      }
    }
  ]
}

PUT companies_3/_doc/1?pipeline=calculate_market_cap_long_pipeline
{ "ticker_symbol" : "ESTC",
  "market_cap" : "8B",
  "share_price" : 85.41
}

```

## String contains method

```
GET companies/_search
{
  "query": {
    "match_all": {}
  },
  "script_fields": {
    "market_cap": {
      "script": {
        "source": """
        long market_cap, mc_long;
        String mc_long_as_string;
        String market_cap_string = doc['market_cap.keyword'].value;
        if (market_cap_string.contains("B")){
          mc_long_as_string = market_cap_string.replace('B', '');
          mc_long = (long) Integer.parseInt(mc_long_as_string);
          market_cap = mc_long * 1000000000
        }
        return(market_cap)
        """
      }
    }
  }
}

GET companies/_search
{
  "query": {
    "match_all": {}
  },
  "script_fields": {
    "market_cap": {
      "script": {
        "source": """
        long market_cap, mc_long;
        String mc_long_as_string;
        String market_cap_string = doc['market_cap.keyword'].value;
        if (market_cap_string.endsWith("B")){
          mc_long_as_string = market_cap_string.replace('B', '');
          mc_long = (long) Integer.parseInt(mc_long_as_string);
          market_cap = mc_long * 1000000000
        }
        return(market_cap)
        """
      }
    }
  }
}

```

## GROK patterns

```
GET companies/_search
{
  "query": {
    "match_all": {}
  },
  "runtime_mappings": {
    "free_float": {
      "type": "long",
      "script": {
        "lang": "painless",
        "source": """
        long market_cap, mc_long;
        String mc_long_as_string=grok('%{NUMBER:markcap}').extract(doc['market_cap.keyword'].value).markcap;
        String factor_as_string=grok('(?<fact>[A-Z])').extract(doc['market_cap.keyword'].value).fact;
        if (factor_as_string == 'B'){
          mc_long = (long) Integer.parseInt(mc_long_as_string);
          market_cap = mc_long * 1000000000
        }
        emit(market_cap);
        """
      }
    }
  },
  "fields": ["free_float"]
}
```

