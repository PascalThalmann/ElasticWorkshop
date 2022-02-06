# Elastic Workshop #3 – Scripting Part 1

You can find here all Queries in full length for the workshop [Elastic Workshop #3 – Scripting Part 1](https://cdax.ch/2022/02/05/elastic-workshop-3-scripting-part-1/)

## Hello World

```
POST /_scripts/painless/_execute
{
  "script": {
    "source": "return('Hello World');"
} }

POST /_scripts/painless/_execute
{
  "script": {
    "source": """
    return(params.phrase)
    """,
    "params": {
      "phrase": "Hello World"
    }
} }

POST /_scripts/painless/_execute
{
  "script": {
    "source": """
    // This is a oneline comment
    return(params.phrase)
    /* This is a
    multiline comment */
    """,
    "params": {
      "phrase": "Hello World"
    }
} }
```

## Working with data

```
PUT persons/_doc/1
{ "name": "John",
  "sur_name": "Smith",
  "year_of_birth": 1925 }

GET persons/_search
{
  "runtime_mappings": {
    "age": {
      "type": "long",
      "script": {
        "source": """
        long age = params.today - doc['year_of_birth'].value;
        emit(age)
        """,
        "params": { "today": 2022 }
      }
    }
  },
  "fields": [ "age" ]
}
```

## Storing Scripts

```
PUT _ingest/pipeline/calc_age_pipeline
{
  "processors": [
    {
      "script": {
        "source": """
          ctx['age'] = params.today - ctx['year_of_birth'];
        """,
        "params": { "today": 2022 }
      }
    }
  ]
}

POST persons/_update_by_query?pipeline=calc_age_pipeline
{
  "query": {
    "match_all": {}
  }
}

GET persons/_search

PUT _scripts/calc_age_script
{
  "script": {
    "lang": "painless", 
    "source": """
      ctx._source['age'] = params['today'] - ctx._source['year_of_birth'];
    """
  }
}
```

## Calling scripts with the ____update_by_query API

```
POST persons/_update_by_query
{
  "script": {
    "id": "calc_age_script",
    "params": { "today": 2022 }
  }, 
  "query": {
    "match_all": {}
  }
}

GET persons/_doc/1
```

## Calling scripts with the _update API

```
POST persons/_update/1
{
  "script": {
    "id": "calc_age_script",
    "params": { "today": 2022 }
  }
}

GET persons/_doc/1
```

## Calling Scripts with the _reindex API

```
POST _reindex
{
  "source": { "index": "persons" },
  "dest": { "index": "persons_with_age" },
  "script": { "id": "calc_age_script", "params": { "today": 1995 } }
}

GET persons_with_age/_search

```
## Calling scripts with the _search API

```
PUT _scripts/calc_age_script
{
  "script": {
    "lang": "painless", 
    "source": """
    params['today'] - doc['year_of_birth'].value;
    """
  }
}

GET persons/_search
{
  "script_fields": {
    "age": {
      "script": {"id": "calc_age_script", "params": { "today": 2022 } }
    }
  }
}
```

## Calling scripts with a search-template

```
PUT _scripts/calc_age_template
{
  "script": {
    "lang": "mustache",
    "source": {
      "runtime_mappings": {
        "age": {
          "type": "long",
          "script": { "source": 
            """ 
             long age = {{act_year}} - doc['year_of_birth'].value;
             emit(age)
            """
          }
        }
      },
      "fields": [ "age" ]
    }
  },
  "params": { "act_year" : "today"}
}

GET persons/_search/template
{
  "source": "fields",
  "id": "calc_age_template",
  "params": {
    "act_year": 2022
  }
}
```
