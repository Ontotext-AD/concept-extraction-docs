---
layout: news_item
title: "CES Cluster with Dynamic dictionary updates"
date: "21 Oct 2014"
author: stefan-enev
version: 1.0.0
categories: [release]
---
## CES Cluster with Dynamic dictionary updates

### HA Cluster

The master/slave architecture of the cluster allows it to scale horizontally by introducing more workers, and also ensures availability. We call the master 'Coordinator' and the slave nodes 'Workers'.

#### Coordinator

* Load balances `/extract` requests;
* Load balances dictionary updates;
* Manages update feeds;
* Manages worker dictionary queries.

#### Worker
* Keeps track of its dictionaries fingerprint;
* Creates a pool of pipelines;
* Serves extraction requests;
* Updates its Gazetteer and Metadata PR SPARQL queries;
* Reloads the whole pipeline dictionary.

#### Dynamic updates

* Multiple retries on failed update; full dictionary reload is triggered after a limit is  reached (configurable);
* The EUF plug-in keeps track of updated entities and serves changelists through special SPARQL queries;
* Keep track of updates via GraphDB's fingerprints.
