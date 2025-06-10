# Migrations

Welds has built in migrations that offer a couple nice benefits SQL script style migrations do not.

1) You can write them once and they work with all backends
2) Welds migrations are database aware, so they automatically generate down migrations.
3) You can always fallback to raw SQL style migrations with the `Manual` migrations struct.

To use migrations make sure the migrations feature flag is enabled.
```bash
cargo add welds --feature=migrations
```

## Common Setup / layout

Feel free to layout migrations in your project how you see fit, but if you are looking for a simple starting place:

1) make a migrations module `src/migrations/mod.rs`
2) add an `up` style function to run all your migrations. migrations that have already executed will be skipped.

```rust 
/*  ./src/migrations/mod.rs  */

use welds::errors::Result;
use welds::migrations::prelude::*;

pub async fn up(client: &dyn welds::TransactStart) -> Result<()> {
    let list: Vec<MigrationFn> = vec![
        m20250210064356_create_table_dogs::step,
    ];
    welds::migrations::up(client, list.as_slice()).await?;
    Ok(())
}

mod m20250210064356_create_table_dogs;
```

3) keep adding your migrations as sub-modules as you need them.
4) call `crate::migrations::up(&client).await?;` when your app boots.


## Types

`welds::migrations::Types` is an Enum of generic sounding types that welds knows how to translate into Rust code and each database backend. 
Stick to these types if you can. Sometimes you have an odd SQL Type you need to work with.
You can do so with `Types::Raw(XXX)`. This variant allows full control of what type is used in SQL.


## Create Table

`create_table` is a function to generate a migration to create a table.
It is a build pattern style call.
Inside each call to column lambda is a nested build pattern style call for each column.

A vary plain example might look like this.
```rust
use welds::errors::Result;
use welds::migrations::prelude::*;

pub(super) fn step(_state: &TableState) -> Result<MigrationStep> {
    let m = create_table("dogs")
        .id(|c| c("id", Type::IntBig))
        .column(|c| c("name", Type::String))
        .column(|c| c("wags", Type::Int));
    Ok(MigrationStep::new("m20250210064356_create_table_dogs", m))
}
```

By default all columns are not null. We can change this by adding onto the columns lambda call.
```rust
    let m = create_table("dogs")
        .id(|c| c("id", Type::IntBig))
        .column(|c| c("name", Type::String).is_null())
        .column(|c| c("wags", Type::Int));
}
```

Notable column settings:
- ***is_null()*** sets the column as nullable
- ***with_index()*** adds an index to the column
- ***with_index_name(&str)*** adds an index to the column, with a name that you provide
- ***with_unique_index()*** Adds a unique constraint on this column
- ***create_foreign_key(\_,\_,\_)*** Adds a foreign key (foreign_table, foreign_column, delete_behavior)


## Alter Table - Alter Columns - Delete Table

Modifications to tables including table drops are accessible from a call to `change_table`.

### Dropping a table
```rust
fn step(state: &TableState) -> Result<MigrationStep> {
    let m = change_table(state, "dogs")?.drop();
    Ok(MigrationStep::new("m20250210064357_drop_table_dogs", m))
}
```

### Add Column
```rust
fn step(state: &TableState) -> Result<MigrationStep> {
    let alter = change_table(state, "dogs")?;
    let m = alter.add_column("belly_rubs", Type::Int);
    Ok(MigrationStep::new("m20250210064357_add_column", m))
}
```

### Change a column
```rust
fn step(state: &TableState) -> Result<MigrationStep> {
    let alter = change_table(state, "dogs")?;
    let m = alter.change("belly_rubs")
        .to_type( Type::IntBig )
        .null() // make nullable
        .rename("rubs");
    Ok(MigrationStep::new("m20250210064357_alter_column", m))
}
```




## Manual SQL

Welds always tries to provide a fallback for a manual option.
Migrations are no exception.
You can always write manual raw SQL to make database changes if needed.

Manual migrations are ran within a transaction and support running multiple commands separated by a semicolon ";"

```rust
use welds::errors::Result;
use welds::migrations::prelude::*;
use welds::migrations::Manual;

fn step(state: &TableState) -> Result<MigrationStep> {
    let m = Manual::up("CREATE TABLE dogs ( id SERIAL PRIMARY KEY )")
        .down("DROP TABLE dogs");
    Ok(MigrationStep::new("m20250210064357_manual_dogs", m))
}
```
