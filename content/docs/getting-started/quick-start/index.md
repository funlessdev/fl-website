+++
title = "Quick Start"
description = "One page summary of how to start using FunLess."
date = 2021-05-01T08:20:00+00:00
updated = 2021-05-01T08:20:00+00:00
draft = false
weight = 20
sort_by = "weight"
template = "docs/page.html"

[extra]
lead = "One page summary on how to start using FunLess."
toc = true
top = false
+++

## Install the CLI

To start hacking with FunLess, all you need is the CLI tool: [fl](https://www.github.com/funlessdev/fl-cli). 

You can check out the [Releases page](https://github.com/funlessdev/fl-cli/releases) and download the most recent one based on your system
(Windows is not currently supported although there is a build for it).

Here are some quick links to download it:

- [Linux amd64](https://github.com/funlessdev/fl-cli/releases/download/v0.2.1/fl-v0.2.1-linux-amd64.tar.gz)
- [Linux arm64](https://github.com/funlessdev/fl-cli/releases/download/v0.2.1/fl-v0.2.1-linux-arm64.tar.gz)
- [Mac amd64](https://github.com/funlessdev/fl-cli/releases/download/v0.2.1/fl-v0.2.1-darwin-amd64.tar.gz)
- [Mac arm64](https://github.com/funlessdev/fl-cli/releases/download/v0.2.1/fl-v0.2.1-darwin-arm64.tar.gz)


Extract the executable from the archive and you can start using fl.


## Deploy FunLess Locally

⚠️ You need to have [Docker](https://docs.docker.com/get-docker/) installed for this! ⚠️

You can use the CLI tool to deploy the platform locally using Docker containers. 

The CLI will pull and launch 3 containers, one for the **Core** component, one for the **Worker** and one for **Prometheus**. 
It will handle the network between them and will remove everything when you are done.

```bash
fl admin dev
```

<img src="./img/fl_admin_dev.gif" style="width: 100%;" />

Now that FunLess is running, you can start deploying and running functions.

## Create a function

FunLess uses [WebAssembly](https://webassembly.org/) runtimes via [Wasmtime](https://wasmtime.dev/) to run your functions.
As of now we support *Rust* and *JavaScript*, but we are working on adding more languages.

The CLI tool already handles the compiling into WebAssembly for you, so you can focus on writing your function.

Let's try to create a simple function that returns the current time, using Javascript.

First of all create a new folder for our function, let's call it `fun_time`. Inside we can add an `index.js` file:

```js
// time_function/index.js
function handler() {
  return new Date().toISOString();
}

module.exports.fl_main = handler;
```

Remember to add a `fl_main` export to your function, this is the entry point that FunLess will use to call your function.

Now you can create the function. In the example we are naming it `time`:

```bash
fl fn create time --source-dir fun_time --language js
```

With `--source-dir` you are telling the CLI where to find the source code for the function, and `--language` is the language you are using.

With this simple function we are not using a `package.json`, but you can add one if you want to use dependencies.


<img src="./img/fl_create.gif" style="width: 100%;" />

## Invoke the function

Now you can invoke it. This function takes no arguments, so we can just call it:

```bash
fl fn invoke time
```

<img src="./img/fl_invoke.gif" style="width: 100%;" />

## Invoke with arguments

Let's create a function that takes a `name` argument, and returns a greeting. Create a folder named `hello` and add an `index.js` file:

```js
// hello/index.js
function fl_main(input) {
  let name = input.name || "World";
  return { body: `Hello ${name}!`}
}

module.exports.fl_main = fl_main;

```

As before, deploy the function:

```bash
fl fn create hello -d hello -l js
```

Note that you can use `-d` instead of `--source-dir` and `-l` instead of `--language` as shortcuts.

Now invoke it with the `-j` flag to pass a JSON object as input:

```bash
fl fn invoke hello -j '{"name": "FunLess"}'
```

<img src="./img/fl_invoke_args.gif" style="width: 100%;" />


## Delete the function

You can delete the function with:

```bash
fl fn delete hello
```

## Cleaning up

When you are done, you can remove the containers and cleanup the dev deployment with:

```bash
fl admin reset
```
<img src="./img/fl_admin_reset.gif" style="width: 100%;" />