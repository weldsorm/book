# Project Setup

For the most part there isn't really much extra you need to do over a normal crate.
It is recommended to install Welds with a simple cargo add command, but you will need to pick your database backends you want to support.
These are enabled as features on the Welds crate.

You will also need to setup `SQLx` or `Tiberius` as well. `Tiberius` for Microsoft SQL Server and SQLx for the other backends.
This has been left up to you because these library support multiple async backends and you need to pick which one to use.

You can also add more features as you need, but a couple simple example would be:

### Welds compiled for Sqlite
```
cargo add welds --features="sqlite"
cargo add sqlx --features="runtime-tokio"
```

### Welds compiled for Postgres
```
cargo add welds --features="postgres"
cargo add sqlx --features="runtime-tokio,tls-rustls"
```

### Welds compiled for Microsoft Sql Server
```
cargo add welds --features="mssql"
cargo add tiberius
```
##
For external extra types you will need to enable them in the underlying database driver.

### Welds compiled for Sqlite with chrono support
```
cargo add welds --features="sqlite"
cargo add sqlx --features="runtime-tokio,chrono"
```

## 
For external extra types with Mssql you will need to enable them in welds-connections too.

### Welds compiled for Sqlite and Microsoft Sql Server with chrono and uuid support
```
cargo add welds --features="sqlite,mssql"
cargo add sqlx --features="runtime-tokio,chrono,uuid"
cargo add tiberius --features=chrono
cargo add welds-connections --features=mssql,sqlite,mssql-chrono
```

