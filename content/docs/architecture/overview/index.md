+++
title = "Overview"
description = "High-level overview of FunLess and its components."
date = 2024-05-06T08:20:00+00:00
updated = 2021-05-01T08:20:00+00:00
draft = false
weight = 1
sort_by = "weight"
template = "docs/page.html"
+++

### FunLess' skeleton

The architecture of FunLess consists of several components:

- The [Core](./../core), acting both and as the platform's entrypoint and scheduler. The Core handles all incoming HTTP requests and forwards each invocation to an available node[^1].
- The [Workers](./../worker), acting as function executors. Each Worker embeds a WebAssembly runtime and is responsible for running functions and recovering their results. Workers communicate with the Core to handle invocation requests. Moreover, each Worker holds the code for several functions in a local cache, reducing cold starts.
- A database (currently [Postgres](https://www.postgresql.org/)), holding all function information[^1]. The database is co-located on the same node as the Core, to minimise communication latency between the two[^2]. 
- A monitoring system (currently [Prometheus](https://prometheus.io/)), scraping metrics from the Workers. The Core periodically queries the monitoring system to retrieve updated information on each Worker's resource usage and status. Like the database, this component is also co-located with the Core[^2].


### Life of a request

A very simple representation of how FunLess processes requests can be seen in the following diagram:

<img class="light-img" alt="Diagram of the FunLess architecture. Contains both components and data-flow." src="img/architecture_diagram_light.png">

<img class="dark-img" alt="Diagram of the FunLess architecture. Contains both components and data-flow." src="img/architecture_diagram_dark.png">

Following the diagram, a function creation requests generally consists of four steps:

1. The user sends a request to the Core to create a function
2. The function is stored in the database
3. The Core sends the function's code to all connected Workers
4. Each Worker saves the code as-is in its local storage (whether in-memory or on a persistent medium)[^3]

An invocation requests, on the other hand, follows this path:

5. The user sends an invocation request for a function
6. The Core retrieves the function definition from the database
7. The Core selects a Worker to run the function, and forwards to it the invocation request
8. The Worker searches its cache for the function's code

9. At this point, the execution branches in two different cases:
    - 9a: The function's code is found. The Worker runs the function, and returns its result to the Core. The workflow terminates here.
    - 9b: The function's code is not found locally. The Worker sends a "no code" message to the Core

If the "b" path was taken in step 9, the request proceeds:

10. The Core sends the same request as before to the Worker, adding the function's code this time
11. The Worker caches the function's code, after a pre-compilation phase[^3] (more on that on the [Worker](../worker) description)
12. The Worker runs the function, and returns its result to the Core. The workflow terminates here. 




<!-- Looking at the steps reported in Fig. 1, once the Core receives
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
or there are errors during the execution, the Core returns an error. -->

---

[^1]: At the time of writing, FunLess only supports single-Core deployments. Support for multiple cores is being worked on. In such deployments, both the database and the metric system would probably be replicated and co-located with each Core instance.

[^2]: FunLess does not currently support multi-tenancy, therefore the database only contains information about functions and modules. User information will be handled generally in the same way.

[^3]: For simplicity, the distinction between storing the code as-is and cache the pre-compiled code is not shown in the diagram. In case the pre-compiled code is not found, but the "raw" code is already in storage, the Worker simply performs the pre-compilation step and caches the result, without talking to the Core. The "raw" code is never used for function invocation, only the pre-compiled version is.