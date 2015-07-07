---
layout: default
title: Home
---

This site aims to be a comprehensive guide to the Ontotext Concept Extraction API. We will cover topics such as getting the engine up and running, ingesting content into the system, logging user actions, tuning the recommendation algorithm, scaling, and deployment.

## Who is this site for?

This is a technical documentation, written to be used by technical people. Whether you are an architect, evaluating how this recommendation engine fits as a component to your system, or you are a developer, who has already integrated it and actively employs its features - this is the complete reference. We also try to provide as much as possible insight into what we are currently working on and our future ideas via the roadmap page, and will keep you posted with news about releases and upgrade paths.

## What is the Concept Extraction API?

The Concept Extraction Service (CES) is a stand-alone service, which allows you to handle large amounts of parallel semantic annotation requests while providing high availability.
It uses data loaded from the GraphDB database as its dictionary of concepts (knowledge base) to recognise mentions of entities, as well as their relevance and algorithm's confidence, on the text. Its integration with GraphDB,
 provides a live stream of dictionary updates to the service as they enter the knowledge base.
Since the concept extraction applications are usually based on a mixture of machine learning routines and rules, CES also provides a Re-training API which allows both manual and automated re-training and evaluation of the underlying statistical models (for example, relevance and confidence criteria are updated automatically over time to adjust to new data).

### Architecture features:

As the Concept Extraction Service is wired to GraphDB through a plug-in called Entity Update Feed (EUF), its dictionaries are dynamically updated and are in near real time synchronization (aiming for eventual consistency) with the data in the semantic repository. It also provides high availability, load balancing and fail-over capabilities. Horizontal scaling also works in the same way as in GraphDB. Moreover, the pipelines running on a single cluster node could be pooled in the way so that multiple pipeline instances share the same dictionary and achieve more semantic annotation request throughput while optimizing memory usage. The pools are also capable of dynamic expanding with respect to the current load. Communication with the service is established over HTTP via RESTful APIs, which makes it easily integrable with various platforms written in different programming languages.

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
