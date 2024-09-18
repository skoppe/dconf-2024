---
# try also 'default' to start simple
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
# apply any windi css classes to the current slide
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# show line numbers in code blocks
lineNumbers: true
# some information about the slides, markdown enabled
title: Structured Concurrency
titleTemplate: '%s - DConf 2024'
info: |
  Structured Concurrency in D
  What is it, how does it work and why do we need it.
# persist drawings in exports and build
drawings:
  persist: false
# use UnoCSS (experimental)
css: unocss
---

<div class="flex flex-col h-full">
<h1>Structured Concurrency Update</h1>
<h3>aka Senders/Receivers pt 2</h3>
<div class="grow h-48"></div>

<div class="flex space-between items-end">
    <div class="flex bg-white text-black rounded-xl px-4 py-2">
      <mdi-link class="text-xl mr-2"/>
      <a class="" href="https://skoppe.github.io/dconf-2024/">https://skoppe.github.io/dconf-2024/</a>
    </div>
    <div class="grow"></div>
    <!-- <img src="/qr-skoppe-github-i.svg" class="w-1/8 rounded-2xl"> -->
</div>
</div>

---
class: flex flex-col h-full
---
# Refresher

<div class="flex flex-col items-center -mt-4 h-full">
  <img src="/nesting.png" class="rounded shadow pb-4 h-full" />
</div>

<!--

The aim is to get any concurrent or parallel task to always be encapsulated by whomever started it.

This makes it a lot more easy to reason about already complex code.

-->
---

# Update

#### Improvements

- memory usage
- fewer external dependencies
- does less work
- streams => sequences (ongoing rewrite to address shortcomings)
<br/>
<br/>
#### New features

- support for IO (iouring only atm)
- support for Fibers

<!--

- We improved memory usage by either removing them or putting them on the stack
- Removed some dependencies. Walter asked me to get this into Phobos, and I expected it to be easy... It isn't. I was depending on several dub libraries, which had to be tackled first
- so I removed them, made things simpler, less moving parts, and performance improvements
- reworked the streams into sequences, mostly to solve things around backpressue and to allow streams of streams etc.

After having completed key parts of the foundation, I moved on to io and fibers.

-->
---

# Using it

| | |
| --- | --- |
| Safe | <mdi-check-bold class="text-green-600"/> |
| Performant | <mdi-check-bold class="text-green-600"/> |
| Composable | <mdi-check-bold class="text-green-600"/> |
| Integrate | <mdi-check-bold class="text-green-600"/> |


<!--

Besides that it is safe, performant and composable, we have also seen that it integrates well.

So if you have some existing async code, it is very amenable to wrap that.

I believe that it because Senders/Receivers captures the essence of an asynchronous computation very well.

But...

-->

---

# Not all is great

- can be daunting to write Senders/Receivers
- having to learn new constructs
- forced to (re)shape your code

<!--

It can be daunting to write Senders/Receiver code
you are going to have to learn some new constructs
and you are going to have to write code unlike you do normally

You also need to start thinking about lifetimes, and it can become a whole problem unto itself.

-->

---

# What if?

<div class="flex flex-col items-center">
<mdi-scale-unbalanced class="size-sm"/>
</div>

<!--

What if we are willing to give up on a bit of performance, and accept say an extra allocation, or some indirection, or a bit of type erasure, then what is the next best thing?

-->

---

# Fibers!

```d
void main() @safe {
    auto io = IOContext.construct(1024);
    shared fd = listenTcp("127.0.0.1", 8888, 512);
    scope(exit) closeSocket(fd);

    io.run(fiber(() @safe shared {
        auto client = acceptAsync(fd).yield();
        scope(exit) closeAsync(client.fd).yield();

        ubyte[1024] buffer;
        for (;;) {
            auto req = readAsync(client.fd, buffer[], 0).yield()
                .parseRequest();

            string response = "HTTP/1.0 200 OK\r\nConnec...";
            writeAsync(client.fd, response, 0).yield();
        }
    })).syncWait().assumeOk;
}
```

<!--

Fibers!

With Fibers you don't need to learn new stuff, you can program synchronously.

This `fiber` here and these `acceptAsync` etc are just senders/receivers themselves.

Because of the integration you can:
- use Fibers and Senders interchangebly
- which means you can pick one first, and then refactor them where needed later on
- start with a Fiber today and move to Sender/Receiver later for the extra performance 

-->

---
layout: section
---
# Fibers? Really?


<!--

Actually, I don't like Fibers. There are a few core issues with them.

But it is currently the best we got.

And I am sure that if we get something else... Let say...

-->
---

# Stackless coroutines

<br/><br/>
<div class="flex flex-col items-center -mt-4 h-1/2">
  <img src="/NotoHuggingFace.svg" class="rounded pb-4 h-full" />
</div>

<!--

Stackless coroutines, we will be able to integrate them in a similar way with the same tradeoffs.

So you get all the safety, performance, composability _and_ also convenience to write structured concurrent code.

-->
