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

### As the entrypoint

The Core contains an HTTP server, developed using the [Phoenix](https://www.phoenixframework.org/) framework. It exposes a REST API, which allow users to perform CRUD operations on several [entities](../entities/)[^1].

When a function is created, the Core also forwards its code to all Workers, to mitigate potential cold starts caused by network delays. Workers are also notified when functions are updated (prompting them to replace their version of the code with a new one) or deleted (prompting them to remove the function's code from their local storage).

### As the scheduler

The Core also acts as the scheduler of the platform, that is, it forwards each invocation request to a suitable worker.

To give a very simple description of the workflow of an invocation:

1. The Core receives a POST request to invoke a certain function
2. The function's definition is retrieved from the database
3. The Core retrieves all known Workers in the cluster
4. The Core retrieves metrics for all these Workers
5. The metrics are used to select the Worker that will receive the function, according to a specific [policy](../entities/) 

Since step 4 might fail (if, e.g., no metrics have ever been retrieved for any Worker ever because of network issues), if no metrics are retrieved, a random Worker is selected, regardless of the scheduling policy. This is of course a last resort to avoid invocation failure, and might be subject to change in the future (if we find invocation failure to be a more reliable/sound approach).

The default policy, as of right now, is to select the Worker with the highest amount of memory available.

#### Policies

FunLess is built to accept different scheduling policies, depending on the associated configuration to each function. This means that it's relatively easy to extend the Core with a new policy, provided that an implementation is provided for the `SchedulingPolicy` protocol (and therefore, an appropriate data type for the relevant configuration).

Policies are simply functions with the following signature:

$$
(t, [Worker], Function) \rightarrow Worker
$$

Where \\(t\\) is the configuration type, and \\([Worker]\\) means a list of "enriched" Workers. Therefore, a policy simply returns a Worker, given a configuration, the available nodes, and the function being invoked.

---

[^1]: Currently functions, modules and configuration scripts.