---
layout: default
title: Home
---

This site aims to be a comprehensive guide to the Ontotext Concept Extraction API. We will cover topics such as getting the engine up and running, ingesting content into the system, logging user actions, tuning the recommendation algorithm, scaling, and deployment.

## Who is this site for?

This is a technical documentation, written to be used by technical people. Whether you are an architect, evaluating how this recommendation engine fits as a component to your system, or you are a developer, who has already integrated it and actively employs its features - this is the complete reference. We also try to provide as much as possible insight into what we are currently working on and our future ideas via the roadmap page, and will keep you posted with news about releases and upgrade paths.

## What is the Concept Extraction API?

The Concept Extraction API is a stand-alone service, which ensures life-cycle automation, i.e., development, evaluation, management and maintenance, of semantic extraction application (SEA). It uses the data in the GraphDB database as its dictionary of concepts (knowledge base) to recognise mentions of entities, as well as their relevance and algorithm's confidence, on the text. It comes with an out-of-the-box default pipeline, which is configured to produce media graph basic concept look ups. The service supports multiple pools of SEA and provides an API to manage them (start/stop/reload/configure), which is very convenient in cases when more sophisticated semantic annotation is required and also when different text analysis techniques are used to span across multiple data domains. Semantic annotation is covered by special queries in the SPARQL API which retrieve the matched concepts and their features. Since the SEA are usually based on a mixture of machine learning routines and rules, SES also provides a Re-training API which allows both manual and automated re-training and evaluation of the underlying statistical models (for example, relevance and confidence criteria are updated automatically over time to adjust to new data).

### Architecture features:

As the Concept Extraction API is wired to GraphDB through its Plug-in API, its dictionaries are dynamically updated in a transactional fashion and are always in synchronization with the data in the semantic repository. It also provides high availability ensured by GraphDB's replication, load balancing and fail-over capabilities. Horizontal scaling also works in the same way as in GraphDB. Moreover, the pipelines running on a single cluster node could be pooled in the way so that multiple pipeline instances share the same dictionary and achieve more semantic annotation request throughput while optimizing memory usage. The pools are also capable of dynamic expanding with respect to the current load. Communication with the service is established over HTTP and the SPARQL protocol, which makes it easily integrable with various platforms written in different programming languages.

## Helpful hints

<div class="info-badge">
Info badges are handy pieces of information.
</div>

<div class="note-badge">
Notes are comments or references that may save you time and unnecessary effort.
</div>

<div class="warning-badge">
Warnings are pieces of advice that turn your attention to things you should be cautious about.
</div>
