+++
title = "Cargo add - Enjoy rust dependencies"
date = 2023-03-12
+++

# What's this?

Yesterday i discovered the add feature of cargo. Before using it, I was just modifying my Cargo.toml manually by looking at the right version to use in [crates.io][2].

`cargo add` behaves similarly to npm or pip, respectively for Node.js and Python, but it's even simpler. It doesn't install any libraries by itself; instead, it helps you declare your dependencies in the Cargo.toml file.

With `cargo add`, you can add, update, or require a new feature with a single command line, and let cargo resolve the dependencies' constraints for you.

Let's create a test project to put our hands-on a practical situation.

```shell
$ cargo new test-cargo-add
$ cd test-cargo-add
```

# Declare dependencies

Suppose we want to create a small HTTP client for a CRON job, and we want to use reqwest and tokio as an asynchronous runtime. With `cargo add`, it's as simple as using the following command:

```shell
$ cargo add tokio
Updating crates.io index
Adding tokio v1.26.0 to dependencies.
        Features:
        - bytes
        - fs
        - full
        - io-std
        - io-util
        - libc
        - macros
        - memchr
        - mio
        - net
        - num_cpus
        - parking_lot
        - process
        - rt
        - rt-multi-thread
        - signal
        - signal-hook-registry
        - socket2
        - stats
        - sync
        - test-util
        - time
        - tokio-macros
        - tracing
        - windows-sys
```

The `cargo add` command will update the crates.io index and add the tokio library with version `1.26.0` to your dependencies. It also provides you with a list of the available features you can use with the library.

The consequence of this command is that it will simply add the following line to the Cargo.toml:

```toml
tokio = "1.26.0"
```

If we want to include `reqwest` as a dependency, we can stack them up in one single command:

```shell
$ cargo add tokio reqwest
```

The `cargo add` command will add the reqwest dependency to the Cargo.toml file.

```toml
reqwest = "0.11.14"
tokio = "1.26.0"
```

# Add features

The crates system includes a small, funny aspect called "features" that you can't avoid. Features are a way to enable some functionalities of the crates. They permit conditional compilation following your needs.

For example, we want to have the `rt-multi-thread` feature in `tokio`. It's a convenient one when you build a simple network client. You can use the following command line to add it:

```shell
$ cargo add tokio --features rt-multi-thread
```

The consequence is the update of the `Cargo.toml` file again, with this time a clear definition of the requested features.

```toml
tokio = { version = "1.26.0", features = ["rt-multi-thread"] }
```

To add several features at once:

```shell
$ cargo add tokio --features rt-multi-thread,fs
```

If you add new features, it will update the features list accordingly.

The command line to add several dependencies with different features is the following.

```shell
$ cargo add tokio reqwest --features tokio/rt-multi-thread,reqwest/json
# or
$ cargo add tokio --features tokio/rt-multi-thread reqwest --features reqwest/json
```

Note this time the need to identify the dependencies with the `<dependencies>/<feature>` pattern.

The full documentation of the cargo add is [here][1]

[1]: https://doc.rust-lang.org/cargo/commands/cargo-add.html
[2]: https://crates.io/
