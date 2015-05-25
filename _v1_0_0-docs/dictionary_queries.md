---
layout: default
title: Dictionary Queries
prev_section:
next_section:
category: HowTo's
permalink: v1_0_0-docs/dictionary_queries/
---

## Dictionary queries

The gazetteer uses SPARQL endpoint, usually GraphDB, to retrieve entities. What exactly is retrieved is configured by
setting a number of SPARQL queries for each gazetteer. The queries should be `SELECT` (as opposed to `CONSTRUCT`) and
gazetteer queries should always have `concept`, `type` and `literal` bindings in that order:
<pre><code>SELECT ?concept ?type ?literal { # ...</code></pre>
Meta data queries need `concept` and `type`, any other bindings will be added to the meta data per entity
Both gazetteer and meta data can use multiple queries. In all API calls, however, a single string per worker is used in which 
individual queries are separated by `@@@` on a line of its own, like this:
<pre><code>
SELECT ?concept ?type ?literal { ... }
@@@
SELECT ?concept ?type ?literal { ... }
</code></pre>

### Query consistency in the cluster

If workers in the same annotation cluster have different queries, the coordinator will generally not attempt to amend the situation, because it has no way of knowing which queries are the "correct" ones. Instead, the coordinator provides an API call to check the query consistency status _# TODO: link_ and another call to set queries on all workers _# TODO: link_

### API reference

_# TODO: general notes from dictionary_reload?_

#### _GET /dictionaries/status_

Returns whether the worker queries in the cluster are in sync

**Query params**: none

**Response**: JSON object with two members:
* **sync** - boolean, true if all workers have the same queries, false otherwise
* **forcedQueries** - the queries, if any, set through one of the *POST /dictionaries/queries* calls

**Status code**: 200 on success

#### _GET /dictionaries/queries_

Returns the queries, if any, set on the coordinator as cluster queries
  
**Query params**: none

**Response**: JSON map mirroring what was set by *POST /dictionaries/queries* call (i.e. a key in the map is a pipeline resource names,
the value is the queries for that resource)

**Status code**: 200 on success

#### _POST /dictionaries/queries_

Sets the cluster queries. The specified queries will be set immediately on all active workers, attempt to set the queries on currently
inactive workers will be made when they become available. Resources that do not exist on a worker will be ignored by that worker.
If setting the queries on a worker fails, it won't be reattempted. 

**Query params**: none

**Request body**: JSON map where the keys are pipeline resource names, the values are queries for the respective resource name.

**Response**: response will be empty

**Status code**: 200 on success

#### _POST /dictionaries/queries/from_

Sets a worker queries as the cluster queries. Behaves as the _POST /dictionaries/queries_, except this calls takes worker URL and
sets its queries as the cluster ones, instead of accepting queries directly in the body

**Query params**: 

* **worker** - a worker's full URL

**Response**: response will be empty

**Status code**: 

* 200 on success
* 400 if the worker does not exist


#### _DELETE /dictionaries/queries_

Removes the queries set by _POST /dictionaries/queries_. Workers that haven't had their queries set after the last 
_POST /dictionaries/queries_ will keep their old queries. 

**Query params**: none

**Response**: response will be empty

**Status code**: 200 on success
