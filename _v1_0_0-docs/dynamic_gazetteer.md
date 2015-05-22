---
layout: default
title: Dynamic Gazetteer
prev_section:
next_section:
category: HowTo's
permalink: v1_0_0-docs/dynamic_gazetteer/
---

## Dynamic Gazetteer

### Motivation

Gazetteers are optimized to quickly look up words against very large set of labels, and take their data from a SPARQL endpoint, usually GraphDB. Ideally, a gazetteer should reflect the changes in its source knowledge base, however loading entire larged datasets from GraphDB can take a couple of hours. The gazetteer should be able to update incrementally, efficiently keeping up with the data.

### Flow

1. A client modifies some data in GraphDB - add a new entity, delete an entity, add/remove/change a label, etc
2. Entity update feed (EUF) plugin in GraphDB tracks transactions and records entities that have been modified in each transaction. A fingerprint (checksum) is calculated after each transaction that serves as an identifier of database state at that point. The EUF plugin provides SPARQL hooks that allow querying for the current fingerprint and the list of modified entities between two arbitrary fingerprints. _do we need more details? special section for euf with predicates and stuff?_
3. The coordinator checks EUF current fingerprint periodically. When the fingerprint changes, the coordinators notify the workers to update to the new fingerprint.
4. A worker receive a request to update gazetteer with the new fingerprint. It then executes the usual queries the gazetteer uses to retrieve its data, with an added clause (that EUF plugin interprets) to leave only the data modified after the current worker fingerprint and the new fingerprint passed by the coordinator. The worker then updates its fingerprint and is up to date

<pre><code>
TODO: diagram?
</code></pre>

### Update propagation

When the coordinator sees an update to GraphDB (by observing new fingerprin) it first picks a single random worker and tries to update it to verify the update can be applied. If the worker fails, another one is picked at random to verify the update. This process is repeated until a worker suceeds or `-Dcoordinator.updates.maxWorkersToVerify` workers fail. If the workers fail, the update is discarded and is not attempted again. If the verification passes, all remaining workers are requested to update simultaneously. 

### Consistency

CES doesn't guarantee that all workers will be up to date with GraphDB at any given time. It does its best to keep the workers updated and consistent, without compromising availability:
* workers will attempt to update to the latest fingerprint on every database change. That means workers that accidently skip or fail an update (due to network outage for example) will be brought up-to-date very soon
* workers that fail updates too many times (as defined by `-Dcoordinator.updates.maxUpdateFailures`) are disabled and taken out of the annotation cluster
* workers that have missied too many incremental updates (as defined by `-Dcoordinator.updates.missedUpdatesBeforeReload`) will be fully reloaded - meaning they are temporarily disabled, their gazetteers completely wiped out and loaded entirely from GraphDB. The workers are then enabled for annotation again. 
* workers that failed a reload are disabled and taken out of the annotation clulster

### Worker update statuses

The possible values for worker.updateStatus are as follows:
* `GOOD` - the worker is up to date
* `VERIFYING` - the worker has been picked to verify an update and is currently doing so
* `UPDATING` - the worker is updating its dictionaries. It means that a verification of that update has successfully passed first
* `RELOADING` - performing full dictionary reload from GraphDB. Can take a couple of hours
* `FAILED` - a previous update or verification request failed. Ususally because of network connection. 
* `RELOAD_FAILED` - a reload failed. This usually means the worker doesn't have connection to the database or there's a disk space or hardware issue
* `TOO_MANY_FAILURES` - a worker has failed to update too many times (as defined by `-Dcoordinator.updates.maxUpdateFailures`) in a row - it will no longer annotate content and requires a human to check on it
