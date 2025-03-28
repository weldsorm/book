# Models

Models are the heart of Welds. All queries are made off of the models you define.
Additionally models are used to de-serialize rows from the database,
so they might not always represent corresponding tables or views.

Relationships are also defined on your models.
These relationships allow for complex queries like joins and sub-queries.


# Basic 

At its very simplest all that is needed is a `#[derive(WeldsModel)]`. This derive gives you basic deserialization. 
If you are working with other models, and selecting into a basic struct, that might be all you need. but most likely you will want a little more.
A better basic example would be as follows.

```rust
#[derive(WeldsModel)]
#[welds(schema = "dto", table = "pets")]
pub struct Pet {
    #[welds(primary_key)]
    pub id: i32,
}
```

This model is fully functional. It can be loaded from the database; saved to the database; or queried in the database.

For example a basic query might look something like this.
```rust
let pets = Pet::all().where_col(|x| x.id.not_equal(42)).run(client.as_ref()).await?;
```


