# Semantic conventions for Elasticsearch

**Status**: [Experimental](../../../document-status.md)

This document defines semantic conventions to apply when creating a span to capture requests to Elasticsearch.

## Span Name

The **span name** SHOULD be of the format `<http.request.method> <url.path *with placeholders*>`.

The elasticsearch url path is used with placeholders in order to reduce the cardinality of the span name. When the
path contains a document id, it SHOULD be represented as the identifier `{id}`. When the path contains a target data stream
or index, it SHOULD be represented as `{target}`.
For example, a request to `/test-index/_doc/123` should have the span name `GET /{target}/_doc/{id}`.
When there is no target or document id, the span name contains the exact path, as in `POST /_search`.

## Span attributes

<!-- semconv db.elasticsearch -->
| Attribute  | Type | Description  | Examples  | Requirement Level |
|---|---|---|---|---|
| `db.elasticsearch.doc_id` | string | The document that the request targets. | `123`; `456` | Conditionally Required: [1] |
| `db.elasticsearch.target` | string | The name of the data stream or index that is targeted. | `users` | Conditionally Required: [2] |
| [`db.operation`](../database.md) | string | The endpoint identifier for the request. [3] | `search`; `ml.close_job`; `cat.aliases` | Required |
| [`db.statement`](../database.md) | string | The request body for a [search-type query](https://www.elastic.co/guide/en/elasticsearch/reference/current/search.html), as a json string. [4] | `"{\"name\":\"TestUser\",\"password\":\"REDACTED\"}"` | Recommended: [5] |
| `http.request.method` | string | HTTP request method. | `GET`; `POST`; `HEAD` | Required |
| [`server.address`](../span-general.md) | string | Logical server hostname, matches server FQDN if available, and IP or socket address if FQDN is not known. | `example.com` | See below |
| [`server.port`](../span-general.md) | int | Logical server port number | `80`; `8080`; `443` | Recommended |
| `url.full` | string | Absolute URL describing a network resource according to [RFC3986](https://www.rfc-editor.org/rfc/rfc3986) [6] | `https://localhost:9200/index/_search?q=user.id:kimchy` | Required |

**[1]:** when the request targets a specific document by id

**[2]:** when a specific index or data stream is targeted by the request

**[3]:** When setting this to an SQL keyword, it is not recommended to attempt any client-side parsing of `db.statement` just to get this property, but it should be set if the operation name is provided by the library being instrumented. If the SQL statement has an ambiguous operation, or performs more than one operation, this value may be omitted.

**[4]:** The value may be sanitized to exclude sensitive information.

**[5]:** Should be collected when a search-type query is executed

**[6]:** For network calls, URL usually has `scheme://host[:port][path][?query][#fragment]` format, where the fragment is not transmitted over HTTP, but if it is known, it should be included nevertheless.
`url.full` MUST NOT contain credentials passed via URL in form of `https://username:password@www.example.com/`. In such case username and password should be redacted and attribute's value should be `https://REDACTED:REDACTED@www.example.com/`.
`url.full` SHOULD capture the absolute URL when it is available (or can be reconstructed) and SHOULD NOT be validated or modified except for sanitizing purposes.
<!-- endsemconv -->