
# Lens Indexer

## Demo Service

A demo instance of the service is running on https://quiet-retreat-3872.herokuapp.com/. It uses an ElasticSearch instance running on https://7zepvxjw:sqnl0xez3nmesypj@rowan-7428284.us-east-1.bonsai.io.
The index is seeded with the elife corpus found at https://s3.amazonaws.com/elife-cdn/

The index has the following structure

```
"settings": {
  "analysis": {
    "filter": {
      "trigrams_filter": {
        "type": "ngram",
        "min_gram": 3,
        "max_gram": 3
      }
    },
    "analyzer": {
      "html_content": {
        "type":      "custom",
        "tokenizer": "standard",
        "char_filter": [ "html_strip" ],
        "filter":   [ "classic" ]
      }
    }
  }
},
"mappings": {
  "article": {
   "properties": {
     // title and intro are indexed for fuzzy full-text search
     "title": { "type": "string", "index" : "analyzed", "analyzer": "html_content", "search_analyzer": 'snowball', "language": "English" },
     "intro": { "type": "string", "index" : "analyzed", "analyzer": "html_content", "search_analyzer": 'snowball', "language": "English" },
     // authors for exact full-text search (no partial matches)
     "authors": { "type": "string", "index" : "analyzed", "analyzer": "standard" },
     // The rest are facets which are used for strict match queries or filtering only
     "published_on": { "type": "string", "index" : "not_analyzed"},
     "article_type": { "type": "string", "index" : "not_analyzed"},
     "subjects": { "type": "string", "index" : "not_analyzed"},
     "organisms": { "type": "string", "index" : "not_analyzed"}
   }
  },
  "fragment": {
    "_parent": {"type": "article"},
    "properties": {
      "id": { "type": "string", "index" : "not_analyzed" },
      "type": { "type": "string", "index" : "not_analyzed" },
      "content": { "type": "string", "index" : "analyzed", "analyzer": "html_content", "search_analyzer": 'snowball', "language": "English" },
      "position": { "type": "integer", "index": "not_analyzed" }
    }
  }
}
```

> Note: there is one index called `articles` having two types of entities, `article` and `fragment`, where a `fragment` is modelled as a child of an `article`.


It is possible to query the index directly using `curl`, such as:

```
curl https://7zepvxjw:sqnl0xez3nmesypj@rowan-7428284.us-east-1.bonsai.io/articles/fragment/_search -d '
{
 "query": {
    "match": {
      "content": "mice"
    }
  }
}'
```

## Local Installation

### Elastic Search

Linux:

Worked for me out of the box. I.e., just downloaded the binaries from http://www.elasticsearch.org/overview/elkdownloads/.

OSX:

1. Download and Install the JRE *and* *JDK* from http://www.oracle.com/technetwork/java/javase/downloads (Get the `.dmg`)

2. brew update & brew install elasticsearch
