---
layout: default
title: Dynamic Gazetteer
prev_section: annotating_content
next_section: dictionary_queries
category: HowTo's
permalink: v1_0_0-docs/dynamic_gazetteer/
---

## Dynamic Gazetteer

### Motivation

Gazetteers are optimized to quickly look up words against very large set of labels, and take their data via a SPARQL endpoint, usually from GraphDB. Ideally, a gazetteer should reflect the changes in its source knowledge base, however loading entire large datasets from GraphDB can take a couple of hours. The gazetteer is be able to update incrementally, efficiently keeping up with the data.

### Flow

1. A client modifies some data in GraphDB - adds a new entity, deletes an entity, adds/removes/changes a label, etc.
2. Entity update feed (EUF) plugin in GraphDB tracks transactions and records entities that have been modified in each transaction. A fingerprint (checksum) is calculated after each transaction and serves as an identifier of the database state at that point. The EUF plugin provides SPARQL hooks that allow querying for the current fingerprint and the list of modified entities between two arbitrary fingerprints.
_do we need more details? special section for EUF with predicates and stuff?_
3. The coordinator checks EUF current fingerprint periodically. When the fingerprint changes, the coordinators notify the workers to update to the new fingerprint.
4. A worker receives a request to update the gazetteer with the new fingerprint. Then it executes the usual queries the gazetteer uses to retrieve its data, with an added clause, which EUF plugin interprets, to leave only the data modified after the current worker fingerprint and the new fingerprint passed by the coordinator. Finally, the worker updates its fingerprint and is up to date.

<img src="img/FT_Dictionary_Updates.png" alt="Dictionary Updates" style="width:304px;height:228px;">

### Update propagation

When the coordinator sees an update to GraphDB (by observing new fingerprin) it first picks a single random worker and tries to update it to verify the update can be applied. If the worker fails, another one is picked at random to verify the update. This process is repeated until a worker suceeds or `-Dcoordinator.updates.maxWorkersToVerify` workers fail. If the workers fail, the update is discarded and is not attempted again. If the verification passes, all remaining workers are requested to update simultaneously.

### Consistency

CES doesn't guarantee that all workers will be up to date with GraphDB at any given time. It does its best to keep the workers updated and consistent, without compromising availability:
* workers will attempt to update to the latest fingerprint on every database change. That means workers that accidently skip or fail an update (due to network outage for example) will be brought up-to-date very soon
* workers that fail updates too many times (as defined by `-Dcoordinator.updates.maxUpdateFailures`) are disabled and taken out of the annotation cluster
* workers that have missed too many incremental updates (as defined by `-Dcoordinator.updates.missedUpdatesBeforeReload`) will be fully reloaded - meaning they are temporarily disabled, their gazetteers completely wiped out and loaded entirely from GraphDB. The workers are then enabled for annotation again.
* workers that failed a reload are disabled and taken out of the annotation cluster

### Worker update statuses

The possible values for worker.updateStatus are as follows:
* `GOOD` - the worker is up to date
* `VERIFYING` - the worker has been picked to verify an update and is currently doing so
* `UPDATING` - the worker is updating its dictionaries. It means that a verification of that update has successfully passed first
* `RELOADING` - performing full dictionary reload from GraphDB. Can take a couple of hours
* `FAILED` - a previous update or verification request failed. Usually because of network connection.
* `RELOAD_FAILED` - a reload failed. This usually means the worker doesn't have connection to the database or there's a disk space or hardware issue
* `TOO_MANY_FAILURES` - a worker has failed to update too many times (as defined by `-Dcoordinator.updates.maxUpdateFailures`) in a row - it will no longer annotate content and requires a human to check on it
