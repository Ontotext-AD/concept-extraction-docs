---
layout: default
title: Annotation Load Balancing
prev_section:
next_section:
category: HowTo's
permalink: v1_0_0-docs/load_balancing/
---

## Annotation Load Balancing

### Worker capacity

The most important characteristic of a worker with respect to annotation load balancing is its capacity - a number specifying how
many simultaneous annotation requests the worker can handle. This number is configured once on the worker and then again for every
coordinator that should know about the worker.

On-the-worker capacity is set through `-Dpipeline-pool-max-size` property (TODO: do I use `-D` here, or just the property name?).
This setting is the actual number of maximum simultaneous annotations the worker can handle at any given time. Any pending requests
while the worker is at full capacity are enqueued and will be handled as soon as running annotations finish.

On-the-coordinator capacity for a worker is set when adding the worker to the coordinator by the `capacity` worker property (e.g.
```
POST /coordinator/workers
Content-type: application/json

[{
    "worker": "http://url-to-worker",
    "capacity": 3
}]
```). It defines how many simultaneous requests this coordinator will try to enqueue on the worker. If this number is greater than
the actual worker capacity, the coordinator will send more annotation requests than the worker can handle, extra requests will wait
for running ones to finish. If the on-coordinator capacity is less than the actual worker capacity - the worker will be guaranteed
to have some free annotation slots. This might be useful if you would like to have two (or more) coordinators that will both be
in heavy use - since coordinators do not communicate between themselves, one coordinator cannot know how many annotation requests 
have been enqueued by other coordinators - therefore, having some "spare" annotation slots on the worker might be useful in those
cases.

### Load balancing under the hood

The coordinator distributes annotation requests by maintaining a queue of free "annotation slots" for each worker. For example, 
let's say we add two workers `W1` with capacity 3 and `W2` with capacity 4 (these are on-the-coordinator capacities - actual 
workers capacity don't matter to the load balancer). The load balancing queue will look similar to this after workers have been 
added
```
     +--+--+--+--+--+--+--+
head |W2|W1|W2|W1|W2|W1|W2| tail
     +--+--+--+--+--+--+--+
```
Each square represents a free annotation slot on the respective worker. When an annotation request arrives at the coordinator, it 
picks the annotation slot at the head, `W2` in this case and forwards the request to it. After the worker finishes annotating,
the slot is pushed back at the tail of the queue (assuming no other requests have arrived in the meanwhile):
```
     +--+--+--+--+--+--+--+
head |W1|W2|W1|W2|W1|W2|W2| tail
     +--+--+--+--+--+--+--+
```
If the queue is empty when a request arrives, the coordinator waits a bit (as configured by 
`-Dcoordinator.annotation.freeWorkerTimeout` parameter) for a free slot to appear in the queue. In the case a worker doesn't 
become free during the specified timeout, `503 Service Unavailable` response is returned back to the client.

### Peak loads - fail-fast vs wait

When the number of simultaneous annotation requests is a lot greater than the sum of all worker capacities, coordinator load
balancing queues will be empty for long periods of time. Configuring small `freeWorkerTimeout` will cause a lot of the "extra"
requests to fail and it's the client's responsibility to retry the request. Configuring large `freeWorkerTimeout` will cause
extra requests to pile up on the coordinator, consuming resources (heap and thread count) and possibly causing out of memory
exceptions and reaching thread count limits (in extreme cases). Which behavior is more appropriate depends on the client's needs.
