+++
title = "Futures with results"
date = 2022-11-12
+++

Yesterday, I spent a lot of time trying to figure out how to manage a collection of results in an async context.

By a collection of results I mean that I have several operation to perform and I want to be aware of the completion of each of them.

## The basics

To solve that, it's worth taking a step back and analyse how it works in a synchronized context.

Let's say that I have a function that, given a number return a result with the same number or an error.

```rust
fn check(number: i32) -> Result<i32, &'static str> {
    if number == 7 {
        return Err("Bouhh, we do not accept 7!");
    }

    Ok(number)
}
```

What happen if I want to `check` over a range of values?

```rust
let results: Vec<_> = (1..10).map(check).collect();
```

`results` contains a list of results enum. To check over the completion it's worth iterating again and doing some `filter_map`, `partition` as described in [Rust by example][1].

## The async way of doing it

When dealing in an async context, using a tokio runtime or a [ThreadPool][2] things aren't that much different. Except that playing with an async context is a bit more "abstract" and that you have to control your async tasks behavior.

For this example, I'm using the [futures][3] crates that bring everything we need to play with async.

In your project, you have to declare dependencies like this. Crates version is the one available at the time of writing.

```toml
[dependencies]
futures = { version = "0.3.25", features = ["executor", "thread-pool"]}
```

Let's rebuild the check function for async and intialize a [ThreadPool][2].

```rust
use futures::{executor::ThreadPool;

async fn async_drain(number: i32) -> Result<i32, &'static str> {
    if number == 7 {
        return Err("7 have been found, bouuh");
    }
    Ok(number)
}

fn main() {
    let pool = ThreadPool::new().unwrap();
}
```

Now we can do mostly the same thing, building an iterator that drain a Range
and catch some Futures. I add the type of the Mapper to better see what's inside.

```rust
// Map<Range<i32>, |i32| -> impl Future<Output = Result<i32, &str>>>
let tasks = (1..10).map(|i| async_drain(i));
```

The big difference is the manipulation that we do to create the future to await _all_ the async tasks.

```rust
let future = futures::future::join_all(tasks);
```

Then to process it with the thread pool we can do the following.

```rust
let test = pool.spawn_with_handle(future).unwrap();
// res: Vec<Result<i32, &str>>
let res = block_on(test);

for r in res {
    match r {
        Ok(v) => {println!("{}", v);},
        Err(err) => {println!("{}", err)}
    }
}
```

`spawn_with_handle` help to capture a result from the future, where `spawn` only capture a correct execution information.

`block_on` block the current thread until you get the resolve the future. It mostly simulate the `.await` in a more classic situation.

I hope those examples help you to overcome this situation if you face it one day.

[1]: https://doc.rust-lang.org/rust-by-example/error/iter_result.html
[2]: https://docs.rs/futures/latest/futures/executor/struct.ThreadPool.html
[3]: https://crates.io/crates/futures
