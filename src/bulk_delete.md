# Bulk Delete

Bulk delete is a way to write SQL delete statements with in welds.
Bulk delete is accessed on [QueryBuilder\<T\>](./query.md) via `delete`. 

To use bulk delete call delete with the client.



```rust
    Product::all()
        .where_col(|x| x.name.ilike("% - discontinued") )
        .delete(&client).await?;
```
