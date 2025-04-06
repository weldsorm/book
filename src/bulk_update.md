# Bulk Update

Bulk update is a way to write SQL update statements with in welds.

Bulk updates are accessible on [QueryBuilder\<T\>](./query.md). To start a bulk update you can call one `set` operations on your query.
`set`, `set_col`, `set_null`, `set_manual`

<br/>

## set / set_col

`set` and `set_col` are functionally identical.
They just provide two coding flavors to the same underlying set operation.

To use them start off with a Query just like you would to select your data.

```rust
    let q = Product::where_col(|x| x.name.ilike("% - discontinued") );
```

You can then call `set` or `set_col` to convert this query into a bulk update

```rust
    let q = Product::where_col(|x| x.name.ilike("% - discontinued") );

    q.set(|o| o.hidden, true).run(&client).await?;
    // or
    q.set_col(|o| o.hidden.equal(true) ).run(&client).await?;

```

## set_null

`set_null` is just a short hand for `set(_, None)`.
It is nice in that it makes your code cleaner to read.

```rust
    let q = Product::where_col(|x| x.name.ilike("% - discontinued") );

    q.set_null(|p| p.category).run(&client).await?;

```

## set_manual

Sometimes you need to set a column based on another columns value; for example a calculation. 
In this scenario `set_manual` is what you are looking for. 
It functions in the same way as the other [manual sql](./query_sql.md) calls

To use it start off with a query,
but this time you can insert a snippit of generic SQL to set the columns value.

```rust
    Order::all()
        .set_manual(|x| x.sell_price, "$.sell_price + ?", (88,))
        .run(&client).await?;
```

NOTE: use `?` for all database backends. It will be translated appropriately for the backends being called.










