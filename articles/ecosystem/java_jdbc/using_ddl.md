---
title: "Using DDL and Metadata"
layout: article
---

## Contents

* [Overview][overview]
* [Using SQL][using-sql]
* [Using DDL][using-ddl]
* [Reusing Connections][reusing-connections]

## Using DDL

DDL operations can be executed using the `db-do-commands` function. The general
approach is:

```clojure
(j/db-do-commands db-spec [sql-command-1 sql-command-2 .. sql-command-n])
```

The commands are executed as a single, batched statement, wrapped in a
transaction. If you want to avoid the transaction, use this approach:

```clojure
(j/db-do-commands db-spec false [sql-command-1 sql-command-2 .. sql-command-n])
```

### Creating tables

For the common operations of creating and dropping tables, java.jdbc provides a
little assistance that recognizes `:entities` so you can use keywords (or
strings) and have your chosen naming strategy applied, just as you can for
several of the SQL functions.

```clojure
(j/create-table-ddl :fruit
                    [[:name "varchar(32)" :primary :key]
                     [:appearance "varchar(32)"]
                     [:cost :int]
                     [:grade :real]]
                    {:table-spec "ENGINE=InnoDB"
                     :entities clojure.string/upper-case})
```
This will generate:

```clojure
CREATE TABLE FRUIT
    (NAME varchar(32) primary key,
     APPEARANCE varchar(32),
     COST int,
     GRADE real) ENGINE=InnoDB
```

which you can pass to `db-do-commands`.

### Dropping tables

Similarly there is a `drop-table-ddl` function which takes a table name and an
optional `:entities` option to generate DDL to drop a table.

```clojure
(j/drop-table-ddl :fruit) ; drop table fruit
(j/drop-table-ddl :fruit {:entities clojure.string/upper-case}) ; drop table FRUIT
```

This will generate:

```clojure
DROP TABLE FRUIT
```

## Accessing metadata

java.jdbc provides two functions for working with database metadata:

* `with-db-metadata` for creating an active metadata object backed by an open
  connection
* `metadata-result` for turning metadata results into Clojure data structures

For example:

```clojure
(j/with-db-metadata [md db-spec]
  (j/metadata-result (.getTables md nil nil nil (into-array ["TABLE" "VIEW"]))))
```

This returns a sequence of maps describing all the tables and views in the
current database. `metadata-result` only transforms `ResultSet` objects, other
results are returned as-is. `metadata-result` can also accept an options map
containing `:identifiers` and `:as-arrays?`, like the `query` function,
and those options control how the metatadata is transformed and/or returned.

[overview]: home.html
[using-sql]: using_sql.html
[using-ddl]: using_ddl.html
[reusing-connections]: reusing_connections.html
