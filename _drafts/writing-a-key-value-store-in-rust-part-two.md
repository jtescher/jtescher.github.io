---
layout: post
title:  "Writing A Key Value Store In Rust Part Two: Networking With Tokio"
date:   2017-11-11 09:00:00
categories: rust databases
---

![Tokio At Night](/assets/writing-a-key-value-store-in-rust-part-two/tokio-at-night.jpg)

_This is part two of a series on learning intermediate programming concepts by writing a database from scratch in Rust.
You can read part one [here](/writing-a-key-value-store-in-rust-part-one) first to catch up_.

In the previous post we created a Rust crate that allows us to embed a simple key value store in another application.
We can now extend this project to be available as a separate server that multiple clients can connect to over a network.

There is a great project in the Rust ecosystem called [Tokio](https://tokio.rs) that describes itself as "A platform for
writing fast networking code with Rust." and it will allow us to define a network protocol that we can use to interact
with our database.

As usual all of the code for this project is available on github at
[jtescher/tokio-minidb](https://github.com/jtescher/tokio-minidb/tree/v2).

## Designing A Protocol

How would we want to be able to get, put, and delete data over the network? One of the simplest options is to use line
endings and spaces to separate our commands, similar to how [Memcached](https://en.wikipedia.org/wiki/Memcached) and
[Redis](https://redis.io)'s text protocols work. This would allow us to use [Telnet](https://en.wikipedia.org/wiki/Telnet)
as a client and interact with our application like this:

```shell
$ telnet localhost 12345
Trying ::1...
telnet: connect to address ::1: Connection refused
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
get user-1
None
put user-1 '{"first_name": "John", "last_name": "Doe"}'
None
get user-1
'{"first_name": "John", "last_name": "Doe"}'
```

That seems like a good place to start. But how would we translate these strings into database commands? When telnet
sends data to your application over a TCP connection it will come as a stream of bytes but luckily Tokio has features
to make encoding and decoding these byte streams relatively easy.

## Adding Tokio

Tokio is broken up into several crates (Rust packages) that let you work with different layers of abstraction
depending on how much control you want. It provides high level interfaces that let you build applications that
interact with the network with minimal boilerplate code. If you want to learn more about how it works and see what else
you could build, including high performance web servers, check out their
[documentation](https://tokio.rs/docs/getting-started/tokio/).

Let's add it to our `Cargo.toml` file now under
dependencies. We will also add the [futures](https://docs.rs/futures) crate for working with async code, and the
[nom](https://github.com/Geal/nom) crate for generating our command parser:

```toml
[dependencies]
bytes = "0.4"
futures = "0.1"
tokio-io = "0.1"
tokio-core = "0.1"
tokio-proto = "0.1"
tokio-service = "0.1"
nom = "^3.2"
num_cpus = "1.7.0"
```

## Defining Command Objects

The first step to having our application understand the data coming off the network is to define command objects that we
can try to extract from the stream of bytes. Let's add a `DBCommand` enum that lists all of our supported commands and
a `process_command` method that looks at the given command and calls the corresponding method with the data. Add the
following to `src/lib.rs`:

```rust
extern crate bytes;
extern crate futures;
#[macro_use]
extern crate nom;
extern crate tokio_io;
extern crate tokio_proto;
extern crate tokio_service;

use std::collections::hash_map::Iter;
use std::io;
use std::sync::{Arc, Mutex};

pub mod server;
mod storage;
pub use storage::in_memory_db::InMemoryDB;

...

    fn iter(&self) -> DBIterator;

    // Helper function to build thread safe reference
    fn handle(self) -> Arc<Mutex<Self>> {
        Arc::new(Mutex::new(self))
    }

    // Helper function to invoke a given command
    fn process_command(&mut self, command: DBCommand) -> DBResult {
        match command {
            DBCommand::Get(ref key) => self.get(key),
            DBCommand::Put(key, value) => self.put(key, value),
            DBCommand::Delete(ref key) => self.delete(key),
            DBCommand::Bad => Ok(Some("Invalid command".to_string())),
        }
    }
}

...

#[derive(Debug)]
pub enum DBCommand {
    Get(String),
    Put(String, String),
    Delete(String),
    Bad,
}

type DBResult = Result<Option<String>, io::Error>;
```

This imports the Tokio crates and allows us to execute database commands with `DBCommand` objects. However in order to
build these `DBCommand` objects we sill need a way to decode the incoming bytes. Fortunately Tokio will help us do just
that via a [Codec](https://en.wikipedia.org/wiki/Codec).

## Implementing The Codec

To encode and decode byte streams, Tokio's IO crate provides the `Encoder` and `Decoder` traits. Let's create a
`src/server` folder in our application and add a `src/server/codec.rs` file with the following:

```rust
use std::io;
use bytes::BytesMut;
use tokio_io::codec::{Decoder, Encoder};

pub struct LineCodec;
use DBCommand;
use super::request_parser::parse_command;

impl Decoder for LineCodec {
    type Item = DBCommand;
    type Error = io::Error;

    fn decode(&mut self, buf: &mut BytesMut) -> io::Result<Option<Self::Item>> {
        if let Some(i) = buf.iter().position(|&b| b == b'\n') {
            // remove the serialized frame from the buffer.
            let line = buf.split_to(i);

            // Also remove the '\n'
            buf.split_to(1);

            Ok(Some(parse_command(&line)))
        } else {
            Ok(None)
        }
    }
}

impl Encoder for LineCodec {
    type Item = String;
    type Error = io::Error;

    fn encode(&mut self, msg: String, buf: &mut BytesMut) -> io::Result<()> {
        buf.extend(msg.as_bytes());
        buf.extend(b"\n");
        Ok(())
    }
}
```

Here we added a `LineCodec` struct and implemented the `Decoder` and `Encoder` traits. To decode the `DBCommand` from a
buffered stream of bytes, we can find the position of the newline character `\n` and use it to split our buffer into
sections. Each section will then contain the bytes of the characters like `GET user-1`. Encoding our responses is
simple because they are already formatted the way we want so we can just convert to bytes and add a `\n` character to
the end.

However our commands are still bytes or `[u8]`. We need a way to figure out which command those bytes represent and a
way to extract keys and values from the commands. In the encoder we have an unimplemented `parse_command` method that
we can create now by building a parser with the nom crate.

## Parsing Commands

In order to parse and extract commands from byte arrays we can use nom. Nom is a parser combinators library written in
Rust and it allows you to create very complicated parsers out of simple building blocks. Parsers are a bit out of the
scope of this post but if you are interested in learning more about nom I'd suggest reading the docs
[here](https://github.com/Geal/nom) or looking over some of the parsers that have been written using nom
[here](https://github.com/Geal/nom/issues/14).

Our command parser will be very simple because we only have 3 commands (get, put, and delete).

Create a file called `src/server/request_parser.rs` with the following:

```rust
use std::str;
use nom::{multispace, IResult};

use DBCommand;

pub fn parse_command(input: &[u8]) -> DBCommand {
    match nom_parse(input) {
        IResult::Done(_, command) => command,
        _ => DBCommand::Bad,
    }
}

// Helper macro to extract word [u8] -> word str
named!(word<&str>, map_res!(is_not!(" \t\r\n"), str::from_utf8));

// Helper macro to extract rest of the input to str
named!(rest_str<&str>, map_res!(is_not!("\r\n"), str::from_utf8));

// Parse commands
named!(nom_parse<&[u8], DBCommand>,
    alt!(cmd_get | cmd_put | cmd_delete)
);

// Get by key command
named!(cmd_get<&[u8], DBCommand>,
    do_parse!(
        tag_no_case!("get") >>
        multispace          >>
        key: word           >>
        (DBCommand::Get(key.to_string()))
    )
);

// Put value for key command
named!(cmd_put<&[u8], DBCommand>,
    do_parse!(
        tag_no_case!("put") >>
        multispace          >>
        key: word           >>
        multispace          >>
        value: rest_str     >>
        (DBCommand::Put(key.to_string(), value.to_string()))
    )
);

// Delete by key command
named!(cmd_delete<&[u8], DBCommand>,
    do_parse!(
        tag_no_case!("delete") >>
        multispace             >>
        key: word              >>
        (DBCommand::Delete(key.to_string()))
    )
);
```

As you can see our `parse_command` function now does what we want by taking a `&[u8]` and returning a `DBCommand`. Each
command is created using the `named!` macro provided by nom and the accompanying `do_parse` macros are fairly easy to
understand without needing to fully grasp the complexity of how parsers work. Keys and values can be extracted from their
space delimited position. For example the bytes of the string "GET user-1" are converted into
`DBCommand::Get("user-1")` by matching the string "get" uppercase or lower case, then one or more white spaces and then
the rest of the bytes are extracted as the string for the key we want to get.

Once we have our parser and codec we need to define how clients should interact with our server via the line protocol
we sketched out at the beginning of this post.

## Implementing The Line Protocol

Tokio provides the tools for implementing many of different types of protocols via the
[tokio-proto](https://github.com/tokio-rs/tokio-proto) crate. You can read more about the different protocol types
[here](https://docs.rs/tokio-proto) but broadly speaking they can either be **pipelined** (server responds to client
requests in the order they were sent) or **multiplexed** (the server responds to client requests in the order of
completion and request IDs are used to match responses back to requests). They can also be **streaming** protocols
(requests and responses can carry body streams which allows partial processing before the complete body has been
transferred) or **non-streaming** (the client sends a complete request in a single message, and the server provides a
complete response in a single message).

For our line protocol we don't need to stream requests and responses and we don't want to have to match request and
response IDs so we can go with the simple pipelined and non-streaming `tokio_proto::pipeline::ServerProto`.

We can add this to our app by creating `src/server/protocol.rs` with the following contents:

```rust
use std::io;
use tokio_io::{AsyncRead, AsyncWrite};
use tokio_io::codec::Framed;
use tokio_proto::pipeline::ServerProto;

use super::codec::LineCodec;
use DBCommand;

pub struct LineProto;

impl<T: AsyncRead + AsyncWrite + 'static> ServerProto<T> for LineProto {
    // For this protocol style, `Request` matches the `Item` type of the codec's `Decoder`
    type Request = DBCommand;

    // For this protocol style, `Response` matches the `Item` type of the codec's `Encoder`
    type Response = String;

    // A bit of boilerplate to hook in the codec:
    type Transport = Framed<T, LineCodec>;
    type BindTransport = Result<Self::Transport, io::Error>;
    fn bind_transport(&self, io: T) -> Self::BindTransport {
        Ok(io.framed(LineCodec))
    }
}
```

Here we first define our `LineProto` struct and then implement the `ServerProto` trait by defining the `Request` and
`Response` types. We then build a tokio transport (the way that it sends and receives frames on its connection) and a
little more boilerplate to let tokio know how to bind the transport for use in servers.

## Implementing The Server

We can use the Tokio `Service` trait to implement our server. Services in Tokio are asynchronous functions from `Request`
to `Response` that are decoupled from their underlying protocols. This abstraction allows us to have our server
implement many protocols (e.g. streaming and non-streaming) with a single implementation.

We can implement the `Service` trait for our server by creating the following `src/server/mod.rs` file:

```rust
use futures::{future, Future};
use std::io;
use std::sync::{Arc, Mutex};
use tokio_service::Service;

mod codec;
mod protocol;
mod request_parser;

pub use self::protocol::LineProto;
pub use self::codec::LineCodec;
use DB;
use DBCommand;

pub struct Server<T: DB> {
    db: Arc<Mutex<T>>,
}

impl<T: DB> Server<T> {
    pub fn new(db: Arc<Mutex<T>>) -> Server<T> {
        Server { db }
    }
}

impl<T: DB> Service for Server<T> {
    // These types must match the corresponding protocol types:
    type Request = DBCommand;
    type Response = String;

    // For non-streaming protocols, service errors are always io::Error
    type Error = io::Error;

    // The future for computing the response; box it for simplicity.
    type Future = Box<Future<Item = Self::Response, Error = Self::Error>>;

    // Produce a future for computing a response from a request.
    fn call(&self, req: Self::Request) -> Self::Future {
        let mut db = self.db.lock().unwrap();
        let response = db.process_command(req);
        Box::new(future::ok(response.unwrap().unwrap_or("None".to_string())))
    }
}
```

Our server struct has a reference to a `DB` instance and and implements the `call` method to produce a `Response` type
for a given `Request`. The body of our `call` method is pretty simple. Each connection the server receives can now
create a new server instance and pass in a thread safe reference to the database. The mutex ensures that only one thread
is accessing the database at any given time and with that we can have our database safely process each command and
return the result.

Our server is now implemented, but we are not yet able to receive requests until we specify how our server makes
itself available for connections. We can add that now.

## Listening On A Port

In order for computers with a single address to run multiple applications all connecting to the network, each application
must bind to a different port number. In the real world you might check the list that the Internet Assigned Numbers
Authority maintains of [registered port numbers](https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers). But
for our simple use case we will bind to port `12345` as an example.

Our `src/main.rs` file can now be re-written to be:

```rust
extern crate num_cpus;
extern crate tokio_minidb;
extern crate tokio_proto;

use tokio_minidb::{InMemoryDB, DB};
use tokio_minidb::server::{LineProto, Server};
use tokio_proto::TcpServer;

fn main() {
    // Specify the localhost address and port
    let addr = "0.0.0.0:12345".parse().unwrap();

    // Instantiate DB
    let db_handle = InMemoryDB::new().unwrap().handle();

    println!("Listening on port 12345");

    // The builder requires a protocol and an address
    let mut srv = TcpServer::new(LineProto, addr);
    srv.threads(num_cpus::get());
    srv.serve(move || Ok(Server::new(db_handle.clone())))
}
```

Now when our database application starts, it first creates a new in memory db instance and uses the `handle` method to
have a safe reference that each thread can use to process commands as they come in. It then uses the `TcpServer` builder
from the `tokio_proto` crate which accepts our line protocol and address as arguments. We then set the number of threads
that will be used to the total number of CPU cores on the current machine via the `num_cpus` crate, and finally call
`serve` which will block on the TCP socket and for each incoming connection, create a new `Server` instance with a
reference to the db to process the request.

## Running the server

You can clone the code from [jtescher/tokio-minidb](https://github.com/jtescher/tokio-minidb/tree/v2) if you want
to run this on your machine.

If you do not have telnet on your computer you can install it with [homebrew](https://brew.sh) `$ brew install telnet`.
You should now be able to run the database with `$ cargo run` and use telnet to connect. You can even do this in multiple
tabs and it will process your connections concurrently!


```shell
$ telnet localhost 12345
Trying ::1...
telnet: connect to address ::1: Connection refused
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
get user-1
None
put user-1 '{"first_name": "John", "last_name": "Doe"}'
None
get user-1
'{"first_name": "John", "last_name": "Doe"}'
```

## For Next Time

You now have a working in memory database that multiple clients can connect to and keep their data in sync. You saw how
to build simple parsers with Nom and how the Tokio stack helps you define codecs for encoding and decoding data,
protocols for defining how requests and responses are sent between clients and servers, and services for processing
requests and returning responses.

This program is still far from complete. One major issue that is that the data is only persisted as long as the server is
running. Every restart of the application loses all data. This might be ok for some caching applications, but if we want
to see how more general purpose databases work we will have to add some form of persistence and recovery from disk. In
the next post I will show you how to do just that via what is called a Write Ahead Log.
