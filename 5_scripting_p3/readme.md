# Elastic Workshop #5 – Scripting Part 3

You can find here all Queries in full length for the workshop [Elastic Workshop #5 – Scripting Part 3]()

```
PUT companies/_doc/1
{ "ticker_symbol" : "ESTC",
  "market_cap" : 8000000000,
  "market_cap_string" : "8B",
  "share_price" : 82.5 }

```

## types

```
GET companies/_search
{
  "runtime_mappings": {
    "test_types": {
      "type": "long",
      "script": {
        "source": """
        long mc = doc.market_cap.value;
        List al = new ArrayList();
        al.add(mc);
        al.add(10);
        emit(al.get(0) + al.get(1))
        """
      }
    }
  },
  "fields": [
    "test_types"
  ]
}
```
