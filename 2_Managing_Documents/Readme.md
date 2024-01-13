# <p align="center">Managing Documents</p>

# Creating & Deleting Indices

## Deleting an index

```
DELETE /pages
```

## Creating an index (with settings)

```
PUT /products
{
  "settings": {
    "number_of_shards": 2,
    "number_of_replicas": 2
  }
}
```

# Indexing documents

## Indexing document with auto generated ID:

```
POST /products/_doc
{
  "name": "Coffee Maker",
  "price": 64,
  "in_stock": 10
}
```

## Indexing document with custom ID:

```
PUT /products/_doc/100
{
  "name": "Toaster",
  "price": 49,
  "in_stock": 4
}
```

# Retrieving documents by ID

```
GET /products/_doc/100
```

# Updating documents

## Updating an existing field

```
POST /products/_update/100
{
  "doc": {
    "in_stock": 3
  }
}
```

## Adding a new field

_Yes, the syntax is the same as the above. ;-)_

```
POST /products/_update/100
{
  "doc": {
    "tags": ["electronics"]
  }
}
```

# Scripted updates

## Reducing the current value of `in_stock` by one

```
POST /products/_update/100
{
  "script": {
    "source": "ctx._source.in_stock--"
  }
}
```

## Assigning an arbitrary value to `in_stock`

```
POST /products/_update/100
{
  "script": {
    "source": "ctx._source.in_stock = 10"
  }
}
```

## Using parameters within scripts

```
POST /products/_update/100
{
  "script": {
    "source": "ctx._source.in_stock -= params.quantity",
    "params": {
      "quantity": 4
    }
  }
}
```

## Conditionally setting the operation to `noop`

```
POST /products/_update/100
{
  "script": {
    "source": """
      if (ctx._source.in_stock == 0) {
        ctx.op = 'noop';
      }
      
      ctx._source.in_stock--;
    """
  }
}
```

## Conditionally update a field value

```
POST /products/_update/100
{
  "script": {
    "source": """
      if (ctx._source.in_stock > 0) {
        ctx._source.in_stock--;
      }
    """
  }
}
```

## Conditionally delete a document

```
POST /products/_update/100
{
  "script": {
    "source": """
      if (ctx._source.in_stock < 0) {
        ctx.op = 'delete';
      }
      
      ctx._source.in_stock--;
    """
  }
}
```

# Upserts

```
POST /products/_update/101
{
  "script": {
    "source": "ctx._source.in_stock++"
  },
  "upsert": {
    "name": "Blender",
    "price": 399,
    "in_stock": 5
  }
}
```

# Replacing documents

```
PUT /products/_doc/100
{
  "name": "Toaster",
  "price": 79,
  "in_stock": 4
}
```

# Deleting documents

```
DELETE /products/_doc/101
```

# Optimistic concurrency control

## Retrieve the document (and its primary term and sequence number)
```
GET /products/_doc/100
```

## Update the `in_stock` field only if the document has not been updated since retrieving it
```
POST /products/_update/100?if_primary_term=X&if_seq_no=X
{
  "doc": {
    "in_stock": 123
  }
}
```

# Update by query

## Updating documents matching a query

Replace the `match_all` query with any query that you would like.

```
POST /products/_update_by_query
{
  "script": {
    "source": "ctx._source.in_stock--"
  },
  "query": {
    "match_all": {}
  }
}
```

## Ignoring (counting) version conflicts

The `conflicts` key may be added as a query parameter instead, i.e. `?conflicts=proceed`.

```
POST /products/_update_by_query
{
  "conflicts": "proceed",
  "script": {
    "source": "ctx._source.in_stock--"
  },
  "query": {
    "match_all": {}
  }
}
```

## Matches all of the documents within the `products` index

```
GET /products/_search
{
  "query": {
    "match_all": {}
  }
}
```

# Delete by query

## Deleting documents that match a given query

```
POST /products/_delete_by_query
{
  "query": {
    "match_all": { }
  }
}
```

## Ignoring (counting) version conflicts

The `conflicts` key may be added as a query parameter instead, i.e. `?conflicts=proceed`.

```
POST /products/_delete_by_query
{
  "conflicts": "proceed",
  "query": {
    "match_all": { }
  }
}
```

# Batch processing

## Indexing documents

```
POST /_bulk
{ "index": { "_index": "products", "_id": 200 } }
{ "name": "Espresso Machine", "price": 199, "in_stock": 5 }
{ "create": { "_index": "products", "_id": 201 } }
{ "name": "Milk Frother", "price": 149, "in_stock": 14 }
```

## Updating and deleting documents

```
POST /_bulk
{ "update": { "_index": "products", "_id": 201 } }
{ "doc": { "price": 129 } }
{ "delete": { "_index": "products", "_id": 200 } }
```

## Specifying the index name in the request path

```
POST /products/_bulk
{ "update": { "_id": 201 } }
{ "doc": { "price": 129 } }
{ "delete": { "_id": 200 } }
```

## Retrieving all documents

```
GET /products/_search
{
  "query": {
    "match_all": {}
  }
}
```

# Importing data with cURL

## Navigating to bulk file directory

```
cd /path/to/data/file/directory
```

### Examples
```
# macOS and Linux
cd ~/Desktop

# Windows
cd C:\Users\[your_username]\Desktop
```

## Importing data into local clusters

```
# Without CA certificate validation. This is fine for development clusters, but don't do this in production!
curl -k -u elastic -H "Content-Type:application/x-ndjson" -XPOST https://localhost:9200/products/_bulk --data-binary "@products-bulk.json"

# With CA certificate validation. The certificate is located at $ES_HOME/config/certs/http_ca.crt
curl --cacert /path/to/http_ca.crt -u elastic -H "Content-Type:application/x-ndjson" -XPOST https://localhost:9200/products/_bulk --data-binary "@products-bulk.json"
```

## Importing data into Elastic Cloud 
```
curl -H "Content-Type:application/x-ndjson" -XPOST -u username https://elastic-cloud-endpoint.com:9243/products/_bulk --data-binary "@products-bulk.json"
```
