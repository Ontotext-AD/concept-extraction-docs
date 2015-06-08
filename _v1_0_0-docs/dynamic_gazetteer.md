---
layout: default
title: Dynamic Gazetteer Explained
prev_section: annotating_content
next_section: dictionary_queries
category: HowTo's
permalink: v1_0_0-docs/dynamic_gazetteer/
---

Gazetteers are text processing resources, optimised to quickly look up words against very large sets of labels. They fill their dictionaries (data set) via a SPARQL endpoint, usually from GraphDB. Ideally, a gazetteer should reflect the changes in its source knowledge base. However, loading entire large datasets from GraphDB can take a couple of hours. Ontotext's Dynamic Linked Data Gazetteer is able to update incrementally, efficiently keeping up with the data. It also has an additional metadata component, which is responsible for adding more entity features to the candidate annotation and is also kept in sync.

## Flow

1. A client modifies some data in GraphDB by adding a new entity, deleting an existing one, adding/removing/changing a label, etc.
2. The Entity Update Feed (EUF) plugin in GraphDB tracks transactions and records entities that have been modified in each transaction. A fingerprint (checksum) is calculated after each transaction and serves as an identifier of the database state at that point. The EUF plugin provides SPARQL hooks that allow querying for the current fingerprint and the list of modified entities between two arbitrary fingerprints.
3. The coordinator checks the EUF current fingerprint periodically. When the fingerprint changes, the coordinators notify the workers to update to the new fingerprint.
4. A worker receives a request to update the gazetteer with the new fingerprint. Then it executes the usual queries that the gazetteer uses to retrieve its data, embellished with a special clause for the EUF plugin. This restricts the query result to only the data modified after the current worker fingerprint. Finally, the worker receives its new fingerprint from the coordinator and it is up to date.
<img src="{{ site.baseurl }}/img/Dictionary_Update_Sequence.png" alt="Dictionary_Update_Sequence" style="width:800px;height:350px">

## Update propagation

When the coordinator sees an update to GraphDB (by observing new fingerprints), it first picks a single random worker and tries to update it to verify that the update can be applied. If the worker fails, another one is picked at random to verify the update. This process is repeated until a worker suceeds or the `-Dcoordinator.updates.maxWorkersToVerify` workers fail. If the workers fail, the update is discarded and is not attempted again. If the verification passes, all remaining workers are requested to update simultaneously.

<img src="{{ site.baseurl }}/img/Cluster Update Propagation Activity.png" alt="Cluster Update Propagation Activity.png" style="width:700px;height:550px">

## Consistency

CES does not guarantee that all workers will be up to date with GraphDB at any given time. It does its best to keep the workers updated and consistent, without compromising availability:

* Workers will attempt to update to the latest fingerprint on every database change. It means that workers that accidentally skip or fail an update (due to network outage for example) will be brought up-to-date very soon;
* Workers that fail updates too many times (as defined by `-Dcoordinator.updates.maxUpdateFailures`) are disabled and taken out of the annotation cluster;
* workers that have missed too many incremental updates (as defined by `-Dcoordinator.updates.missedUpdatesBeforeReload`) will be fully reloaded. In other words, they are temporarily disabled, their gazetteers completely wiped out and loaded entirely from GraphDB. The workers are then enabled for annotation again;
* Workers that failed a reload are disabled and taken out of the annotation cluster.

## Worker update statuses

The possible values for worker.updateStatus are as follows:

* `GOOD` - the worker is up to date;
* `VERIFYING` - the worker has been picked to verify an update and is currently doing so;
* `UPDATING` - the worker is updating its dictionaries. It means that a verification of that update has successfully passed first;
* `RELOADING` - performing full dictionary reload from GraphDB. It can take a couple of hours;
* `FAILED` - a previous update or verification request failed. Usually, because of network connection;
* `RELOAD_FAILED` - a reload failed. This usually means that the worker does not have a connection to the database or there is an issue with the disk space or the hardware;
* `TOO_MANY_FAILURES` - a worker has failed to update too many times in a row (as defined by `-Dcoordinator.updates.maxUpdateFailures`). It will no longer annotate content and requires a human to check on it.
