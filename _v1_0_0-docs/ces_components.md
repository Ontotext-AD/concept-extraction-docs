---
title: CES Components
layout: default
prev_section: installation
next_section: annotating_content
category: Getting Started
permalink: v1_0_0-docs/ces_components/
---

## Overview

This section provides information about all components required to build a resilient Concept Extraction API with dynamically updated Gazetteer dictionaries. If you only need to be able to extract named entities from text with a static dictionary and are not interested in high availability, you can do it with a single worker.

## Worker

### Configuration

#### General

* `-Dworker.name` (optional) - a sub-directory in the worker persistence directory. By default it points to `~/.ces-worker`.

#### GATE

* `-Dgate.app.location` (required) - a full path to the `*.xgapp` file to load, including the filename and extension. It should start with `file:/`, otherwise it is interpreted as relative to the application context.
* `-Dpipeline-pool-max-size` (optional, default = 1) - the maximum number of GATE pooled applications. It shows the number of simultaneous annotations this worker supports.

#### Recommended JVM settings

* GC:
   * `-XX:+UseConcMarkSweepGC -verbose:gc-verbose:sizes`
   * `-Xloggc:/path/to/logs/gc.log`
   * `-XX:+PrintGCDetails`
   * `-XX:+PrintGCDateStamps`
   * `-XX:+PrintTenuringDistribution`
   * `-XX:+UseGCLogFileRotation`
   * `-XX:NumberOfGCLogFiles=5`
   * `-XX:GCLogFileSize=2M`
* Compiler:
    `-XX:+TieredCompilation`
* `-Xmx:` depends on the pipeline in use - each pipeline package should state how much memory it requires.

## Coordinator

### Configuration

All timeouts are in milliseconds unless specified otherwise.

#### General

* `-Dcoordinator.name` (optional) - the name of this coordinator. It is used for the suffix of the directory under HOME in which the coordinator persists its state.
* `-Dcoordinator.stateDirectory` (optional, default = `<home>/.coordinator`) - sets the directory for the state files of the coordinator;
* `-Dcoordinator.baseUrl` (required) - the base address of this coordinator. It gives the workers URLs that point back to the coordinator.

#### GraphDB

* `-Dcoordinator.sparql.endpoint` (required) - the remote SPARQL endpoint URL, including the repository. Usually in the form of `http://<host>:<port>/graphdb/repositories/<repo_name>`.
* `-Dcoordinator.sparql.connectionTimeout` (optional, default = 10000) - establishes the connection to the SPARQL endpoint timeout;
* `-Dcoordinator.sparql.socketTimeout` (optional, default = 600000) - the socket timeout for the SPARQL queries.

#### Workers

* `-Dcoordinator.worker.connectionTimeout` (optional, default = 10000) - establishes the connection to a worker timeout;
* `-Dcoordinator.worker.socketTimeout` (optional, default = 10000) - the socket timeout for the worker communication;
* `-Dcoordinator.worker.retries` (optional, default = 2);
* `-Dcoordinator.worker.retryDelay` (optional, default = 2000);
* `-Dcoordinator.worker.retryDelayMult` (optional, default = 2.0).

#### Updates (dictionaries)

* `-Dcoordinator.updates.checkDelay` (optional, default = 10000) - the initial delay before the first check for updates;
* `-Dcoordinator.updates.checkRate` (optional, default = 600000) - the interval between the checks for updates;
* `-Dcoordinator.updates.maxWorkersToVerify` (optional, default = 2) - a change is first verified on a single worker before it is propagated to all workers. This specifies the maximum number of workers that it attempts to change before giving up;
* `-Dcoordinator.updates.verificationTimeout` (optional, default = 1800000) - the maximum wait time for the update verification.

#### Updates (models)

* `-Dcoordinator.models.endpoint` (optional) - the training node base URL. If not specified, the worker models are not updated;
* `-Dcoordinator.models.schedule` (optional, default = "0 0 2 * * ?") - a cron expression, specifying when to check for updates.  The default value checks for models every day at 2am. For the full syntax and explanation, see [Spring's CronSequenceGenerator documentation](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/scheduling/support/CronSequenceGenerator.html).

#### Annotation

* `-Dcoordinator.annotation.freeWorkerTimeout` (optional, default = 30000) - the maximum wait time for a free worker to become available for annotation;
* `-Dcoordinator.annotation.connectionTimeout` (optional, default = 10000) - establishes the connection to a worker for the annotation timeout;
* `-Dcoordinator.annotation.socketTimeout` (optional, default = 60000) - the socket timeout for annotation to a worker.

#### Watchdog / heartbeat checker

* `-Dcoordinator.watchdog.checkDelay` (optional, default = 60000) - the initial delay before the first heartbeat check;
* `-Dcoordinator.watchdog.checkRate` (optional, default = 60000) - the interval between the heartbeat checks.

#### Files

All files relative to `~/.coordinator/\[$\{coordinator.name\}\]/`, which is `~/.coordinator`, if `coordinator.name` is unset, and `~/.coordinator/<coordinator.name>/`, if it is set.

* `workers.json` - the persisted workers list and configuration;
* `sparql-update-history.json` - the update history for `SparqlUpdatesManager`;
* `models.json` - the latest known models for `ModelUpdatesManager`.

#### JVM settings

* GC:
  * `-XX:+UseConcMarkSweepGC \-verbose:gc \-verbose:sizes \-Xloggc:/path/to/logs/gc.log`;
  * `-XX:+PrintGCDetails \-XX:+PrintGCDateStamps`;  
  * `-XX:+PrintTenuringDistribution`;
  * `-XX:+UseGCLogFileRotation`;
  * `-XX:NumberOfGCLogFiles=5`;
  * `-XX:GCLogFileSize=2M`;
* Compiler: `-XX:+TieredCompilation`;
* `-Xmx:` depends on the pipeline in use - each pipeline should come with memory requirements.

## GraphDB and EUF plug-in

This is the semantic database required to enable the dynamic dictionary updates functionality. If you do not have GraphDB, you can get the latest version [here](http://info.ontotext.com/graphdb-lite-eval-graphdb). For information how to install, etc., see its [ documentation](http://graphdb.ontotext.com/display/GraphDB6/Home).

EUF stands for 'Entity Updates Feed'. This plug-in publishes entity update feeds that are used by the coordinator.

### Configuration

To install the EUF plug-in in GraphDB:
1. Provide the following Java parameter to GraphDB on startup `-Dregister-external-plugins=/your/plugins/home`.
2. Unpack the [EUF plug-in zip. file](http://maven.ontotext.com/content/repositories/publishing-releases/com/ontotext/ces/graphdb-euf-plugin/1.0.0/graphdb-euf-plugin-1.0.0.zip) in your plugins home prior to starting GraphDB.
