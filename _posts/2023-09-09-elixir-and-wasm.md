---
layout: post
title: Running Wasm in Elixir & Erlang
category: articles
tags: [wasm, elixir, erlang]
---

I think someone is going to need to write a native Elixir or Erlang Wasm runtime. Let me explain why.

## Background

One of the first projects I took on with [Extism](https://extism.org/) was to build an [Elixir Host SDK](https://hexdocs.pm/extism/0.5.0/readme.html). The architecture is fairly simple. I use [Rustler](https://docs.rs/rustler/latest/rustler/) to create a [NIF](https://www.erlang.org/doc/tutorial/nif.html) in Rust. The NIF depends directly on the Extism runtime, which is itself written in Rust. So when you pull down and compile the Extism Elixir package, it compiles the Extism runtime and wraps it in a NIF which is further wrapped by some Elixir code.

## Host Function Support

I built this before we had support for [Host Functions](https://extism.org/docs/concepts/host-functions) in the Extism Runtime, but it still works just fine. For example, we built all of our [GameBox](https://extism.org/blog/extending-fly-io-distributed-game-system-part-2) demo application with just this functionality.

There were some things we had to work around without host functions, but we did it with some hacks. Now that we're grocery shopping features for Extism 1.0, I went to see what it would cost to add host functions to the Elixir SDK. That's where I ran into an unfortunate limitation.

## Async Host Functions

Implementing host functions would require the NIF to invoke a function at the Erlang / Elixir level. This appears to be impossible. Instead folks recommend calling `enif_send` to a process with the callback. It seems this is effectively what [wasmex does](https://github.com/tessi/wasmex/issues/256#issuecomment-848339952). They spin up a thread, send a message to the callback then, wait for the result.

I don't mean to criticise this solution. I think it's a fairly elegant solution within the confines of the NIF. And I'm sure it works for simple cases where you just want a Wasm module to trigger some action outside of it's memory boundary. But I think not being able to do fast, efficient, and synchronous host functions will eventually rule out a class of modules.

## Starting From Scratch

The only way I can see to solve this is to write a native Erlang or Elixir Wasm runtime. From a native runtime you'd be able to interop very deeply with Erlang and calling synchronous host functions wouldn't be a problem. You may even be able to write some kind of Wasm to BEAM Bytecode JIT to make it relatively fast.

I don't intend to pick up this project, but if you do, please reach out to me! Also, please reach out if you think I'm wrong or know of some other ways around this problem.

