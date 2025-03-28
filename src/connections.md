# Connections

Connections in welds are done using the crate `welds-connections`. This crate is re-exported out of welds for ease of use.
`welds-connections` is a common interface that allows for talking to different databases in a common way.

You can create a database client for a specific database backend, but generally It is recommended to use the generic connection that supports whatever backends you have enabled.
To do this you make a call into welds connections passing a connection_string.

```rust
let connection_string = "sqlite::./database.sqlite";
let client = welds::connections::connect(connection_string).await?;
```


## Client

A couple important things to understand about the client.

1) Client is an `Arc<>` feel free to call `clone()` on it as much as you want.
2) Client is a Connection Pool. It contains multiple connections and will handle sharing this resource for you.



## Connection String
Connection string follow the format of there corresponding database backends.

For SQLx you use a URL style connection string with the database backend being the protocol.

Valid backends are
```
postgres://
mysql://
sqlite://
```

And follow the format:
```
postgres://[user[:password]@][host][:port][/dbname][?params]
```

Microsoft SQL Server uses the ADO.NET style connection that they have documented [here](https://learn.microsoft.com/en-us/sql/connect/ado-net/connection-string-syntax?view=sql-server-ver16)

```rust
// export DATABASE_URL='server=127.0.0.1;user id=sa;password=password!123;Database=AdventureWorksDW2022;TrustServerCertificate=true;'
let connection_string = env::var("DATABASE_URL").unwrap();
let client = welds::connections::connect(connection_string).await?;
```
