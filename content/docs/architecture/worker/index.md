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