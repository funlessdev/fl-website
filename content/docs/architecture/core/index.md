+++
title = "Core"
description = "The Core component."
date = 2024-05-06T08:20:00+00:00
updated = 2021-05-01T08:20:00+00:00
draft = false
weight = 2
sort_by = "weight"
template = "docs/page.html"
+++

The Core plays two separate roles in FunLess' architecture:
- It's the main entrypoint, that is, it handles all requests coming from the outside
- It's the platform's central scheduler, and dispatches each function invocation to an appropriate worker

In a way, communication within FunLess can be separated according to these roles: it either goes from the user to the Core, or from the Core to the Workers.

### Core as the Entrypoint

The Core contains an HTTP server, developed using the [Phoenix](https://www.phoenixframework.org/) framework. It exposes a REST API, which allow users to perform CRUD operations on several [entities](../entities/)[^1].

When a function is created, the Core also forwards its code to all Workers, to mitigate potential cold starts caused by network delays. Workers are also notified when functions are updated (prompting them to replace their version of the code with a new one) or deleted (prompting them to remove the function's code from their local storage).

### Core as the Scheduler

#### Policies

<!-- The Core is the controller of the platform. It exposes an HTTP
REST API to the users, handles authentication and authorization,
and manages functions’ lifecycle and invocations.
Although the Core implements the main coordination logic
and functionalities of FunLess, it is a lightweight component. For
instance, on a Raspberry Pi 3B+ its local bare-metal deployment
(that includes the database, the monitoring system and the
underlying operating system and services) occupies ca. 600 MB
of RAM when idle.
Functionality-wise, FunLess users create a new function by
compiling its source code to Wasm—using either the language’s
default compiler (for Rust), an alternative one (for Go), or an external
tool (for JavaScript)—and uploading the resulting binary to the
Core, assigning to it a name. Users can group functions in modules
and, when uploading a function, they can optionally specify which
module the function belongs to. Moreover, users should also specify
the amount of memory reserved for the execution of the function.
Looking at the steps reported in Fig. 1, once the Core receives
the request to create a function (1. Upload), it stores its binary in
the database (2. Store). Fetch, update, and deletion happen via the
assigned function name. When the Core successfully creates a
function, it notifies the Workers (3. Broadcast) to store a local copy
of the function binary (4. Cache) compiled from the source code
with the given metadata (i.e., module and function names). This
push strategy helps to reduce part of the overhead of cold starts.
Indeed, most FaaS platforms follow a pull policy where, if the
execution nodes do not have the function in their cache (e.g., it is
the first time they execute), they fetch, cache, and load the code
of the function, undergoing latency. The small occupancy of Wasm
binaries makes it affordable for FunLess to employ a push strategy,
helping to reduce cold-start overheads.
Since both the Core and the Workers run on the BEAM, these
components communicate via the BEAM’s built-in lightweight
distributed inter-process messaging system, avoiding the need
(complexity, weight) for additional dependencies for data formatting,
transmission, and component connection.
When a function invocation reaches the Core (5. Invoke), the
latter checks the existence of the function in the database and
retrieves its code (6. Retrieve). If the function is present in the
database, the Core uses the most recent metrics—we represent the
pushing of the data, updated every 5s by default, from Prometheus
to the Core with the dashed line in Fig. 1—to select on which of
the available Workers to allocate the function (7. Request). The
selection algorithm starts from the Worker with the largest amount
of free memory to the one with the smaller. If no worker has enough
memory to host the function, the invocation will return with an error.
After the Worker successfully ran the function (we detail this part
of the workflow in the section about Workers, below) it sends back
to the Core the result (if any), which the Core relays back to the
user (10a/13b. Reply). If no Worker is available at scheduling time
or there are errors during the execution, the Core returns an error.
Another important feature of FunLess is that the Core can
automatically discover the Workers in its same network. This feature
derives from Elixir’s libcluster library8, which provides a mechanism
for automatically forming clusters of BEAM/Erlang nodes. Techni-
cally, when deployed on bare metal, FunLess follows the Multicast
UDP Gossip algorithm of the library, to automatically find available
workers. Instead, when deployed using Kubernetes, FunLess relies
on the service discovery capabilities of the container orchestration
engine to connect the Core with the Workers, paired with the “Kuber-
netes” modality of the library. Users can manually connect Workers
from other networks via a simple message (e.g., a ping) thanks to
the BEAM’s built-in capability of connecting to other BEAM nodes. -->

---

[^1]: Currently functions, modules and configuration scripts.