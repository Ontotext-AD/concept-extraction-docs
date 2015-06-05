---
layout: news_item
title: "CES 1.3.0"
date: "2 June 2015"
author: itrend
version: 1.3.0
categories: [release]
---

## CES 1.3.0

This release is focused around the support for 2 (or more) independent coordinators in order to avoid single point of failure. It also features a few new web endpoints, which are handy for cluster management.

### New features

* [FT-516](http://jira.ontotext.com/browse/FT-516): coordinator call to set dictionary queries on all workers
* [FT-526](http://jira.ontotext.com/browse/FT-526): support for two (or more) independent coordinators
* [FT-549](http://jira.ontotext.com/browse/FT-549): ability to enable and disable checks for models

### Improvements

* [DSP-750](http://jira.ontotext.com/browse/DSP-750): root pom update

### Bug fixes

* [DSP-813](http://jira.ontotext.com/browse/DSP-813): compression fix in coordinator-worker communication
