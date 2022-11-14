+++
title = "Rust - Inject a trace" 
date = "2022-11-14"
+++


Hard day at work, sad that days have only 24h. I want to write about a small trick inspired by javascript promise based on [`and_then`][1].
I recently faced an issue with an wrapper function around sqlx where I want to inject at the level of this function. 
But to frankly speaking, this was that kind of single expression function that makes rust so enjoyable (scala's fellows will know). 

Let's make an example. Imagine that the `check` function is out of my crate / package (I recently understood the diff between a crate and a packages, check [here][2] for more information).

```rust
fn check(number: i32) -> Result<i32, &'static str> {
    if number == 7 {
        return Err("bouhh")
    }
    
    Ok(number)
}


fn handle_check(number: i32) -> Result<i32, &'static str> {
    check(number)
}
```

I have this wrapper function called handle_check that makes some glue on top of the check function. 
Imagine a bit more conf plus that the check function is a big builder that you consume in an upper function.

```rust
fn main() {
    let rok = handle_check(1);
    let rerr = handle_check(7);
}
```

The question is simple, how to inject some trace in the `handle_check` elegantly ?
There are no options to do it without unwrapping the content of the result obviously. So a naive and (to me) ugly solution is to do it with a `match`

```rust
fn handle_check(number: i32) -> Result<i32, &'static str> {
    let r = check(number);
    match r {
      Ok(v) => Ok(v),
      Err(err) => {
        println!("log something");
        Err(err)
      }
    }
}
```

A much cleaner way is again to use Result operators. They're most of use a brilliant way to express simple things simply. 
Here [`and_then`][1] or [`or_else`] make it explicit about the actions you perform in case the result is Ok or Err without loosing concision.  

```rust
fn handle_check(number: i32) -> Result<i32, &'static str> {
    check(number)
      .and_then(|res| {
          println!("log something good");
          Ok(res)
      })
      .or_else(|err| {
          println!("log something bad");
          Err(err)
      })
}
```

This is not optimal, we usually want to unwrap the error / result at the good moment to avoid extra consumption. 
But for tracability, it's really interesting to have the information at the good level.

See you!


[1]: https://doc.rust-lang.org/std/result/enum.Result.html#method.and_then
[2]: https://doc.rust-lang.org/book/ch07-01-packages-and-crates.html#:~:text=The%20crate%20root%20is%20a,A%20package%20contains%20a%20Cargo.
[3]: https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=facaeaf761091344199e69b672b3d6dd
[4]: https://doc.rust-lang.org/std/result/enum.Result.html#method.or_else
