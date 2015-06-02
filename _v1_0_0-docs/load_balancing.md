---
layout: default
title: Annotation Load Balancing
prev_section: dictionary_reload
next_section: troubleshooting
category: HowTo's
permalink: v1_0_0-docs/load_balancing/
---

## Annotation Load Balancing

### Worker capacity

The most important characteristic of a worker with respect to annotation load balancing is its capacity - a number specifying how many simultaneous annotation requests the worker can handle. This number is configured once on the worker and then again for every coordinator that should know about the worker.

On-the-worker capacity is set through `-Dpipeline-pool-max-size` property, as described in the <a href="{{ site.baseurl }}/v1_0_0-docs/ces_components">worker configuration section</a>.
This setting is the actual number of maximum simultaneous annotations the worker can handle at any given time. Any pending requests while the worker is at full capacity are enqueued and will be handled as soon as thread(s) in the pipeline pool are released by the currently running annotation job(s).

On-the-coordinator capacity for a worker is usually set when adding the worker to the coordinator (and can be updated later) by the `capacity` worker property e.g.
<pre></code>POST /coordinator/workers
Content-type: application/json

[{
    "worker": "http://sample.worker.url",
    "capacity": 3
}]</code></pre>

It defines how many simultaneous requests this coordinator will try to enqueue on the worker. If this number is greater than the actual worker capacity, the coordinator will send more annotation requests than the worker can handle and extra requests will wait for running ones to finish. If the on-coordinator capacity is less than the actual worker capacity, the worker will be guaranteed
to have some free annotation slots. This is useful when you have two (or more) coordinators
in use. Since coordinators do not communicate between themselves, one coordinator cannot know how many annotation requests have been enqueued by other coordinators. Therefore, having some "spare" annotation slots on the worker ensures that the workers are not overloaded.

### Load balancing under the hood

The coordinator distributes annotation requests by maintaining a queue of free "annotation slots" for each worker. For example, let's say we add two workers `W1` with capacity 3 and `W2` with capacity 4 (these are on-the-coordinator capacities - actual workers' capacity does not matter to the load balancer). The load balancing queue will be similar to this, after the workers have been added.

<pre><code>     +--+--+--+--+--+--+--+
head |W2|W1|W2|W1|W2|W1|W2| tail
     +--+--+--+--+--+--+--+</code></pre>

Each square represents a free annotation slot on the respective worker. When an annotation request arrives at the coordinator, it picks the annotation slot at the head, `W2` in this case, and forwards the request to it. After the worker finishes annotating, the slot is pushed back at the tail of the queue (assuming no other requests have arrived in the meantime):

<pre><code>     +--+--+--+--+--+--+--+
head |W1|W2|W1|W2|W1|W2|W2| tail
     +--+--+--+--+--+--+--+</code></pre>

If the queue is empty when a request arrives, the coordinator waits for a while (as configured by
`-Dcoordinator.annotation.freeWorkerTimeout` parameter) for a free slot to appear in the queue. In the case a worker does not become free during the specified timeout, `503 Service Unavailable` response is returned back to the client.

### Peak loads - fail-fast vs wait-and-see

Since workers can block indefinitely, this parameter effectively controls whether a client's annotation job fails fast when there are no resources available or is eventually acted upon.

When the number of simultaneous annotation requests is a lot greater than the sum of all worker capacities, the coordinator load balancing queues will be empty for long periods of time. Configuring small `freeWorkerTimeout` will cause a lot of the "extra" requests to fail and it is the client's responsibility to retry the request. Configuring large `freeWorkerTimeout` causes the extra requests to pile up on the coordinator, which will eventually distribute them when the load goes down. This, however, may lead to additional resources consumption (heap and thread count), which can eventually manifest in out of memory (OOM) exceptions or blocking due to exhausting the thread count limits (in extreme cases). Which behavior is more appropriate depends on the client's needs.
