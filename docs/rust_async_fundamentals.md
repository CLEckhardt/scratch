# Rust Async Fundamentals

## Overview

This is more a "things that are good to know if you're working with async
Rust."

Many good tutorials and resources on async Rust:
* [Async Rust Book](https://rust-lang.github.io/async-book/) - The official
  async Rust book
* [Tokio docs](https://tokio.rs/tokio/tutorial) - Tokio is a common async
  runtime for Rust, and their documentation is an excellent description of both
  the crate and the concepts behind it
* [Crust of Rust on async/await](https://www.youtube.com/watch?v=ThjvMReOXYM) -
  Jon Gjengset goes into detail about how to think about async/await in Rust
* [Rust for Rustaceans](https://rust-for-rustaceans.com/) - In-depth chapter on
  async Rust

And many more via your favorite search engine...

## Async basics

### Async vs. multithreading

* Both fall under "concurrent programming"
* Async is good for __IO-bound__ computing, where the processor spends
  significant time waiting for other tasks to complete or data to arrive (ex.
  waiting for a response from a POST request, waiting for user input, waiting
  for a drive to retrieve data, etc.)
* Multithreading (parallel) is good for __CPU-bound__ computing, where a core
  is being used to its limit (ex. calculations, simulations, etc.)
* Using async for CPU-bound problems will result in no improvement (when using
  a single-threaded runtime)
* Using multithreading for IO-bound problems will provide limited improvement
  and is very inefficien

### How async works (very simply)
* Tasks are defined by "futures" (sometimes called "promises"), which represent
  data that will exist in the future
* Code that uses futures will wait until the future completes before continuing
  (wait... then what's the point???)
* When code is waiting for multiple futures, the OS can schedule multiple
  futures to maximize processor use (this usually involves waiting on a list of
  futures until one or all of the futures complete)
* When a future is polled, it will do work until it must wait on something
  else, then it will yield back the thread to the OS; if the future completes,
  work goes back to the code that was waiting on the future
* If a future yields the thread to the OS, the OS can then run a different
  future while the first waits
* Often, we don't poll futures directly, we have a runtime drive them to
  completion for us

_Writing async code does not make your code async!! It is possible to write
code that is only ever waiting on one future to complete - this makes the
future pointless!_

## Async in Rust

### Writing async code

* It's easy!
* One of Rust's key features - has a dedicated working group (Rust async
  working group)
* Rust's `async` and `await` keywords are syntax sugar for some
  somewhat-complex future structs
* `async` functions/methods must be `await`ed
* Using the runtime of your choice is straightforward - you can even build your
  own if you really want to

I strongly recommend you try a tutorial to learn more about writing async code
in Rust!

### Runtimes in Rust

* Unlike other languages, Rust does not have a built-in runtime (yay!)
* We can pick our own runtime based on project needs
* Check the runtime documentation for implementation specifics
* See tokio description below
* Unlike other languages, futures do not automatically get driven to
  completion; they must be `awaited`! (see "Known issues with async Rust:
  Futures cancelation" below)

#### Tokio <3

* The tokio runtime is by far the most popular runtime used across Rust
* Underlies many other popular crates like actix and reqwest
* Very powerful and sophisticated under the hood
* Relatively easy to use (support for a complete async main function or
  spawning a runtime on a separate thread for sync/async interface)
* Includes lots of helpful things like async-specific channels
* Very flexible
* Very stable (developed and maintained by Amazon)
* BUT... It was developed by Amazon -> it was developed for the scale of
  Amazon. It's one of the few areas of Rust that does not strive to put
  simplicity first. Out-of-the-box, it is also multithreaded, meaning your code
  must be `Send + Sync` and have all the required lifetimes (as of this
  writing). It has single-threaded options, but they are not the focus of the
  crate and development. There are a few other, single-threaded runtimes out
  there that you may want to explore if this is important for your project (I
  haven't used them, so I won't recommend any in particular - lmk if you try
  one!).

### Known issues with async Rust

#### Async traits

* Currently, traits don't have native support for async methods
* Developing this is a top priority for the Rust async working group
* The async working group recommends using the `async-trait` crate
* `async-trait` provides a macro that rewrites your functions so that act like
  native async methods
* So what's the problem? The `async-trait` implementation puts data on the
  heap - many projects won't be impacted by this, but it can have performance
  impacts for high-scale/load projects
* The curious can check out [why async fn in traits are hard](http://www.smallcultfollowing.com/babysteps/blog/2019/10/26/async-fn-in-traits-are-hard/)

#### Future cancelation: when Rust is not memory safe ;(

* Put simply, there are situations in which data can be lost when futures are
  dropped
* For example, if you have a future that is reading data from a TCP stream, and
  that future is dropped (maybe you are waiting for either data or a user
  input, and you only use whichever future completes first), you will lose that
  data
* Other languages avoid this problem by driving futures to completion and
  making them available again
* For more info, check out
  [Jon Gjenset and the Pimemagen](https://youtu.be/xO2xyBfhKDY?t=1647) or
  [Niko Matsakis](http://www.smallcultfollowing.com/babysteps/blog/2022/06/13/async-cancellation-a-case-study-of-pub-sub-in-mini-redis/)

## State of async in Rust

Check out the
[async update from RustConf 2022](https://www.youtube.com/watch?v=tHrvYtPNAHA)!
