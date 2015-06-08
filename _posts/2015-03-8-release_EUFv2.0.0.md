---
layout: news_item
title: "EUF 2.0.0"
date: "8 March 2015"
author: stefan-enev
version: 2.0.0
categories: [release]
---

## EUF 2.0.0

## Breaking changes

Instead of using Java serialisation to fully serialise all updates from the beginning, in each transaction we now do the following:

* Append an entry at the end of the data file;
* The entry looks like this - short (magic number marker) - int (updated subjects count) - long (fingerprint) - longs (updated subjects).

In case the transaction is aborted, which is rare during normal operation, we delete the plug-in data file and dump the whole history without the changes in the aborted transaction. This can be further optimised in future releases.

## Migration from 1.x.x

1. Stop the GraphDB worker.
2. Delete the plugin directory from the repository - `/repositories/${repo.name}/storage/euf`.
3. Deploy version 2.x.x of the plugin to the path where -`Dregister-external-plugins` points to.
4. Start the GraphDB worker.
