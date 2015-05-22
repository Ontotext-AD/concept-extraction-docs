---
layout: default
title: Dictionary Queries
prev_section:
next_section:
category: HowTo's
permalink: v1_0_0-docs/dictionary_queries/
---

## Dictionary Queries

The gazetteer uses SPARQL endpoint, usually GraphDB, to retrieve entities. What exactly is retrieved is configured by
setting a number of SPARQL queries for each gazetteer. The queries should be `SELECT` (as opposed to `CONSTRUCT`) and
gazetteer queries should always have `concept`, `type` and `literal` bindings in that order:
<pre><code>SELECT ?concept ?type ?literal { # ...</code></pre>
Meta data queries need `concept` and `type`, any other bindings will be added to the meta data per entity

### Query consistency in the cluster

TODO:

