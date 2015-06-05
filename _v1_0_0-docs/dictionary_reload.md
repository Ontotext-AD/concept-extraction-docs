---
layout: default
title: Reload dictionaries
prev_section: dictionary_queries
next_section: load_balancing
category: HowTo's
permalink: v1_0_0-docs/dictionary_reload/
---

Reloading a worker's dictionary means clearing it and executing a number of SPARQL queries to fill it afresh from a given SPARQL endpoint. It is potentially a very long operation and should be done only in few cases:

* when a worker is brand new;
* when the database model has changed drastically, in which case queries should be updated before reloading (see <a href="{{ site.baseurl }}/v1_0_0-docs/dictionary_queries">Manage Dictionary Queries</a>);
* when gazetteer caches have become corrupted on a worker (i.e. caused by disk failure);
* when a worker is too far behind the data in the database for incremental updates to be practical.

Workers cannot annotate while they are reloading. When the coordinator requests a worker reload or sees that a worker is reloading, it will not send further annotation requests to it.

## API reference

* All API calls URLs are relative to the coordinator's root, e.g. if the coordinator is installed at `http://example.org/coordinator`, the full URL of the `/update/force/reload` will be `http://example.org/coordinator/updates/force/reload`;
* All requests produce JSON without envelope (unless noted otherwise). Therefore, the request `Accept` header should match `application/json` and the response `Content-type` header will be `application/json`;
* Query parameters for `POST` and `PUT` requests can be specified either in the URL or as `application/x-form-www-urlencoded`;
* All requests that specifically expect a request body accept only `application/json`.

### `POST /updates/force/reload`
<div class="info-badge">
Reloads a single worker. If the worker is the only one in the cluster or there are other workers available (i.e. not down, reloading, etc.), it is reloaded immediately. If it is the last one enabled in the cluster, it is not reloaded immediately, in order to prevent loss of service.</div>

* *Query params*: `url` - the full URL of a worker to reload;
* *Response*: Empty on success;
* *Status codes*:
  * 202 on success;  
  * 400 if the worker is not found.


### `POST /updates/force/reload/all`
<div class="info-badge">
Reloads all workers. Not all workers are reloaded simultaneously. The coordinator tries to keep one worker available for annotation. In case there is only one worker, it will be reloaded immediately.</div>

* *Query params*: none;
* *Response*: Empty on success;
* *Status codes*: 202 on success.
