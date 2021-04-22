# elasticsearch-payload-score
Score documents with payload in elasticsearch 7.12.0

## Releases
2021-04-22 `7.12.0` targets elasticsearch 7.12.0

## Overview


## Scoring
```java
int freq = postings.freq();
float sum_payload = 0.0f;
for(int i = 0; i < freq; i ++)
{
    postings.nextPosition();
    BytesRef payload = postings.getPayload();
    if(payload != null) {
        sum_payload += ByteBuffer.wrap(payload.bytes, payload.offset, payload.length)
                .order(ByteOrder.BIG_ENDIAN).getFloat();
    }
}

return sum_payload;
```

## Plugin installation
Target elasticsearch version is 7.12.0 and java 1.8

## Example

**mapping & setting**

```javascript 1.8
{
  "settings": {
    "index": {
      "number_of_shards": 2,
      "number_of_replicas": 0
    },
    "analysis": {
      "analyzer": {
        "payload_analyzer": { 
          "type": "custom",
          "tokenizer": "payload_tokenizer",
          "filter": [
            "payload_filter"
          ]
        }
      },
      "tokenizer": {
        "payload_tokenizer": {
          "type": "whitespace",
          "max_token_length": 64
        }
      },
      "filter": {
        "payload_filter": {
          "type": "delimited_payload",
          "encoding": "float"
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "key": {
        "type": "text",
        "analyzer" : "payload_analyzer",
        "term_vector": "with_positions_offsets_payloads",
        "store" : true
      }
    }
  }
}
```

**test collection**
```javascript
{ "index" : { "_index" : "payload-test", "_type" : "_doc", "_id" : "1" } }
{ "key" : ["yellow|3 blue|1.1 red|1" ] }
{ "index" : { "_index" : "payload-test", "_type" : "_doc", "_id" : "2" } }
{ "key" : ["yellow|2 yellow|2.5 blue|1 red|5"] }
{ "index" : { "_index" : "payload-test", "_type" : "_doc", "_id" : "3" } }
{ "key" : ["yellow|10 blue|2 red|4.3" ] }
{ "index" : { "_index" : "payload-test", "_type" : "_doc", "_id" : "4" } }
{ "key" : ["dark_yellow|10 blue|3 red|4.2" ] }
{ "index" : { "_index" : "payload-test", "_type" : "_doc", "_id" : "5" } }
{ "key" : ["yellow|102020.95 blue red|1" ] }
```

**query**
```bash
curl -H 'Content-Type: application/json' -X POST 'localhost:9200/payload-test/_search?pretty' -d '
{
  "query": {
    "function_score": {
      "query": {
        "match": {
          "key": "yellow"
        }
      },
      "functions": [
        {
          "script_score": {
            "script": {
                "source": "payload_score",
                "lang" : "irgroup",
                "params": {
                    "field": "key",
                    "term": "yellow"
                }
            }
          }
        }
      ]
    }
  }
}
'
```
