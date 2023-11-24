# vespa-fuzzy-bolding-problem

A minimalistic example to show that fuzzy matching breaks bolding.

## Reproducing

```shell
docker run --detach \
  --rm \
  --name vespa \
  --hostname vespa-container \
  --publish 8080:8080 --publish 19071:19071 \
  vespaengine/vespa:8.263.7
  
vespa version
# Vespa CLI version 8.263.7 compiled with go1.21.4 on darwin/arm64

vespa deploy --wait 60 vap

vespa status --wait 60

vespa feed data.json
```

Query
```shell
vespa query '
select * 
from doc 
where {defaultIndex: "body"}userInput(@query)
' \
query="vespa" \
presentation.bolding=true
```
returns
```json
{
  "root": {
    "id": "toplevel",
    "relevance": 1.0,
    "fields": {
      "totalCount": 1
    },
    "coverage": {
      "coverage": 100,
      "documents": 1,
      "full": true,
      "nodes": 1,
      "results": 1,
      "resultsFull": 1
    },
    "children": [
      {
        "id": "id:doc:doc::demo",
        "relevance": 0.3682531210656552,
        "source": "content",
        "fields": {
          "sddocname": "doc",
          "body": "<hi>Vespa</hi> is a fully featured search engine and vector database.",
          "documentid": "id:doc:doc::demo",
          "author": "Vespa"
        }
      }
    ]
  }
}
```

See that matching term in the body is bolded with `<h1>` tags.

When a fuzzy query clause is added the bolding is no longer there:

```shell
vespa query '
select * 
from doc 
where {defaultIndex: "body"}userInput(@query) OR author contains fuzzy(@query)
' \
query="vespa" \
presentation.bolding=true
```
The response:
```json
{
  "root": {
    "id": "toplevel",
    "relevance": 1.0,
    "fields": {
      "totalCount": 1
    },
    "coverage": {
      "coverage": 100,
      "documents": 1,
      "full": true,
      "nodes": 1,
      "results": 1,
      "resultsFull": 1
    },
    "children": [
      {
        "id": "id:doc:doc::demo",
        "relevance": 0.36999604045563345,
        "source": "content",
        "fields": {
          "sddocname": "doc",
          "body": "Vespa is a fully featured search engine and vector database.",
          "documentid": "id:doc:doc::demo",
          "author": "Vespa"
        }
      }
    ]
  }
}
```

The discussion started in the [Vespa Slack](https://vespatalk.slack.com/archives/C01QNBPPNT1/p1700757803742789).

## Cleanup

```shell
docker kill vespa
```
