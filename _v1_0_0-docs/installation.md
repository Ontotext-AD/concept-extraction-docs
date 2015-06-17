---
title: Installation
layout: default
prev_section: quick-start
next_section: ces_components
category: Getting Started
permalink: v1_0_0-docs/installation/
---
## General prerequisites

* [Java 8](http://www.oracle.com/technetwork/java/javase/downloads/index.html);
* Credentials to our Nexus publishing repositories;
* GATE pipeline using the proprietary Ontotext Dynamic Gazetteer.

## Standard (and easy) setup

1. Unzip the content of your GATE pipeline in a directory (for the purpose of this guide - `/home/user/pipeline`).
2. Install a web application container. If you do not have one, you can use [Apache Tomcat 7](http://tomcat.apache.org/download-70.cgi).
3. Set a few JVM parameters. In Tomcat, this is done from `/apache-tomcat/bin/setenv.sh`. See the <a href="{{ site.baseurl }}/v1_0_0-docs/ces_components">worker configuration</a> section.
4. Download [extractor-web.war](http://maven.ontotext.com/content/repositories/publishing-releases/com/ontotext/ces/extractor-web/1.0.1/extractor-web-1.0.1.war).
5. Start your `webapp` container.
6. Deploy the `war` you have just downloaded. In Tomcat, move it to its `/webapps` sub-directory where it will be picked up.
7. Now go to [http://localhost:8080/extractor-web/apidocs](http://localhost:8080/extractor-web/apidocs) for the live documentation.

<div class="note-badge">

Due to the <a href="https://helloreverb.com/developers/swagger">Swagger</a> limitations, the most important endpoint, namely <code>extract</code>, cannot have a live documentation. Therefore, it is explained <a href="{{ site.baseurl }}/v1_0_0-docs/annotating_content">here</a>.
</div>


## High-availability setup

The high-availability setup architecture includes several components, which communicate through RESTful calls. Each of them has its own role in the environment:

* _GraphDB with EUF (Entity Update Feed) plug-in_ - maintains a semantic database, which contains the RDF data used in the system. The EUF plug-in is responsible for providing notifications about every entity (concept) in the database that has been modified (added, removed, or edited).

* _Concept Extraction API Coordinator_ - accepts annotation requests and dispatches them to a group of concept extraction workers. It communicates with the semantic database in order to track changes leading to updates in every worker's dynamic gazetteer.

* _Concept Extraction API Worker_ - evaluates the annotation requests. It maintains a pool of GATE pipeline instances, used for text analysis and concept extraction.

### Installing GraphDB and the EUF plugin

Go to the official [GraphDB documentation](http://graphdb.ontotext.com/display/GraphDB6/Home) to learn how to install and use the GraphDB semantic database.

In order to install the EUF plug-in, check the <a href="{{ site.baseurl }}/v1_0_0-docs/ces_components">CES Components section</a>.

<div class="note-badge">

(optional): To activate the EUF plug-in, insert a single random statement with <code>rdfs:label</code> as predicate.
</div>

### Setting up a Coordinator

In this example, Apache Tomcat is used as a web application container.

<ol>
<li>Download the [Coordinator web application](http://maven.ontotext.com/content/repositories/publishing-releases/com/ontotext/ces/coordinator/1.0.1/coordinator-1.0.1.war) from our Nexus instance.</li>
<li>Add the coordinator-specific parameters to the Tomcat setup. Use the <pre><tomcat-home>/bin/setenv.sh</pre> file. For example:</li>

<pre><code>
coordinator setenv.sh

#!/bin/bash
# general options -- name, storage directory location, URL (required)
export GENERAL_OPTS="-Dcoordinator.name=master -Dcoordinator.stateDirectory=/path/to/storage/dir/coordinator -Dcoordinator.baseUrl=http://the.base.url:7070/coordinator"

# sparql endpoint options -- location
export ENDPOINT_OPTS="-Dcoordinator.sparql.endpoint=http://sparql.endpoint.be:8080/graphdb/repositories/my-repo"

# VM options -- heap size, etc.
export JVM_OPTS="-XX:+UseConcMarkSweepGC -XX:+TieredCompilation -Xmx1g"

export CATALINA_OPTS="$GENERAL_OPTS $ENDPOINT_OPTS $JVM_OPTS"
</code></pre>

<li>Deploy the Coordinator web application in Tomcat's `webapps` directory.</li>
<li>(Re-)start the Tomcat instance.</li>
</ol>

<div class="info-badge">
For more information about all Coordinator's configuration parameters, see <a href="{{ site.baseurl }}/v1_0_0-docs/ces_components">here</a>.
</div>

### Setting up a Worker node

In this example, Apache Tomcat is used as a web application container.

<ol>
<li>Download the [CES Worker web application](http://maven.ontotext.com/content/repositories/publishing-releases/com/ontotext/ces/extractor-web/1.0.1/extractor-web-1.0.1.war) from our Nexus instance.</li>
<li>Add the worker-specific parameters to the Tomcat setup. Use the `<tomcat-home>/bin/setenv.sh` file. For example:</li>

<pre><code>
worker setenv.sh
#!/bin/bash

# garbage collection and compiler
export J_OPTS="-XX:+UseConcMarkSweepGC -XX:+TieredCompilation -Xmx4g"

# worker options - path to pipeline, size of pipeline pool
export W_OPTS="-Dworker.name=st-worker -Dgate.app.location=file:/path/to/pipeline/application.xgapp -Dpipeline-pool-max-size=2"

export CATALINA_OPTS="$J_OPTS $W_OPTS"
</code></pre>

<li>Deploy the `extractor-web.war` in Tomcat's `webapps` directory.</li>
<li>(Re-)start the Tomcat instance.</li>
</ol>

<div class="info-badge">
For more information about all Worker's configuration parameters, see <a href="{{ site.baseurl }}/v1_0_0-docs/ces_components">here</a>.
</div>

### Adding a Worker node to the Coordinator

<div class="info-badge">
Assumptions:

<ul>
<li> the coordinator instance is located at http://coordinator.url:7070/coordinator;</li>
<li> the worker instance is located at http://worker.url:6060/worker;</li>
<li> the worker instance has a pipeline pool with size 2.</li>
</ul>
</div>

Using a REST client, execute the following request to the *coordinator* instance (`http://coordinator.url:7070`):

<pre><code>
POST /coordinator/workers
Content-type: application/json

[{"capacity":2, "url":"http://worker.url:6060/worker"}]
</code></pre>

* `capacity` is the number of pipeline instances in the worker pool;
* `url` is the location of the worker instance.

<div class="note-badge">
Instead of a REST client, one could use the Coordinator's Swagger Documentation endpoint, located at http://coordinator.url:7070/coordinator/apidocs and the specific <code>POST</code> and <code>PUT</code> requests.
</div>
