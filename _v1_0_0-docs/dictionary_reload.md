---
layout: default
title: Dictionary Reload
prev_section:
next_section:
category: HowTo's
permalink: v1_0_0-docs/dictionary_reload/
---

## Dictionary reload

Reloading a worker's dictionary means clearing them and executing a number of SPARQL queries to fill them afresh from a given SPARQL endpoint. It is potentially very long operation and should only be needed in few cases:
* when a worker is brand new
* when the database model has changed drastically (in which cases, queries should generally be updated too)
* when gazetteer caches have become corrupted on a worker
* when a worker is too far behind the data in the database for incremental updates to be practical

Workers can not annotate while they are reloading. When the coordinator requests a worker reload or sees a worker is reloading, it will not send further annotation requests to that worker.

### API reference

_# TODO: get something from swagger and expand with examples_
