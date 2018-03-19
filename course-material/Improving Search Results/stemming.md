# Stemming

## Creating a test index

```
PUT /stemming_test
{
  "settings": {
    "analysis": {
      "filter": {
        "synonym_test": {
          "type": "synonym",
          "synonyms": [
            "firm => company",
            "love, enjoy"
          ]
        },
        "stemmer_test" : {
          "type" : "stemmer",
          "name" : "english"
        }
      },
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "synonym_test",
            "stemmer_test"
          ]
        }
      }
    }
  },
  "mappings": {
    "default": {
      "properties": {
        "description": {
          "type": "text",
          "analyzer": "my_analyzer"
        }
      }
    }
  }
}
```

## Adding a test document

```
POST /stemming_test/default/1
{
  "description": "I love working for my firm!"
}
```

## Matching the document with the base word (`work`)

```
GET /stemming_test/default/_search
{
  "query": {
    "match": {
      "description": "enjoy work"
    }
  }
}
```

## The query is stemmed, so the document still matches

```
GET /stemming_test/default/_search
{
  "query": {
    "match": {
      "description": "love working"
    }
  }
}
```

## Synonyms and stemmed words are still highlighted

```
GET /stemming_test/default/_search
{
  "query": {
    "match": {
      "description": "enjoy work"
    }
  },
  "highlight": {
    "fields": {
      "description": {}
    }
  }
}
```