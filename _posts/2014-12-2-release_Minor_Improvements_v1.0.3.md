---
layout: news_item
title: "Minor Improvements"
date: "2 Dec 2014"
author: stefan-enev
version: 1.0.3
categories: [release]
---
## Minor Improvements

### New features

* `GET /extract` with `url` parameter.

### Moved/deleted

* Generic-document-schema moved to <a href="https://github.com/Ontotext-AD/commons-pub">commons-pub</a>;
* `extractor-tag` deleted.

### Improvements

* Deployment infrastructure:
  * Updated Tomcat module (our own) with rolling, limited logs <a href="https://jira.ontotext.com/browse/DSP-485">DSP-485</a>;
  * Manifests for the internally hosted tagging service backend.
* Build infrastructure:
  * Updated `root` POM to <a href="http://maven.ontotext.com/service/local/repositories/internal/content/com/ontotext/parents/root/4.0.1/root-4.0.1.pom">4.0.1</a>.
* Codebase:
  * Trimming for ping document so it is logged on a single line instead of two;
  * Fixed Coordinator forwarding bug for `GET` requests.
