+++
title = "Error converter in rust"
date = 2022-11-13
+++

Do you remember three tier architecture? Between logic tier and data tiers in your application server code, you'll always find a bunch of code related to error management about databases issues. Most of the time you want to map the database issue with something that you can manage on your side.

Let's say that I want to deal with the following errors.

```rust
enum MyError {
    SomethingWeirdHappen(String),
    IKnowWellThisError(String)
}
```

But several libs that I use are providing something like a `DatabaseError` or just their own `Error` enum.

```rust
fn db_exec(statement: &str) -> Result<(), DatabaseError> {
    ...
}

fn send_message_mq() -> Result<(), mq::Error> {
    ...
}

fn main() -> Result<(), MyError> {

    match db_exec("...") {
        Ok(v) => {
            match send_message_mq() {
                Ok(v) => {
                    ...
                },
                Err(err) => {
                    return Err(MyError::IKnowWellThisError(err.to_string()))
                }
            }
        },
        Err(err) => {
            return Err(MyError::SomethingWeirdHappen(err.to_string()))
        }
    }

    Ok(())
}
```

To simplify a bit, I try to introduce a converter function that will handle lib error and convert it to my own.
With this converter, it's easy to wrap fully the scope of the subsequent error you have to deal to what you manage on your side.

```rust
fn mq_error(err: mq::Error) -> MyError {
    match err {
        mq::Error::TheBasicError(err) => MyError::IKnowWellThisError(err.to_string()),
        _ => MyError::SomethingWeirdHappen(err.to_string())
    }
}
```

To use it, [`map_err`][1] will be your savior. You can pass the converter directly to the function and retrieve what you need.

```rust
let res: Result<(), MyError> = send_message_mq().map_err(mq_error);
```

As the error you manage is now a `MyError` you can leverage the [`?` shortcut][2] to simplify your code.

```rust
fn main() -> Result<(), MyError> {
    let v = db.exec("...").map_err(db_error)?;
    send_message_mq().map_err(mq_error)?;

    Ok(())
}
```

By mapping most of the error that can appear in dependencies lib, you'll have a safer design will knowing better what happen at runtime.

[1]: https://doc.rust-lang.org/std/result/enum.Result.html#method.map_err
[2]: https://doc.rust-lang.org/rust-by-example/std/result/question_mark.html
