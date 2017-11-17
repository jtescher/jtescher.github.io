---
layout: post
title:  "Writing A Key Value Store In Rust Part Three: Persistence"
date:   2017-11-12 09:00:00
categories: rust databases
---

![Library](/assets/writing-a-key-value-store-in-rust-part-three/library.jpg)

_This is part three of a series on learning intermediate programming concepts by writing a database from scratch in Rust.
You can read part two [here](/writing-a-key-value-store-in-rust-part-two) first to catch up_.

In the previous post we connected our server to the network and allowed multiple clients to execute commands over a
simple line protocol. We can now think about how we want to save the data so it can be persisted across database
restarts and resistant to application crashes. But how would we want to do this? Let's start with some data persistence
and recovery options.

As usual all of the code for this project is available on github at
[jtescher/tokio-minidb](https://github.com/jtescher/tokio-minidb/tree/v3).

## Storing Data On Disk

Here is a quick overview of a few of the database persistence options for storing data on disk:

1. **Flat File**: One of the simper implementations of persistence is a [flat file](https://en.wikipedia.org/wiki/Flat_file_database).
Using this method, all data in the file must be read in its entirety by the application when it starts, and must then be
written back to disk before the application stops. This might be a plain text or a binary file, and it is "flat" because
it has no structure for indexing and it usually has no structural relationships between the records. This simple format
was often used for storing data in the '80s and is still useful today for applications that want to persist small
amounts of editable data like configuration files.
1. **Indexed Sequential Access Method (ISAM)**: A more complex method is [ISAM](https://en.wikipedia.org/wiki/ISAM) in
which data is organized into records which themselves are composed of fixed length fields. Records are stored
sequentially in files and a secondary set of hash tables are used as indexes and point to records in the tables. A
benefit of ISAM is that the indexes are relatively small and can be searched quickly. This allows the database to access
only the records it needs and makes updates only need to write to change one file and the corresponding indices,
leaving the rest of the files are unchanged. Some popular ISAM-style databases include MySQL via
[MyISAM](https://en.wikipedia.org/wiki/MyISAM) and [Berkeley DB](https://en.wikipedia.org/wiki/Berkeley_DB).
1. **B+ Tree**: [B+ trees](https://en.wikipedia.org/wiki/B%2B_tree) are trees with a few distinct properties that make
them convenient for storing database records. They have large number of children per node (which reduces the number of
IO operations needed to find an element), the nodes contain only keys and pointers to the data they store (potentially
separating the key storage from the value storage), and each leaf at the bottom of the tree is linked together allowing
traversal of the stored values. This structure is used as indices in databases like
[PostgreSQL](https://www.postgresql.org/docs/9.6/static/indexes-types.html) and
[MySQL](https://dev.mysql.com/doc/refman/5.7/en/innodb-physical-structure.html) and as the data store for databases like
[CouchDB](https://en.wikipedia.org/wiki/CouchDB) and
[Tokyo Cabinet](https://en.wikipedia.org/wiki/Tokyo_Cabinet_and_Kyoto_Cabinet).
1. **Log-Structured Merge (LSM) Tree**: [LSM trees](https://en.wikipedia.org/wiki/Log-structured_merge-tree) are a more
complicated structure that typically are made of two trees, a small in memory tree and a separate large one on disk.
When you want to add a new record, it is first added to the in memory tree and when that tree reaches a certain size it
gets flushed to the on disk tree as a contiguous block of records. LSM trees often have multiple levels and in order to
keep reads performant these levels are often merged together into sorted blocks. These are used by many modern databases
like [RocksDB](http://rocksdb.org), [Cassandra](https://cassandra.apache.org), and [MongoDB](https://www.mongodb.com).

For crash recovery, almost all databases chose to have a Write Ahead Log (WAL) file that stores each command before
the command is considered successful. These files come in many formats but they all allow applications to replay state
either from the beginning or from a known snapshot to recover the operations that were committed before the crash.

The on-disk persistence methods listed above are useful when you need to store more data than you can hold in memory,
but for our toy database we can get away with assuming small data sizes. This allows us to have full persistence and
recovery by simply adding a WAL file and recovering from it every time the app starts.

## Adding the WAL

In order to log commands, we can create a `src/storage/persistent_db` directory and add the following to
`src/storage/persistent_db/wal.rs`:

```rust
use std::fs::{File, OpenOptions};
use std::io;
use std::io::Write;

pub struct WAL {
    inner: File,
}

impl WAL {
    pub fn new() -> Result<Self, io::Error> {
        let inner = OpenOptions::new()
            .read(true)
            .append(true)
            .create(true)
            .open("target/wal.txt")?;

        Ok(WAL { inner })
    }

    pub fn into_buf_reader(self) -> io::BufReader<File> {
        io::BufReader::new(self.inner)
    }

    pub fn write_all(&mut self, buf: &[u8]) -> Result<(), io::Error> {
        self.inner.write_all(buf)
    }
}
```

This struct wraps a `File` and exposes a simple API for writing and buffered reading. Ordinarily you would allow the
location of the WAL file to be customized via environment variable or config file, but in our case we can just write it
to a known location like our target directory as a text file. We use the `read(true)` option so we can  read the contents
of the WAL file and `create(true)` and `append(true)` options to tell Rust to create a new file if one does not yet
exist, and that writes should be appended to the file instead of overwriting what is there.

Now that we have the WAL defined, we can create a persistent db version that uses it for persistence and recovery.

## Creating A Persistent Store

To create a persistent store, we can now by pair the WAL file with our existing in memory store. We
can define this persistent store in `src/storage/persistent_db/mod.rs` with the following:

```rust
mod wal;

use bytes::BytesMut;
use std::io;
use std::io::BufRead;
use tokio_io::codec::Decoder;

use {DBIterator, DBResult, DB};
use server::LineCodec;
use storage::in_memory_db::InMemoryDB;
use self::wal::WAL;

pub struct PersistentDB {
    storage: InMemoryDB,
    wal: WAL,
}

impl PersistentDB {
    fn recover_from_wal(db: &mut InMemoryDB) -> Result<(), io::Error> {
        let wal = WAL::new()?;
        let mut codec = LineCodec;
        for command in wal.into_buf_reader().lines() {
            let mut command = BytesMut::from(command.unwrap_or("".to_string()));
            command.extend("\r\n".as_bytes());
            db.process_command(codec.decode(&mut command)?.unwrap())?;
        }
        Ok(())
    }
}

impl DB for PersistentDB {
    fn new() -> Result<PersistentDB, io::Error> {
        let mut storage = InMemoryDB::new()?;
        let wal = WAL::new()?;

        PersistentDB::recover_from_wal(&mut storage)?;

        Ok(PersistentDB { storage, wal })
    }

    fn put(&mut self, key: String, value: String) -> DBResult {
        self.wal
            .write_all(format!("put {} {}\n", key, value).as_bytes())
            .and_then(|_| self.storage.put(key, value))
    }

    fn get(&self, key: &String) -> DBResult {
        self.storage.get(key)
    }

    fn delete(&mut self, key: &String) -> DBResult {
        self.wal
            .write_all(format!("delete {}\n", key).as_bytes())
            .and_then(|_| self.storage.delete(key))
    }

    fn iter(&self) -> DBIterator {
        self.storage.iter()
    }
}
```

We first define a `PersistentDB` struct that will hold our WAL file and an in memory db. Our `new` method can now
initialize the in memory store and call `recover_from_wal` which will iterate through each line of the WAL and decode
the command using the codec we defined in the last post. It can then have the database process each command to replay
all of the command history.

In order to `put` or `delete` for the persistent db, we simply write the commands to the WAL and then call the
corresponding method on the in memory db to keep everything in sync.

## Switching To The Persistent Store

To switch to the persistent store we first have to expose the `persistent_db` module. We can add this to
`src/storage/mod.rs` making it:

```rust
pub mod in_memory_db;
pub mod persistent_db;
```

And to `src/lib.rs`:

```rust
...
pub mod server;
mod storage;
pub use storage::in_memory_db::InMemoryDB;
pub use storage::persistent_db::PersistentDB;
...
```

And finally we can use it in our `src/main.rs` file:

```rust
extern crate num_cpus;
extern crate tokio_minidb;
extern crate tokio_proto;

use tokio_minidb::{PersistentDB, DB};
use tokio_minidb::server::{LineProto, Server};
use tokio_proto::TcpServer;

fn main() {
    // Specify the localhost address
    let addr = "0.0.0.0:12345".parse().unwrap();

    // Instantiate DB
    let db_handle = PersistentDB::new().unwrap().handle();

...
```

## Running The Server

We now have a persistent database that is fault tolerant! This can be tested by starting the server with `$ cargo run`,
then executing a few commands:

```shell
telnet localhost 12345
Trying ::1...
telnet: connect to address ::1: Connection refused
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
put key-1 value-1
None
put key-2 value-2
None
get key-1
value-1
get key-2
value-2
```

Then stopping the server and starting it again with `$ cargo run` and the values should still be there!

```shell
...
get key-2
value-2
Connection closed by foreign host.

$ telnet localhost 12345
Trying ::1...
telnet: connect to address ::1: Connection refused
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
get key-1
value-1
```

Success! The database is now much more useful. In future posts I will add other common features like authentication,
clustering, snapshoting, and on disk persistence so the amount of data stored could exceed available memory.
