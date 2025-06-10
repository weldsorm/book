# Bulk Insert

When adding a lot of data into your database using a bulk insert can save a lot of time.
This has to do with the way databases work, and is true at the database level.

There are many different ways to "Bulk Insert" into a database, Welds tries to provide a common interface for the best option for each of its backends.

## bulk_insert(\_,\_)



The `bulk_insert` function is that common interface. It is pretty strait forward taking in a [Client](./fundamentals.md#weldsclient) and a slice of `WeldsModels`

bulk_insert is not accessible via the query builder like other operations. You will need to include it with:
```rust
    use welds::query::insert::bulk_insert;
```

First build a slice of data that you want to insert.

```rust
#[derive(WeldsModel)]
#[welds(table = "products")]
pub struct Product {
    #[welds(primary_key)]
    pub id: i32,
    pub name: String,
}
```

```rust
    let mut products: Vec<Product> = Vec::default();
    for i in (0..1000) {
        products.push( Product {
            id: 0,
            name: format!("product #{}", i)
        });
    }
```

<br/>

Things to notice:
1) The column `id` is marked as a `primary_key`. Typically in welds
if the primary_key field is `Default::default()` it is ignored when inserting.
Here in Bulk insert the primary_key is ***ALWAYS*** ignored.
If you want to insert a primary_key column, you can use the method: `welds::query::insert::bulk_insert_with_ids`
2) DbState is not used with bulk operations

You can now insert your data with a call to `bulk_insert`

```rust
    use welds::query::insert::bulk_insert;
    bulk_insert(&client, &products).await?;
```






