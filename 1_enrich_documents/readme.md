# Elastic Workshop #1 – Enrich documents

You can find here all Queries in full length for the workshop [Elastic Workshop #1 – Enrich documents](https://cdax.ch/2022/01/28/elastic-workshop-1-enrich-documents/)

## Create documents that will be later enriched

```
POST _bulk
{ "index" : { "_index" : "companies", "_id" : "1" } }
{ "company_name": "Elastic EV", "address": "800 West El Camino Real, Suite 350", "city": "Mountain View, CA 94040","ticker_symbol": "ESTC", "market_cap": "8B"}
{ "create" : { "_index" : "companies", "_id" : "2" } }
{ "company_name": "Mongo DB, Inc","address": "1633 Broadway, 38th Floor","city": "New York, NY 10019","ticker_symbol": "MDB","market_cap": "23B"}
{ "create" : { "_index" : "companies", "_id" : "3" } }
{ "company_name": "Splunk Inc","address": "270 Brannan Street","city": "San Francisco, CA 94107", "ticker_symbol": "SPLK","market_cap": "18B"}



GET companies/_search

PUT stocks/_doc/1
{
  "ticker": "ESTC",
  "last_trade": 82.5
}


PUT stocks/_doc/2
{
  "ticker": "MDB",
  "last_trade": 365
}
```

## Add an enrichment policy

```
PUT /_enrich/policy/add_company_data_policy
{
  "match": {
    "indices": "companies",
    "match_field": "ticker_symbol",
    "enrich_fields": ["company_name", "address", "city", "market_cap"]
  }
}


PUT /_enrich/policy/add_company_data_policy/_execute


GET .enrich-add_company_data_policy
```

## Add a pipeline that uses the enrichment policy

```
PUT _ingest/pipeline/enrich_stock_data
{
  "processors": [
    {
      "set": {
        "field": "enriched",
        "value": 1
      }
    },
    {
      "enrich": {
        "policy_name": "add_company_data_policy",
        "field": "ticker",
        "target_field": "company"
      }
    }
  ]
}
```


## Enrich existing documents

```
POST _reindex/
{
  "source": {
    "index": "stocks"
  },
  "dest": {
    "index": "full_stock_data",
    "pipeline": "enrich_stock_data"
  }
}

GET full_stock_data/_search
```

## Enrich incoming data

```
PUT /full_stock_data/_doc/3?pipeline=enrich_stock_data
{
  "ticker": "SPLK",
  "last_trade": 113
}

GET full_stock_data/_doc/3

PUT full_stock_data/_settings
{
  "index.default_pipeline": "enrich_stock_data"
}

POST companies/_doc/4
{
  "company_name": "Datadog, Inc.",
  "address": "620 8th Avenue, 45th Floor",
  "city": "New York, NY 10018",
  "ticker_symbol": "DDOG",
  "market_cap": "40B"
}

PUT full_stock_data/_doc/4
{
  "ticker": "DDOG",
  "last_trade": 113
}

GET full_stock_data/_doc/4
```


## Fix documents that could not be enriched by the last run

```
PUT /_enrich/policy/add_company_data_policy/_execute

GET .enrich-add_company_data_policy

POST full_stock_data/_update_by_query
{
  "query": {
    "bool": {
      "must_not": [
        {
          "exists": {
            "field": "company"
          }
        }
      ]
    }
  }
}
```


GET full_stock_data/_doc/4

## Conclusion

```
GET _cat/indices/.enrich-add_company_data_policy*,companies,full_stock_data?s=i&v&h=idx,storeSize
```

