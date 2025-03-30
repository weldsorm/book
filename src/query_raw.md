# Raw SQL

Welds has the mindset that you should be able to drop down to SQL if you need it.
We provide several ways to use SQL snipits in your queries, but this chapter is how to bypass the QueryBuilder altogether.

Your `client` from when you connected has everything on it that you could need. 
Most likely you have connected with a generic `AnyClient` connection. `AnyClient` 
and the four specific variants all implement the two traits `welds_connections::Client` and `welds_connections::TransactStart`. 
These two trans are the common way to tall to any database.

<br/>

It should be noted while the interface is common between the different database backends, the database backend all expect their own slightly different flavor of SQL.
If you are writing raw SQL, you will need to write it for all the backends you would like to support.


## execute

If you aren't expecting to get any data back from the database, `execute` is what you are looking for.
You pass in raw SQL with SQL parameters, and the database runs it for you.

```rust
    use welds::prelude::*;
    let client = welds::connections::connect(connection_string).await?;
    let text: String = "I'm a String".to_owned();
    client.execute("select ?, ?", &[&2, &text]).await?;
```

There are a couple things worth pointing out in this example.
1) Make sure you are including the `prelude`. It includes the traits you need
2) Those parameters are a little crazy looking. Are They mixing types?

The parameters are actually just a slice of borrowed `dyn Param` (can be sent to database).
This is why all parts of it are borrowed `&`.

A much more verbose way of writing this might look like this.

```rust
    use welds::prelude::*;
    use welds::connections::Param;
    let mut params: Vec<&(dyn Param + Sync)> = Vec::default();
    let text: String = "I'm a String".to_owned();
    params.push(&2);
    params.push(&text);
    client.execute("select ?, ?;", &params).await?;
```

## fetch_rows, Into Model

`fetch_rows` works exactly like `execute` except it returns a `Vec<Row>`.
All welds models impl `TryFrom<Row>`.
This allows you to read out raw values from each row or map them into a `WeldsModel`.

The welds model
```rust
#[derive(WeldsModel)]
struct Car {
    id: i32,
}
```

Running the raw SQL and mapping into your struct
```rust
    use welds::prelude::*;
    let client = welds::connections::connect(connection_string).await?;
    let rows = client.execute("select id from cars", &[]).await?;
    let cars: Vec<Car> = rows.collect_into()?;
```

There are a couple things to note about this example.
1) The mapping into the model is done by ***column name***. 
Make sure the field name on your model matches what is being selected.
2) `collect_into()` is a helper method extension added to `Vec<Row>`


## fetch_rows, By Hand

If you are just reading a single value like a count or total, it might be easier to avoid a welds model.
Here are two examples that read a value directly from a row.

```rust
    use welds::prelude::*;
    let rows = client.fetch_rows("select 1 + 1 as total;", &[]).await?;
    let first_row = rows.first().unwrap();
    let total: i32 = first_row.get("total")?;
    println!("Total: {total}");
```

You can read values by position in the row with `get_by_position`.
Note it is zero based.

```rust
    use welds::prelude::*;
    let rows = client.fetch_rows("select 1 + 2;", &[]).await?;
    let first_row = rows.first().unwrap();
    let total: i32 = first_row.get_by_position(0)?;
    println!("Total: {total}");
```

## Fetch Many

Last we have one more function that is used by welds internally for optimizations and you can use it too if you wish. 
`fetch_many` takes in a slice of `welds_connections::Fetch` (pairs of sql and params).

This is specifically useful when you want to force multiple database calls onto the same connection in a database pool.
There are a couple niche use cases you might want to do this, but most likely `fetch_rows` is what you want.

```rust
    use welds::prelude::*;
    use welds::connections::Fetch;

    let fetches = vec![
        Fetch {
            sql: "select 1",
            params: &[],
        },
        Fetch {
            sql: "select 2",
            params: &[],
        },
    ];

    let groups_of_rows = client.fetch_many(&fetches).await?;
    for rows in groups_of_rows {
        // work with each individual set of rows
    }

```


