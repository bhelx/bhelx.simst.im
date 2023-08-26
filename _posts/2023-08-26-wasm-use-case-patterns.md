---
layout: post
title: On Wasm Use Cases
category: articles
tags: [wasm]
---

The goal of [Extism](https://extism.org/) was to show that Wasm could be the *"Last Plug-In System We Need"* as [Steve](https://twitter.com/nilslice) likes to say. I think we have made a good case to developers in the know, but simultaneously, I have had difficulty explaining to developers what a plug-in system is and [why you need one](https://www.youtube.com/watch?v=FCUSMRHCsLU).

I think part of the problem is that we forgot how and why to build plug-in systems [when we moved to the web](https://youtu.be/FCUSMRHCsLU?t=228).
Another issue is that Wasm is more than just a plug-in system and it can be tough to express how and what the use cases are exactly.

Instead of enumerating all the possible things you could do with Wasm, I wanted to attempt to distill the use cases to their simplest form, like a pattern. This pattern doesn't cover everything, but it focuses on what [I have said is the most compelling and differentiating property](/articles/why-webassembly.html) of Wasm: its sandboxing capabilities.

## Application = Computation & Data

An application is, in its simplest form, two things: *compute* and *data*.
Modern applications are themselves composed of many other applications, all of which exist across the internet and belong to different organizations.
Each of these applications also have their own code and their own data.

## Moving Data to Compute

The primary way we "integrate" these applications is by moving data from one application which owns it to the computation that needs it.

<img alt="Moving Data to Compute" src="/public/images/data-to-compute.png">

Let's say one of our applications we are integrating with is [Stripe](https://stripe.com/).
We might make HTTP calls to their API and do something with, or store, the result as data.
Or they might, unprompted, send us a data payload representing an event in their system (a webhook).

There are many downsides to doing things this way. A couple examples:

* Data Governance
  - Once the data leaves your system, you've lost control of it. And remember, these applications are spread out through space which includes multiple legal jurisdictions.
* High Latency & Low Reliability
  - If an event occurs and the compute needed to handle it belongs to a company across the world,
  how can you resolve the event in a timely and reliable manner?

## Moving Compute to Data

The other option is to move the compute to the data. This isn't a new idea and is used frequently in the "big data" world. One example that comes top of mind is [Apache Hadoop](https://hadoop.apache.org/).

<img alt="Moving Compute to Data" src="/public/images/compute-to-data.png">

But why is this so rare in the web-development world? The primary issue again comes down to isolation.
All of these cloud applications are multi-tenant.
Shipping code to Stripe for them to run on our behalf when a customer event happens opens them up many risks.
If we ship them Ruby code to run in their Rails application, what is to stop us from extracting secrets or data from them or their customers?
The current solutions for isolation, e.g. a container, are too heavyweight for most of these scenarios. Imagine if we must send over a whole OS and VM to Stripe so they can boot it up and decide what discounts to apply to an invoice.
The result has been that the downsides of sharing data are less than the downsides of sharing compute.

## Wasm Makes This Option Viable

Of course, I think Wasm has changed the balance here. We can easily accept Wasm modules from our partners because they are small, fast, isolated, language independent, and can be run in process. They present as a function to the host developer and are easy to invoke. Because you control the whole world as the host, you can also ensure some level of determinism and reliability as well.

## What Does the Future Look Like?

So, should we move all our applications to this new model? Not necessarily. But what we need to do is take stock of the last 20 years of development and ask ourselves what is possible now that we have this option.

