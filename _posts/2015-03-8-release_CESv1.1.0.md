---
layout: news_item
title: "CES 1.1.0"
date: "8 March 2015"
author: stefan-enev
version: 1.1.0
categories: [release]
---
## CES 1.1.0

### New features

* <a href="https://jira.ontotext.com/browse/FT-430">FT-430</a> Query management through the Coordinator;
* <a href="https://jira.ontotext.com/browse/FT-417">FT-417</a> Utility endpoint to reload all workers;
* <a href="https://jira.ontotext.com/browse/FT-448">FT-448</a> A parameter for maximum number of missed SPARQL updates before full worker reload;
* <a href="https://jira.ontotext.com/browse/FT-449">FT-449</a> New functionality added for disabling and notifying when a worker fails to update its models numerous times in a row;
* <a href="https://jira.ontotext.com/browse/FT-455">FT-455</a> Options for using the Force to check for new models;
* <a href="https://jira.ontotext.com/browse/FT-450">FT-450</a> New functionality added to the Training API for evaluating models against different text corpora.

### Improvements

* <a href="https://jira.ontotext.com/browse/FT-435">FT-435</a> Counters in the Coordinator that distinguish between failure and success;
* <a href="https://jira.ontotext.com/browse/FT-436">FT-436</a> Coordinator does not ping workers that are already in use;
* <a href="https://jira.ontotext.com/browse/FT-459">FT-459</a> Manifest @Controller for the training-api & content-api.


### Bug fixes

* <a href="https://jira.ontotext.com/browse/FT-434">FT-434</a> Message converters in CES should throw 400 on bad input;
* <a href="https://jira.ontotext.com/browse/FT-456">FT-456</a> Model hashes null in Coordinator `GET /workers`;
* <a href="https://jira.ontotext.com/browse/FT-442">FT-442</a> `POST /coordinator/worker` does not update capacity of existing workers;
* <a href="https://jira.ontotext.com/browse/FT-440">FT-440</a> Coordinator gets confused when worker responds with 404.
