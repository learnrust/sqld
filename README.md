# `sqld` - a server mode for libSQL

The `sqld` ("SQL daemon") project is a server mode for [libSQL](https://libsql.org).

Embedded SQL databases such as libSQL and SQLite are great for a lot of use cases, but sometimes you really do want to consume your database as a server.
For example, with serverless apps, fitting a database engine, as slim as it may be, might be hard, and even when it's _possible_, it might be really inconvenient, which is why we created `sqld`.

With `sqld` you can use SQLite-like interface in your app, but have a transparent proxy translates the C API calls to PostgreSQL wire protocol to talk to a database server, which is internally libSQL.

_Disclaimer: although you can connect to `sqld` with a PostgreSQL client and many things just work because PostgreSQL and SQLite are so similar, the goal of this project is to provide SQLite compatibility at the client.
That is, whenever there's a conflict in behaviour between SQLite and PostgreSQL, we always go with SQLite behavior.
However, if libSQL starts to provide more [PostgreSQL compatibility](https://github.com/libsql/libsql/issues/80), `sqld` will also support that._

## Features

* SQLite dialect layered on top of HTTP or the PostgreSQL wire protocol.
* TypeScript/JavaScript client
* SQLite-compatible API that you can drop-in with `LD_PRELOAD` in your application to switch from local database to a remote database.
* Read replica support.
* Integration with [mvSQLite](https://github.com/losfair/mvsqlite) for high availability and fault tolerance.
 
## Roadmap

* Client authentication and TLS
* Integration with libSQL's [bottomless storage](https://github.com/libsql/bottomless)

## Getting Started

Start a server with a postgres and http listeners, writing to the local SQLite-compatible file `foo.db`:

```console
sqld -d foo.db -p 127.0.0.1:5432 --http-listen-addr=127.0.0.1:8000
```

connect to it with psql:

```console
psql -h 127.0.0.1
```

or HTTP:

```console
curl -s -d '{\"statements\": [\"SELECT * from sqlite_master;\"] }' http://127.0.0.1:8000
```

You can also inspect the local foo.db file with the `sqlite3` shell

### Integration with S3 bottomless replication

`sqld` is integrated with [bottomless replication subproject](https://github.com/libsql/sqld/tree/main/bottomless). With bottomless replication, the database state is continuously backed up to S3-compatible storage. Each backup session is called a "generation" and consists of the main database file snapshot and replicates [WAL](https://www.sqlite.org/wal.html) pages.

In order to enable automatic replication to S3 storage, run `sqld` with `--enable-bottomless-replication` parameter:
```console
sqld --http-listen-addr=127.0.0.1:8000 --enable-bottomless-replication
```

#### Configuration
Replication needs to be able to access an S3-compatible bucket. The following environment variables can be used to configure the replication:
```sh
LIBSQL_BOTTOMLESS_BUCKET=my-bucket                 # Default bucket name: bottomless
LIBSQL_BOTTOMLESS_ENDPOINT='http://localhost:9000' # address can be overridden for local testing, e.g. with Minio
AWS_SECRET_ACCESS_KEY=                             # regular AWS variables are used
AWS_ACCESS_KEY_ID=                                 # ... to set up auth, regions, etc.
AWS_REGION=                                        # .
```

#### bottomless-cli
Replicated snapshots can be inspected and managed with the official command-line interface.

The tool can be installed via `cargo`:
```console
RUSTFLAGS='--cfg uuid_unstable' cargo install bottomless-cli
```
For usage examples and description, refer to the official bottomless-cli documentation: https://github.com/libsql/sqld/tree/main/bottomless#cli

## Homebrew

You can install `sqld` through homebrew by doing:

```
brew tap libsql/sqld
brew install sqld-beta
```

Note that until a stable version is released, it is provided as a separate tap, with a `beta` suffix.

## Clients

`sqld` ships with a native Javascript driver for TypeScript and Javascript. You can find more information [here](https://www.npmjs.com/package/@libsql/client)

## Building from Sources

### Dependencies

**Linux:**
 Tested on CentOS Stream 9, Fedora 34+, MacOS with brew  and ubuntu 20+
```console
./scripts/install-deps.sh
```

### Submodules

```run
git submodule update --init --force --recursive --depth 1
```

### Building

```console
cargo build
```

### Running tests

```console
make test
```

## License

This project is licensed under the MIT license.

### Contribution

Unless you explicitly state otherwise, any contribution intentionally submitted for inclusion in `sqld` by you, shall be licensed as MIT, without any additional terms or conditions.
