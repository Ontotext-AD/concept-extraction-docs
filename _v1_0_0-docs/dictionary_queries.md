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

### Query consistency in the cluster

If workers in the same annotation cluster have different queries, the coordinator will generally not attempt to amend the situation, because it has no way of knowing which queries are the "correct" ones. Instead, the coordinator provides an API call to check the query consistency status _# TODO: link_ and another call to set queries on all workers _# TODO: link_

### API reference

_# TODO: get something from swagger and expand with examples_
