---
layout: news_item
title: "EUF for Java 7"
date: "11 Nov 2014"
author: stefan-enev
version: 1.0.2
categories: [release]
---

## EUF for Java 7

* Dependency cleanup, only external dependency now is commons-lang;
* Split build from CES;
* Changed the fingerprint calculation, see <a href="https://jira.ontotext.com/browse/FT-404">FT-404</a>;
* Now the fingerprint is never 0 or unbound but is always calculated the first time the plugin is run, see again <a href="https://jira.ontotext.com/browse/FT-404">FT-404</a>;
* Compile target is now 1.7.
