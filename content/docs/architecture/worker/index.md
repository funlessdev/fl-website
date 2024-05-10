+++
title = "Worker"
description = "The Worker component."
date = 2024-05-06T08:20:00+00:00
updated = 2021-05-01T08:20:00+00:00
draft = false
weight = 3
sort_by = "weight"
template = "docs/page.html"
+++

The Worker is the main function executor of the FunLess platform. In a standard deployment, there normally are as many Workers as the total number of nodes (potentially minus one, if the Core node is not used as an executor).

Each Worker embeds a WebAssembly runtime (currently [Wasmtime](https://github.com/bytecodealliance/wasmtime)), to instantiate and run functions. Each function is a WebAssembly module, with the addition of the several imports/exports (see below).



### FunLess-compatible WebAssembly modules

As of right now, standard WebAssembly modules are not runnable by FunLess (and vice-versa, FunLess-compatible modules are not runnable by standalone runtimes), because of several requirements in terms of imported and exported functions. We list here the required imports/exports the module must satisfy.

To produce FunLess-compatible modules, we currently use language-specific wrappers, which can be found in a dedicated [repository](https://github.com/funlessdev/fl-wasm). This approach is subject to change and we plan on greatly improving the usability of the platform, by integrating the building process with our CLI and increasing compatibility with standard WebAssembly modules.

#### Imported functions

- `__get_input_data` asks the runtime to write the input data to memory. It takes a single integer as input, representing the pointer where the data will be written.
- `__insert_response` inserts the function's response in memory. It takes two integers as input, representing the pointer to the response, and its length.
- `__insert_error` same as `__insert_response`, used in case the function encountered an error.
- `__http_request` performs an HTTP request, with a JSON body and a JSON response. Takes several parameters, in the following order:
    - pointer to the response and pointer to its length, which will be populated by the runtime after the request is performed
    - pointer to the status, also populated by the runtime
    - method (an integer from 0 to 3, representing `GET`, `POST`, `PUT` and `DELETE` in this order)
    - pointer and length of the URL
    - pointer and length of the headers, codified as a string
    - pointer and length of the body
This function will probably be scrapped, as the WASI standard has recenly included HTTP requests and socket operations, so there will be no need for a custom import.
- `__console_log` is a debugging function, to be removed in the future (or at least disabled in production environments). Prints the given string (identified by two integers, a pointer to the message and its length) as a debugging log on the Worker. 

#### Exported functions

- `__invoke` is used by FunLess to actually run the module. This function basically extracts the input from memory using `__get_input_data`, runs the `main` function (or its equivalent) in the given language, and inserts the results in memory using `__insert_response` or `__insert_error`. `__invoke` takes a single integer as argument, representing the length of the function's input (generally stringified JSON).


When loading the input data, the general workflow is:
- initialize an array of length `N`, where `N` is the parameter passed to `__invoke`
- call `__get_input_data` with a pointer to the array
- parse the array as a string
- pass the string to the `main` function (the actual user-defined function)


`__invoke` also returns either 0 or 1, for success and failure respectively, after having called `__insert_response` (or `__insert_error`).

<!-- The Worker executes the functions requested by the Core. The
Workers employs Wasmtime, a standalone runtime for Wasm and
WASI by the Bytecode Alliance [23]. The main reasons behind this
choice come from the ease of integration, amount of contributors,
and security-oriented focus of the project. While Workers integrate
Wasmtime, we modelled their architecture to abstract away the
peculiarities of specific Wasm runtimes so that future variants can
use different runtimes and even extend support for multiple ones
(possibly letting users specify which one to use).
When a Worker receives a request from the Core to execute a
function (7. Request), it first checks whether it has a cached version
of the function’s binary (8. Retrieve). If that is the case, it loads
and runs the function’s binary and returns to the Core the result
of the computation (9a. Result). If the Worker does not find the
8 https://hexdocs.pm/libcluster/readme.html.code of the function in its local cache, it contacts the Core (9b.
No Code Message), which responds with a request that carries
the code of the function to the Worker (10b. Request with Code).
Upon reception, the Worker compiles the code, caches the binary
for future invocations (11b. Cache), loads it to run the function, and
relays the result to the Core (12b. Result).
The above mechanism is an important advantage afforded by
FunLess for the edge case. Function fetching (if needed) transmits
small pieces of binary code (rather than heavyweight containers).
Wasm binaries achieve the two-fold objective of having Workers
run functions on different hardware architectures (e.g., AMD64,
ARM) and allowing users to write their functions once, knowing
that they will execute irrespective of the hardware of the Worker.
Summarising, fetching and precompiling (if any, depending
on cache status) constitutes most of the “cold start” overhead in
FunLess, which the platform greatly reduces w.r.t. alternatives
relying on containers (which are heavier both in terms of bandwidth
and memory occupancy).
Regarding caching and eviction, Workers set a threshold for the
cache memory (configurable at deployment time). If the storing of a
new function exceeds that threshold, the Worker evicts the function
with the longest period of inactivity (invocation- or update-wise).
Additionally, Workers automatically evict functions if inactive for
a set amount of time (by default, 45 minutes). -->