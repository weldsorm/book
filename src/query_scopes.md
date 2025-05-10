# Scopes

Welds supports name scopes to help abstract away complicated queries.

## Introduction

Often when working with data in a database there is a need to reuse filters.
This can be useful to keep complicated logic in a single location.
Welds provides a nice mechanism to write reusable snippets of queries.

## How to write a scope 

To write a scope in welds start off with a trait defining your scope.

```rust
pub trait ProductScopes {
    fn ready_to_sell(self) -> Self;
}
```

Next implement your trait for the querybuilder and your model.

```rust
impl ProductScopes for QueryBuilder<Product> {
    fn ready_to_sell(self) -> Self {
        self.where_col(|p| p.active.not_equal(false))
            .where_col(|p| p.price.not_equal(None))
            .where_col(|p| p.description.not_equal(None))
    }
}
```

## How to use a scope

Once written, your scopes are available any time you write a query as long as your trait is included in the module.


```rust
// top of file
use ProductScopes;



let data = Product::all().ready_to_sell().run(&client).await?;
```




