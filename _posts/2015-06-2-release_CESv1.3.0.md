---
layout: news_item
title: "CES 1.3.0"
date: "2 June 2015"
author: itrend
version: 1.3.0
categories: [release]
---

## CES 1.3.0

This release is focused around the support for 2 (or more) independent coordinators in order to avoid a single point of failure. It also features a few new web endpoints that are handy for cluster management.

### New features

* [FT-516](http://jira.ontotext.com/browse/FT-516): A coordinator call to set dictionary queries on all workers;
* [FT-526](http://jira.ontotext.com/browse/FT-526): Support for two (or more) independent coordinators;
* [FT-549](http://jira.ontotext.com/browse/FT-549): Ability to enable and disable checks for models.

### Improvements

* [DSP-750](http://jira.ontotext.com/browse/DSP-750): `root` POM update.

### Bug fixes

* [DSP-813](http://jira.ontotext.com/browse/DSP-813): A compression fix in the coordinator-worker communication.
