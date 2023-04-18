+++
title = "Rust - Object store for newbies"
date = 2022-11-11
+++

The [object_store][1] package is a convenient way to adress common cloud object stores.

Cloud object store is a simple, efficient and cheap way of storing files. Use cases goes from serving static website content to host gigabytes of files for data processing.

The package [object_stores][1] that we will use today is hosted in the arrow-rs project and essentially designed to manage files used in arrow / datafusion contexts.

We're going to goes through the API to be able to:

- configure your ovh cloud storage
- list files
- create a new small file
- read it back in memory
- finally delete it

Crates documentation is available [here][2]

## List files

```rust
use futures::stream::StreamExt;
use object_store::{aws::AmazonS3Builder, path::Path, Error, ObjectMeta, ObjectStore};

async fn display_metadata(maybe_meta: Result<ObjectMeta, Error>) {
    let meta = maybe_meta.unwrap();
    println!("Name: {}, size: {}", meta.location, meta.size);
}

#[tokio::main]
async fn main() {
    let os = AmazonS3Builder::from_env()
        .with_bucket_name("object-store-demo")
        .build()
        .unwrap();

    let prefix = Path::from("data");

    os.list(Some(&prefix))
        .await
        .expect("Error while listing files")
        .for_each(display_metadata)
        .await;
}
```

## Create a new file object

You can replace the code for listing file object with the following one.
The key part of this piece of code is the ObjectStore that implements [put][3] to save bytes to the target [Path][4] location.

```rust
let content = r#"
Lorem ipsum dolor sit amet, consectetur adipiscing elit,
sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.
Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris
nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in
reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla
pariatur."#;

let target = Path::from("data/lorem_ipsum.txt");
if let Err(err) = os.put(&target, Bytes::from(content)).await {
    panic!("{}", err);
} else {
    println!("File have been written");
}
```

## Get back the file content

There are several way to capture file content. Especially with large file you should consider a [streaming approach][5].

In our example, we just poll the data in memory and convert it to an utf8 string with [`std::str::from_utf8`][6].

```rust
// Let's cut it in several steps
let file = os.get(&target).await.unwrap();
let byte_content = file.bytes().await.unwrap();
let utf8_content = std::str::from_utf8(&byte_content).unwrap();
```

## Delete file content.

In the end, file deletion is a really simple task.

```rust
if let Err(err) = os.delete(&target).await {
    println!("Issue while deleting file object: {}", err);
};
```

[1]: https://crates.io/crates/object_store
[2]: https://docs.rs/object_store/latest/object_store/
[3]: https://docs.rs/object_store/latest/object_store/trait.ObjectStore.html#required-methods
[4]: https://docs.rs/object_store/latest/object_store/path/struct.Path.html
[5]: https://docs.rs/object_store/latest/object_store/index.html#fetching-objects
[6]: https://doc.rust-lang.org/stable/std/str/fn.from_utf8.html#
