# Whats Added To your Model

When you use the Welds ORM derive on your struct, it automatically provides several methods to help you create, retrieve, query, and manage data. Here’s an overview of those methods:

<br/>

## YourModel::new()
Purpose: Creates a new instance of the model in memory (not yet inserted into the database).
call `save()` to insert into the database.

Usage:
```
let mut model = YourModel::new();
model.save(client.as_ref()).await?;
```

Typical Use Cases:
- Create an empty model to fill with data.
- Prepare a model before saving to the database.

Alternatives:
Sometimes you want to compile time check that all fields have values. In this case it is recommended to create your new instance with state.
```rust
let mut model = DbState::new_uncreated(Model {
  id: 0, /* A default value like 0 will be filled in from database */
  name: "Bobby".to_owned(),
});
model.save(client.as_ref()).await?;
// The new id is updated on the model.
assert_eq( model.id, 1);
```


<br/>
<br/>
<br/>

## YourModel::all()

Purpose: Returns a query (un-executed) of all rows in the table.

Usage:
```rust
let query = YourModel::all();
```
Typical Use Cases:
- Starting place for querying a table.
- Starting place for querying a related table using `map_query`.

<br/>
<br/>
<br/>

## YourModel::find_by_id()

Purpose: Finds a single record by its primary key ID.

Usage:
```rust
let id = 42;
let maybe_record = YourModel::find_by_id(client.as_ref(), id).await?;
if let Some(record) = maybe_record {
    // Found the record with ID = 42
} else {
    // No record with that ID
}
```

Typical Use Case:
- Retrieve a single row when you know its primary key.


<br/>
<br/>
<br/>

## YourModel::where_col()

Purpose: Returns a query (un-executed) of a filtered set of rows in the table.

[You can read a detailed explanation of basic queries here](./query_basic.md)

Usage:
```rust
let query = YourModel::where_col(|col| col.name.equal("some_name"));
```

Typical Use Cases:
- Starting place for querying a table.
- Starting place for querying a related table using. `map_query`.
- Used to pass into other queries to help filter them more. (sub-query) `where_relation`

<br/>
<br/>
<br/>


## YourModel::from_raw_sql()

Purpose: Execute custom or raw SQL that maps directly to your model fields.

It is recommended to avoid if you can, but it is recognized that welds doesn't do everything and sometimes you just need raw SQL.

Usage:
```rust
let name = "bobby".to_string();
let params: welds::query::clause::ParamArgs = vec![&42, &name];
let profiles = Profile::from_raw_sql(
    "select * from profiles where id > ? OR name = ?",
    &params,
    client.as_ref(),
)
.await?;
```

Typical Use Cases:
- Complex queries not easily expressed with the provided query builder.
- Performing specialized joins, aggregates, or database-specific functionality, while still returning YourModel items.


<br/>
<br/>
<br/>


## YourModel::select()

Purpose: Starts a customizable “select” query builder for this model.
This is used to return a sub-set of your columns on your model. You can also start a "select" query off of any query such as where_col.

[You can read a detailed explanation of select queries here](./query_select.md)

***NOTE: You will want a second model that represents your subset of columns***

Usage:

Subset Model
```rust
// Because all you need is deserealization. All you need to add is WeldsModel

#[derive(WeldsModel)]
pub struct IdAndName {
    pub id: i32,
    pub name: String,
}
```

Query filling your subset model
```rust
let query = YourModel::select(|x| x.id).select(|x| x.name);
let data: Vec<IdAndName> = query.run(client.as_ref()).await?.collect_into()?;
```

Typical Use Cases:
- When you want only some of the data from your table.
- When you want columns of data across table. (Join / Left Join)

