---
layout: default
title: Manage Dictionary Queries
prev_section: dynamic_gazetteer
next_section: dictionary_reload
category: HowTo's
permalink: v1_0_0-docs/dictionary_queries/
---

The gazetteer and metadata components use a GraphDB SPARQL endpoint to retrieve entities. What is retrieved is configured by setting a number of SPARQL queries for each component.


The queries should be `SELECT` (as opposed to `CONSTRUCT`) and gazetteer queries should always have `concept`, `type` and `literal` bindings in that order:

<pre><code>SELECT ?concept ?type ?literal { # ...</code></pre>

Metadata queries need `concept` and `type`. Any other bindings will be added to the metadata per entity. Both the gazetteer and the metadata can use multiple queries. However, all API calls use a single string per worker and the individual queries are separated by `@@@`.

<pre><code>
SELECT ?concept ?type ?literal { ... }
@@@
SELECT ?concept ?type ?literal { ... }
</code></pre>

## Query consistency in the cluster

If the workers of one annotation cluster have different queries, the coordinator will generally not attempt to amend the situation as it does not know which are the correct queries. Instead, the coordinator provides an API call to check the query [consistency status](#statusQueries) and another call to [set queries](#setQueries) on all workers.

## API reference

### `GET /dictionaries/status` <a id="statusQueries"/>
<div class="info-badge">Returns whether the worker queries in the cluster are in sync.</div>

- *Query params*: none;
- *Response*: a JSON object with two members:
  * `sync` - boolean, true, if all workers have the same queries, otherwise - false;
  * `forcedQueries` - the queries, if any, set through one of the `POST /dictionaries/queries` calls.
- *Status code*: 200 on success.

### `GET /dictionaries/queries` <a id="getQueries"/>
<div class="info-badge">Returns the queries, if any, set on the coordinator as cluster queries.</div>

* *Query params*: none;
* *Response*: a JSON map mirroring what was set by the `POST /dictionaries/queries` call (the keys in the map are pipeline resource names (gazetteer/metadata), the values are the respective queries for each resource);
* *Status code*: 200 on success.

### `POST /dictionaries/queries` <a id="setQueries"/>

<div class="info-badge">
Sets the cluster queries. The specified queries will be set immediately on all active workers. On currently inactive workers, the queries will be set when they become available. Resources that do not exist on a worker will be ignored by this worker. If setting the queries on a worker fails, it will not be reattempted.</div>

* *Query params*: none;
* *Request body*: a JSON map where the keys in the map are pipeline resource names (gazetteer/metadata), the values are the respective queries for each resource;
* *Response*: empty;
* *Status code*: 200 on success.

### `POST /dictionaries/queries/from` <a id="setQueriesFrom"/>

<div class="info-badge">
Sets a worker queries as cluster ones. It behaves as <code>POST /dictionaries/queries</code> but instead of accepting queries directly in the body, it takes the worker URL and sets its queries as cluster ones.</div>

* *Query params*: `worker` - a worker's full URL;
* *Response*: empty;
* *Status code*:
  * 200 on success;
  * 400, if the worker does not exist.


### `DELETE /dictionaries/queries` <a id="deleteQueries"/>

<div class="info-badge">Removes the queries set by <code>POST /dictionaries/queries</code>. Workers whose queries are not set after the last <code>POST /dictionaries/queries</code> will keep their old queries.</div>

* *Query params*: none;
* *Response*: empty;
* *Status code*: 200 on success.
