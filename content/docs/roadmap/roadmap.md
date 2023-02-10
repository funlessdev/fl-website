+++
title = "Roadmap"
description = "The 2023 FunLess roadmap."
date = 2021-05-01T18:10:00+00:00
updated = 2021-05-01T18:10:00+00:00
draft = false
weight = 410
sort_by = "weight"
template = "docs/page.html"

[extra]
lead = "A view of what we are working on and what we will do in the future."
toc = true
top = false
+++

## 2023 Roadmap

These are the expected milestones for this year. 
The plan is subject to change as this is still an experimental platform, 
the purpose of which is first and foremost to assist in our research work. 
Because of this, usability and frontend-focused additions may fall behind in priority compared to research-driven features 
(e.g. scheduling optimization).

### Q1

- [ ] Authentication & Authorization
- [ ] HTTP requests from wasm functions
- [ ] Functional Kubernetes deployment
- [ ] Benchmarks and comparison with other OSS serverless platforms  
- [ ] Small slack bot demo

### Q2

- [ ] Topology-aware scheduling
- [ ] Multi-core deployments with reverse-proxy
- [ ] Nats connector/data sink
- [ ] OpenTelemetry integration setup
- [ ] Helm chart
- [ ] Benchmarks with multi-core scenarios

### Q3

- [ ] Custom scheduling policies
- [ ] Web functions
- [ ] Function workflows
- [ ] Kafka connector/data sink
- [ ] More OpenTelemetry/Metrics 
- [ ] Bare-metal deployment with ansible
- [ ] Small web functions demo

### Q4

- [ ] Admin & Playground interface
- [ ] Wasm functions registry
- [ ] Utility library to develop functions
- [ ] Nomad deployment
- [ ] Support more languages
- [ ] Postgres connector/data-sink
- [ ] Web site demo made with FunLess

### Beyond

- Low-Code platform
- More connectors/data-sinks (Amazon S3, MySql, Google Cloud Storage...)