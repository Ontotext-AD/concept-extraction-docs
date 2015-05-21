---
title: Installation
layout: default
prev_section: quick-start
next_section:
category: Getting Started
permalink: v1_0_0-docs/installation/
---
## General prerequisites

* [Java 8](http://www.oracle.com/technetwork/java/javase/downloads/index.html)
* Credentials to our Nexus publishing repos

## Standard (and easy) setup

1. Get a Semantic Pipeline and unzip it to a directory. For the purpose of this guide, we'll assume you have your pipeline contents unpacked in `/home/user/pipeline`.
2. Install a web application container. If you already have one - great, if not - we use [Apache Tomcat 7](http://tomcat.apache.org/download-70.cgi).
3. You will need to set a few JVM params, in Tomcat this is done from `/apache-tomcat/bin/setenv.sh`. See the <a href="{{ site.baseurl }}/v1_0_0-docs/ces_components">worker configuration page</a>.
4. Download [extractor-web.war](http://maven.ontotext.com/content/repositories/publishing-releases/com/ontotext/ces/extractor-web/1.0.1/extractor-web-1.0.1.war).
5. You can now start your webapp container.
6. Deploy the war you just downloaded. In Tomcat you simply need to move it to its `/webapps` sub-directory and it will get picked up.
7. Now go to http://localhost:8080/extractor-web/apidocs for live documentation.

<div class="note-badge">
Due to <a href="https://helloreverb.com/developers/swagger">Swagger</a> limitations, the most important endpoint, namely <code>/extract</code>, cannot have live documentation. This is why it's explained <a href="{{ site.baseurl }}/v1_0_0-docs/annotating_content">here</a>.</div>

## High-availability setup

The high-availability setup architecture includes several components, communicating through RESTful calls. Each component has its own role in the environment. Here is a list with brief explanations for each module:

* *GraphDB with EUF plugin* -- the GraphDB module maintains a semantic database, containing RDF data used within the system. Its EUF plugin (EUF stands for _Entity Update Feed_) is responsible for providing the outer world with notifications about every entity (concept) within the database that has been modified in any way (added, removed, edited).
* *Concept Extraction API Coordinator* -- the Coordinator module accepts annotation requests and dispatches them towards a group of Concept Extraction Workers (see below). The Coordinator communicates with the semantic database in order to track for changes leading to updates in every Worker's Dynamic Gazetteer.
* *Concept Extraction API Worker* -- the Worker module evaluates annotation requests. It maintains a pool of GATE pipeline instances, used for text analysis and concept extraction.

### Installing GraphDB and the EUF plugin

Information about installing and using the GraphDB semantic database can be found on the official [GraphDB documentation](http://graphdb.ontotext.com/display/GraphDB6/Home).

In order to install the Entity Update Feed plugin, check the <a href="{{ site.baseurl }}/v1_0_0-docs/ces_components">CES Components</a>.

<div class="note-badge">
OPTIONAL: insert a single random statement having <code>rdfs:label</code> as predicate in order to activate the EUF plugin.
</div>

### Setting up a Coordinator

We use Apache Tomcat as a web application container for this example.

1. Download the [Coordinator web application](http://maven.ontotext.com/content/repositories/publishing-releases/com/ontotext/ces/coordinator/1.0.1/coordinator-1.0.1.war) from our Nexus instance.
2. Add the coordinator-specific parameters to the Tomcat setup -- use the `<tomcat-home>/bin/setenv.sh` file. Example:
<pre><code>
coordinator setenv.sh
# !/bin/bash

# general options -- name, storage directory location, URL (required)
export GENERAL_OPTS="-Dcoordinator.name=master -Dcoordinator.stateDirectory=/path/to/storage/dir/coordinator -Dcoordinator.baseUrl=http://the.base.url:7070/coordinator"

# sparql endpoint options -- location
export ENDPOINT_OPTS="-Dcoordinator.sparql.endpoint=http://sparql.endpoint.be:8080/graphdb/repositories/my-repo"

# VM options -- heap size, etc
export JVM_OPTS="-XX:+UseConcMarkSweepGC -XX:+TieredCompilation -Xmx1g"

export CATALINA_OPTS="$GENERAL_OPTS $ENDPOINT_OPTS $JVM_OPTS"
</code></pre>

3. Deploy the coordinator web application in Tomcat's `webapps` directory.
4. (Re-)start the Tomcat instance.

<div class="info-badge">
For more information about all Coordinator configuration parameters, check <a href="{{ site.baseurl }}/v1_0_0-docs/ces_components">CES Components</a>.
</div>


### Setting up a Worker node

We use Apache Tomcat as a web application container for this example.

1. Download the [CES Worker web application|http://maven.ontotext.com/content/repositories/publishing-releases/com/ontotext/ces/extractor-web/1.0.1/extractor-web-1.0.1.war] from our Nexus instance.
2. Add the worker-specific parameters to the Tomcat setup -- use the `<tomcat-home>/bin/setenv.sh` file. Example:
<pre><code>
worker setenv.sh
# !/bin/bash

# garbage collection and compiler
export J_OPTS="-XX:+UseConcMarkSweepGC -XX:+TieredCompilation -Xmx4g"

# worker options -- path to pipeline, size of pipeline pool
export W_OPTS="-Dworker.name=st-worker -Dgate.app.location=file:/path/to/pipeline/application.xgapp -Dpipeline-pool-max-size=2"

export CATALINA_OPTS="$J_OPTS $W_OPTS"
</code></pre>
6. Deploy the `extractor-web.war` in Tomcat's `webapps` directory.
7. (Re-)start the Tomcat instance

<div class="info-badge">
For more information about all Worker configuration parameters, check <a href="{{ site.baseurl }}/v1_0_0-docs/ces_components">CES Components</a>.
</div>

### Adding a Worker node into the Coordinator

<div class="note-badge">
Assumptions:

<ul>
<li>the <code>coordinator</code> instance is located at http://coordinator.url:7070/coordinator.</li>
<li>the <code>worker</code> instance is located at http://worker.url:6060/worker.</li>
<li>the <code>worker</code> instance has a pipeline pool with size 2.</li>
</ul>
</div>

Using a REST client, execute the following request to the coordinator instance (`http://coordinator.url:7070`):

<pre><code>
POST /coordinator/workers
Content-type: application/json

[{"capacity":2, "url":"http://worker.url:6060/worker"}]
</code></pre>

* <code>capacity</code> is the number of pipeline instances in the worker pool.
* <code>url</code> is the location of the worker instance.

<div class="note-badge">
Instead of a REST client, one could use the Coordinator' Swagger Documentation endpoint, located at `http://coordinator.url:7070/coordinator/apidocs` and the specific <code>POST</code> or <code>PUT</code> requests.
</div>
