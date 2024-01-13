# Setting up Elasticsearch & Kibana on Windows

## Starting up Elasticsearch

```
cd path\to\elasticsearch
bin\elasticsearch.bat
```

## Resetting the `elastic` user's password

If you lose the password for the `elastic` user, it can be reset with the following commands.

```
cd path\to\elasticsearch
bin\elasticsearch-reset-password.bat -u elastic
```

## Generating a new Kibana enrollment token

If you need to generate a new enrollment token for Kibana, this can be done with the following commands.

```
cd path\to\elasticsearch
bin\elasticsearch-create-enrollment-token.bat -s kibana
```

## Starting up Kibana

```
cd path\to\kibana
bin\kibana.bat
```

# Setting up Elasticsearch & Kibana on macOS & Linux

## Extracting the archives

Both the Elasticsearch and Kibana archives can be extracted by using the below commands.
Alternatively, simply double-clicking them should do the trick.

```
cd /path/to/archive/directory
tar -zxf archive.tar.gz
```

## Starting up Elasticsearch

```
cd /path/to/elasticsearch
bin/elasticsearch
```

## Resetting the `elastic` user's password

If you lose the password for the `elastic` user, it can be reset with the following commands.

```
cd /path/to/elasticsearch
bin/elasticsearch-reset-password -u elastic
```

## Generating a new Kibana enrollment token

If you need to generate a new enrollment token for Kibana, this can be done with the following commands.

```
cd /path/to/elasticsearch
bin/elasticsearch-create-enrollment-token --scope kibana
```

## Disabling Gatekeeper for Kibana directory

macOS contains a security feature named Gatekeeper, which prevents Kibana from starting up.
We can disable it for just the Kibana directory, which allows Kibana to start up correctly.
Simply use the following command to do so.

```
xattr -d -r com.apple.quarantine /path/to/kibana
```

## Starting up Kibana

```
cd /path/to/kibana
bin/kibana
```

# Inspecting the cluster

## Checking the cluster's health

```
GET /_cluster/health
```

## Listing the cluster's nodes

```
GET /_cat/nodes?v
```

## Listing the cluster's indices

```
GET /_cat/indices?v&expand_wildcards=all
```

# Sending queries with cURL

## Handling self signed certificates

Elasticsearch is protected with a self signed certificate by default, which HTTP clients do not trust. 
Sending a request will therefore fail with a certificate error. To fix this, we have a couple of options.

### 1. Skip certificate verification

One option is to entirely skip the verification of the certificate. This is not exactly best practice, 
but if you are just developing with a local cluster, then it might be just fine. To ignore the 
certificate, use either the `--insecure` flag or `-k`.

```
curl --insecure [...]
curl -k [...]
```

### 2. Provide the CA certificate

A better approach is to provide the CA certificate so that the TLS certificate is not just ignored. 
The path to the file can be supplied with the `--cacert` argument. The CA certificate is typically stored within 
the `config/certs` directory, although the `certs` directory may be at the root of your Elasticsearch 
home directory (`$ES_HOME`) depending on how you installed Elasticsearch.

```
# macOS & Linux
cd /path/to/elasticsearch
curl --cacert config/certs/http_ca.crt [...]

# Windows
cd C:\Path\To\Elasticsearch
curl --cacert config\certs\http_ca.crt [...]
```

Alternatively, you can specify the absolute path to the file.

## Authentication
All requests made to Elasticsearch must be authenticated. For local deployments, use the password that 
was generated for the `elastic` user the first time Elasticsearch started up.

```
curl -u elastic [...]
```

The above will prompt you to enter the password when running the command. Alternatively, you can enter 
the password directly within the command as follows (without the brackets).

```
curl -u elastic:[YOUR_PASSWORD_HERE] [...]
```

Note that this exposes your password within the terminal, so this is not best practice from a security perspective.

## Adding a request body & `Content-Type` header

To send data within the request, use the `-d` argument, e.g. for the `match_all` query. Note that using 
single quotes does not work on Windows, so each double quote within the JSON object must be escaped.

```
# macOS & Linux
curl [...] https://localhost:9200/products/_search -d '{ "query": { "match_all": {} } }'

# Windows
curl [...] https://localhost:9200/products/_search -d "{ \"query\": { \"match_all\": {} } }"
```

When sending data (typically JSON), we need to tell Elasticsearch which type of data we are sending. This 
can be done with the `Content-Type` HTTP header. Simply add it with cURL's `-H` argument.

```
curl -H "Content-Type:application/json" [...]
```

## Specifying the HTTP verb

You may also specify the HTTP verb (e.g. `POST`). This is necessary for some endpoints, such as when 
indexing documents. `GET` is assumed by default.

```
curl -X POST [...]
```

## All together now

```
# macOS & Linux
curl --cacert config/certs/http_ca.crt -u elastic https://localhost:9200/products/_search -d '{ "query": { "match_all": {} } }'

# Windows
curl --cacert config\certs\http_ca.crt -u elastic https://localhost:9200/products/_search -d "{ \"query\": { \"match_all\": {} } }"
```

# Sharding and scalability

## Listing the cluster's indices

```
GET /_cat/indices?v
```

# Understanding replication

## Creating a new index

```
PUT /pages
```

## Checking the cluster's health

```
GET /_cluster/health
```

## Listing the cluster's indices

```
GET /_cat/indices?v
```

## Listing the cluster's shards

```
GET /_cat/shards?v
```

# Adding more nodes to the cluster (for development)

## Checking the cluster's health

```
GET /_cluster/health
```

## Checking the shard distribution

```
GET /_cat/shards?v
```

## Generating an enrollment token
When adding a new node to an existing Elasticsearch cluster, we first need to generate an enrollment token.

```
# macOS & Linux
bin/elasticsearch-create-enrollment-token --scope node

# Windows
bin\elasticsearch-create-enrollment-token.bat -s node
```

## Adding a new node to the cluster
To add a new node to an existing cluster, run the following command. Remember to have the working 
directory set to the new node's `$ES_HOME` directory (use the `cd` command for this).

```
# macOS & Linux
bin/elasticsearch --enrollment-token [INSERT_ENROLLMENT_TOKEN_HERE]

# Windows
bin\elasticsearch.bat --enrollment-token [INSERT_ENROLLMENT_TOKEN_HERE]
```

Once the node has been added, starting up the node again in the future is as simple as 
running `bin/elasticsearch` (macOS & Linux) or `bin\elasticsearch.bat` (Windows).

# Overview of node roles

## Listing the cluster's nodes (and their roles)

```
GET /_cat/nodes?v
```