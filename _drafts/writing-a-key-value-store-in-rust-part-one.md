---
layout: post
title:  "Writing A Key Value Store In Rust: Part One"
date:   2017-11-10 09:00:00
categories: rust databases
---

![Oracle Database Icon Style Offices](/assets/writing-a-key-value-store-in-rust-part-one/databases.jpg)

As an engineer it's important to find ways to increase your technical skills. There are lots of projects to get you
started with a language or framework but once you move beyond the basics it is not always easy to find projects that can
teach you intermediate or advanced concepts without a large up front time commitment. A good way to approach learning
these more complicated concepts is to try to find or build a toy implementation that lets you see the structure of the
idea or solution without having to understand all of the details and optimizations at once.

In this blog post I will build a very simple database from scratch as example. It will be a key value store instead of
a full relational database for simplicity but as you will see many of the concepts would carry over. I will also be
working in [rust](https://www.rust-lang.org/) both because it's good to pick up new languages and because rust is a
systems language so it is particularly well suited for this type of application because of it's speed and concurrency
strength.

All of my code for this project is available on github at [jtescher/tokio-minidb](https://github.com/jtescher/tokio-minidb/tree/v1).

## Why Are Key Value Stores Useful

A key value store is essentially a service that allows you to store data under a certain key. It's like an object, hash,
or dictionary that come in the standard library of languages but it is exposed with a specific interface and can be
embedded in applications or connected over a network. As usual [Wikipedia](https://en.wikipedia.org/wiki/Key-value_database)
has a good description and list of popular implementations:

> A key-value database, or key-value store, is a data storage paradigm designed for storing, retrieving, and managing
associative arrays, a data structure more commonly known today as a dictionary or hash. Dictionaries contain a
collection of objects, or records, which in turn have many different fields within them, each containing data.
These records are stored and retrieved using a key that uniquely identifies the record, and is used to quickly find the
data within the database.

So why are these useful? Key value stores are often used as shared caches like
[Memcached](https://en.wikipedia.org/wiki/Memcached), simple data stores like [Redis](https://redis.io), or as building
blocks for more complicated databases like [RocksDB](http://rocksdb.org).

Now that you know what key value stores are and why you might want to build a simple one, let's get started.

## Create The New Rust Project

To start, let's create a new rust project using [Cargo](http://doc.crates.io). We will be using [Tokio](https://tokio.rs)
which I will explain later to provide a network interface for the server, but for now lets call the project
`tokio-minidb`:

```shell
$ cargo new --bin tokio-minidb
```

You can test that everything is set up properly by going into this directory and running the app:

```shell
$ cd tokio-minidb && cargo run
   Compiling tokio-minidb v0.1.0 (file:///Users/julian/code/tokio-minidb)
    Finished dev [unoptimized + debuginfo] target(s) in 1.19 secs
     Running `target/debug/tokio-minidb`
Hello, world!
```

### Sketch the API

Excellent. now that we have a working rust app scaffold, let's sketch out a simple use case for storing and retrieving
some data. Open your `src/main.rs` file and replace it with the following:

```rust
extern crate tokio_minidb;

use tokio_minidb::DB;

fn main() {
    // Create new database.
    let mut db = DB::new().unwrap();

    // Create some data. In this case store user data as JSON.
    let key = &"user-1".to_string();
    let value = &"{ \"first_name\": \"John\", \"last_name\": \"Doe\"}".to_string();

    // When we get a key that is not yet set it should return None.
    println!("Get {} => {:?}", key, db.get(key).unwrap());

    // When we add data it should save the data and return what was there before, in this case None.
    println!("Put {} => {}", key, value);
    db.put(key.clone(), value.clone()).unwrap();

    // Now when we get the data it should return our value that we associated with this key.
    println!("Get {} => {}", key, db.get(key).unwrap().unwrap());

    // We should be able to iterate over all keys and values in the store.
    println!("Iterate over keys...");
    for (k, v) in db.iter() {
        println!("Key: {}, Value: {}", k, v);
    }

    // If we delete data it should return the value that was stored at the key.
    println!("Delete key {}", key);
    db.delete(&key).unwrap();

    // And finally we should see our empty database so we know the delete was successful.
    println!("Final db: {:?}", db);
}
```

Ok. If you try to build this project now it will of course not work because we haven't implemented any of these methods.
The simplest working version of this would be an in-memory database that will be completely re-created every time we
start the application. Let's start by making a trait to encode the API we sketched out in our main file. Create a new
file `src/lib.rs` with the following trait:

```rust
use std::collections::hash_map::Iter;
use std::io;

// Simple key value database interface
pub trait DB: Sized {
    fn new() -> Result<Self, io::Error>;
    fn put(&mut self, key: String, value: String) -> DBResult;
    fn get(&self, key: &String) -> DBResult;
    fn delete(&mut self, key: &String) -> DBResult;
    fn iter(&self) -> DBIterator;
}

pub struct DBIterator<'a> {
    inner: Iter<'a, String, String>,
}

impl<'a> Iterator for DBIterator<'a> {
    type Item = (&'a String, &'a String);

    fn next(&mut self) -> Option<Self::Item> {
        self.inner.next()
    }
}

type DBResult = Result<Option<String>, io::Error>;
```

Using just the rust standard library we have defined a trait that we can implement to store and retrieve our data. We
also defined an `iter` method and using the rust convention we use a struct to hold the state of the iterator and
implement the `Iterator` trait so we can do simple `for (key, value) in db` syntax in the main file. We also defined a
`DBResult` type for our methods so they will all return an optional string or an io error when they are called.

### Implement The In Memory Database

Now that we know what we want from our limited key value store let's add a storage module. Create a file called
`src/storage/mod.rs` with the following:

```rust
pub mod in_memory_db;
```

And then add `src/storage/in_memory_db.rs` with:

```rust
use std::collections::HashMap;
use std::io;

use {DBIterator, DBResult, DB};

#[derive(Debug)]
pub struct InMemoryDB {
    data: HashMap<String, String>,
}

impl DB for InMemoryDB {
    fn new() -> Result<InMemoryDB, io::Error> {
        Ok(InMemoryDB {
            data: HashMap::new(),
        })
    }

    fn put(&mut self, key: String, value: String) -> DBResult {
        Ok(self.data.insert(key, value))
    }

    fn get(&self, key: &String) -> DBResult {
        Ok(self.data.get(key).cloned())
    }

    fn delete(&mut self, key: &String) -> DBResult {
        Ok(self.data.remove(key))
    }

    fn iter(&self) -> DBIterator {
        DBIterator {
            inner: self.data.iter(),
        }
    }
}
```

Our `InMemoryDB` is nothing more than a wrapper around rust's `HashMap` class. Each method call to `get`, `put` or
`delete` can simply delegate to the underlying `HashMap` and the `iter` method can iterate over the keys and values in
the map.

Now we can go to our `src/lib.rs` file and expose the storage module for public use by adding:

```rust
use std::collections::hash_map::Iter;
use std::io;

mod storage;
pub use storage::in_memory_db::InMemoryDB;

...
```

And change our `src/main.rs` file to use the in memory db:

```rust
extern crate tokio_minidb;

use tokio_minidb::{DB, InMemoryDB};

fn main() {
    // Create new database.
    let mut db = InMemoryDB::new().unwrap();
    ...
```

And now if you run your app you should see everything working:

```shell
$ cargo run
Get key-1 => None
Put user-1 => { "first_name": "John", "last_name": "Doe"}
Get user-1 => { "first_name": "John", "last_name": "Doe"}
Iterate over keys...
Key: user-1, Value: { "first_name": "John", "last_name": "Doe"}
Delete key key-1
Final db: InMemoryDB { data: {} }
```

## For Next Time

You now have a working (very minimal) in memory key value store that can be embedded in other applications as a rust
package (crate). In the next part I will show you how to expose this database over the network so other applications can
store and retrieve their data.
