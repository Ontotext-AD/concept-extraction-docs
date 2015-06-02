---
layout: default
title: Dictionary Reload
prev_section: dictionary_queries
next_section: load_balancing
category: HowTo's
permalink: v1_0_0-docs/dictionary_reload/
---

## Dictionary reload

Reloading a worker's dictionary means clearing it and executing a number of SPARQL queries to fill it afresh from a given SPARQL endpoint. It is potentially very long operation and should only be needed in few cases:
* when a worker is brand new;
* when the database model has changed drastically (in which cases, queries should generally be updated too);
* when gazetteer caches have become corrupted on a worker;
* when a worker is too far behind the data in the database for incremental updates to be practical.

Workers can not annotate while they are reloading. When the coordinator requests a worker reload or sees a worker is reloading, it will not send further annotation requests to that worker.

### API reference

Notes _(TODO: common for all API references - maybe move somewhere?)_
* all API calls URLs are relative to coordinator's root, e.g. if the coordinator is installed at http://example.org/coordinator,
the /update/force/reload call's full URL will be http://example.org/coordinator/updates/force/reload
* all requests produce JSON without envelope (unless noted otherwise); therefore the request `Accept` header should match
`application/json` and the response `Content-type` header will be `application/json`
* query parameters for POST and PUT requests can be specified either in the URL or as `application/x-form-www-urlencoded`
* all requests that specifically expect request body accept only `application/json`

#### _POST /updates/force/reload_

Reloads a single worker. The coordinator will not immediately start reloading the worker if it is the last one enabled in the cluster
to prevent loss of service. The worker will be immediately reloaded if it is the only one in the cluster or there are other workers
that are available (i.e. not down or reloading, etc)

**Query params**:

* **url** - the full URL of a worker to reload

**Response**: Response will be empty on success

**Status codes**:

* 202 - when the call was successful
* 400 - if the worker was not found


#### _POST /updates/force/reload/all_

Reloads all workers. Not all workers are reloaded simultaneously - the coordinator will try to keep one worker available for
annotation (unless there's only one worker - in that case it is reloaded immediately)

**Query params**: none

**Response**: Response will be empty on success

**Status codes**:

* 202 - when the call was successful
