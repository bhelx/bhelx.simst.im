---
layout: post
title: Why WebAssembly
category: articles
tags: [wasm]
---

I've done many disparate things in the first decade of my career. For the next decade, I want to narrow my focus on one domain.
I believe [WebAssembly](https://webassembly.org/) is this domain.
I'm writing this post to explain my thoughts to those who've been asking me and for my future self.

There are two perspectives to this question of *"Why Wasm?"*.
Why does the industry need it?
And why is this problem well suited for me?

## Why we need Wasm

There are a few things that make Wasm different than say, the Java bytecode, but I want to focus on what I think is the most compelling: Sandboxing.
By default, Wasm programs have no capabilities unless explicitlyexplicitly  granted to them by the host.
The program's memory access is contained to a contiguous block of linear memory.

It can give you a similar level of isolation as an operating system with the lightweight nature of a thread.
This makes it ideal for running untrusted code in your own infrastructure.
My feeling is that running untrusted code has always been a very hard and scary feature that has been left to the domain of browser and OS engineers.
What suboptimal patterns and architectures have we built based out of fear that can now be relaxed?
One of the big accomplishments of Wasm will be to enhance security of existing patterns; but the security is also an opportunity in that we can more safely do things we instinctively see as unsafe today.

## Why I want to work on it

A few times in my life I've encountered an idea so simple but so powerful that it changed the way I think.
I remember feeling this way the first time I learned to program.
I learned how to do loops and print text to the screen.
With that tiny bit if knowledge, I spent the entire night making little console games that animated text.

I felt similarly when I first hand built a TTL computer. I made a toolchain and emulator to go along with it.
I believe doing this re-invigorated my passion for computers.
These momements always come with a feeling of fun and an uncontrollable outpouring of ideas.
Everytime I get this feeling I've been rewarded for following it.

I feel similar feelings about WebAssembly.
I still believe it's a 10 yr journey to mass adoption and there will be challenges and pivots along the way.
I'm not really sure where I'm going to end up when I come out of this, but I know I'm going in the right direction.


