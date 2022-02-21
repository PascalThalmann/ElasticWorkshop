# Elastic Workshop #5 – Scripting Part 3

You can find here all Queries in full length for the workshop [Elastic Workshop #5 – Scripting Part 3](https://cdax.ch/2022/02/19/elasticsearch-workshop-5-scripting-part-3/)

## data

```
PUT companies/_doc/1
{ "ticker_symbol" : "ESTC",
  "market_cap" : 8000000000,
  "market_cap_string" : "8B",
  "share_price" : 82.5 }

GET companies/_search
```

## Primitive datatypes and their reference objects

```
GET companies/_search
{
  "runtime_mappings": {
    "test_types": {
      "type": "boolean",
      "script": {
        "source": """
        boolean a;
        long i = doc.market_cap.value;
        if (i > 1000){
          a = true
        }else{
          a = false
        }
        emit(a)
        """
      }
    }
  },
  "fields": [
    "test_types"
  ]
}

GET companies/_search
{
  "runtime_mappings": {
    "test_types": {
      "type": "boolean",
      "script": {
        "source": """
        def a;
        def i = doc.market_cap.value;
        if (i > 1000){
          a = true
        }else{
          a = false
        }
        emit(a)
        """
      }
    }
  },
  "fields": [
    "test_types"
  ]
}
```

## non-static methods

```
GET companies/_search
{
  "runtime_mappings": {
    "test_types": {
      "type": "double",
      "script": {
        "source": """
        // double doubleValue()
        double i = (double)doc.market_cap.value;
        double a = i.doubleValue();
        emit(a)
        """
  } } },
  "fields": [
    "test_types"
  ] }
  
GET companies/_search
{
  "runtime_mappings": {
    "test_types": {
      "type": "boolean",
      "script": {
        "source": """
        // boolean equals(Object)
        boolean i = true; 
        boolean j = false;
        boolean a = i.equals(j);
        emit(a)
        """
  } } },
  "fields": [
    "test_types"
  ] }

GET companies/_search
{
  "runtime_mappings": {
    "test_types": {
      "type": "keyword",
      "script": {
        "source": """
        // null toString()
        long i = doc.market_cap.value;
        String a = i.toString();
        emit(a)
        """
  } } },
  "fields": [
    "test_types"
  ] }
  
GET companies/_search
{
  "runtime_mappings": {
    "test_types": {
      "type": "long",
      "script": {
        "source": """
        // int compareTo(Long)
        long i = doc.market_cap.value;
        long j = 2000;
        int a = i.compareTo(j);
        emit(a)
        """
  } } },
  "fields": [
    "test_types"
  ] }
```

## static methods

```
GET companies/_search
{
  "runtime_mappings": {
    "test_types": {
      "type": "long",
      "script": {
        "source": """
        // static long max(long, long)
        long i = doc.market_cap.value;
        long j = 4000;
        long a = Long.max(i,j);
        emit(a)
        """
  } } },
  "fields": [
    "test_types"
  ] }
  
GET companies/_search
{
  "runtime_mappings": {
    "test_types": {
      "type": "long",
      "script": {
        "source": """
        // static long divideUnsigned(long, long)
        long i = doc.market_cap.value;
        long j = 4000;
        long a = Long.divideUnsigned(i,j);
        emit(a)
        """
  } } },
  "fields": [
    "test_types"
  ] }

GET companies/_search
{
  "runtime_mappings": {
    "test_types": {
      "type": "long",
      "script": {
        "source": """
        // static int compare(long, long)
        Long i;
        i = doc.market_cap.value;
        long j = 4000;
        long a = Long.compare(i,j);
        emit(a)
        """
  } } },
  "fields": [
    "test_types"
  ] }

GET companies/_search
{
  "runtime_mappings": {
    "test_types": {
      "type": "long",
      "script": {
        "source": """
        // static int numberOfTrailingZeros(long)
        Long i;
        i = doc.market_cap.value;
        long a = Long.numberOfTrailingZeros(i);
        emit(a)
        """
  } } },
  "fields": [
    "test_types"
  ] }
```

## Arrays

```
GET companies/_search
{
  "runtime_mappings": {
    "test_types": {
      "type": "long",
      "script": {
        "source": """
        int[] a = new int[] {1, 2, 3};
        a[2] = 5;
        emit(a[-1])
        """
      }
    }
  },
  "fields": [
    "test_types"
  ]
}

GET companies/_search
{
  "runtime_mappings": {
    "test_types": {
      "type": "long",
      "script": {
        "source": """
        int[][] i = new int[2][5];
        i[0][0] = 12;
        emit(i[0][0])
        """
      }
    }
  },
  "fields": [
    "test_types"
  ]
}

int[][][] ia3 = new int[2][3]

GET companies/_search
{
  "runtime_mappings": {
    "test_types": {
      "type": "long",
      "script": {
        "source": """
        int[] a = new int[] {1, 2, 3};
        int[] b = new int[4];
        int c = a.length;
        int i = 0;
        while (i < c-1){
          i++;
          b[i] = a[i];
        }
        b[c] = 5;
        emit(b[-1])
        """
      }
    }
  },
  "fields": [
    "test_types"
  ]
}
```

## ArrayLists

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

## HashMaps

```
GET companies/_search
{
  "runtime_mappings": {
    "test_types": {
      "type": "long",
      "script": {
        "source": """
        Map mp = new HashMap();
        mp.put('one', 1);
        mp.put('two', 2);
        emit(mp.get('two'))
        """
      }
    }
  },
  "fields": [
    "test_types"
  ]
}
```

## String data type

```
GET companies/_search
{
  "runtime_mappings": {
    "test_types": {
      "type": "keyword",
      "script": {
        "source": """
        String mc = doc['market_cap_string.keyword'].value;
        String ticker = doc['ticker_symbol.keyword'].value;
        String phrase = " is worth ";
        emit(ticker + phrase + mc)
        """
      }
    }
  },
  "fields": [
    "test_types"
  ]
}

GET companies/_search
{
  "runtime_mappings": {
    "test_types": {
      "type": "keyword",
      "script": {
        "source": """
        // int length()
        // null substring(int)
        String mc = doc['market_cap_string.keyword'].value;
        int len = mc.length();
        def mc_clean = mc.substring(0, len - 1);
        emit(mc_clean)
        """
      }
    }
  },
  "fields": [
    "test_types"
  ]
}
```

## casting

```
GET companies/_search
{
  "runtime_mappings": {
    "test_types": {
      "type": "double",
      "script": {
        "source": """
        long mc = doc.market_cap.value;
        double mc_double = (double)mc;
        emit(mc_double)
        """
      }
    }
  },
  "fields": [
    "test_types"
  ]
}

```
