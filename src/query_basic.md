# Basic - (single table query)

## Overview

Welds uses a builder-like API for constructing queries against your database models.
These queries are strongly typed, ensuring compile-time safety and making it clearer which columns and constraints are being used.
You can chain methods such as where_col(), order_by_asc(), and others to build increasingly complex queries without sacrificing readability.


## Executing Queries

Welds queries don't run in the database until you call `run` or `count`.
These calls build the required SQL, execute it in the database, and returns the corresponding data.
```rust
let row_count = Model::all().count(client.as_ref()).await?;
let models = Model::all().run(client.as_ref()).await?;
```

<br/>
<br/>


## where_col(...)

Use `where_col()` to filter rows based on one or more columns. 
The closure you provide gives you typed access to the model’s fields, letting you specify constraints such 
as `equal`, `not_equal`, `lt`, `lte`, `gt`, `gte`, `like`, etc. These filters are dependent on the underlying field type. (I.E. i32 doesn't have ilike)

Usage:
```rust
let query = Product::all()
    .where_col(|p| p.active.equal(true))
    .where_col(|p| p.name.like("%Cookie%"))
    .where_col(|p| p.price.not_equal(None));
```

#### List of filters by types:
- Everything
    - equal
    - not_equal
    - in_list (sql "in")
- Numbers (everything plus)
    - gt
    - lt
    - gte
    - lte
- Text (everything plus)
    - like
    - not_like
    - ilike
    - not_ilike

<br/>
<br/>

## limit()
Use `limit()` to restrict the number of rows returned by the query. 
This is particularly useful for pagination or if you only need a certain number of records.

Usage
```rust
let products = Product::all()
    .limit(10) // Only fetch up to 10 products
    .run(&client)
    .await?;
```

Typical Use Case
- Implementing pagination (in combination with offset() if needed).
- Quickly previewing only a small subset of rows without fetching an entire table.
- Performance optimization when you only need the first few rows.

<br/>
<br/>

## offset()
Use `offset()` to skip over a set of rows
This is particularly useful for pagination.

Usage
```rust
let products = Product::all()
    .limit(10) // Only fetch up to 10 products
    .offset(5) // skips the first 5 rows
    .run(&client)
    .await?;
```

Typical Use Case
- Implementing pagination (in combination with limit()).

<br/>
<br/>

## order_by_asc() *and* order_by_desc()

Use order_by_asc() and order_by_desc() to sort your query results in on a specified column. Calls to order_by and be compounded to make complex order by statements

Usage
```rust
let query = Product::all()
    .order_by_desc(|x| x.created_at)
    .order_by_asc(|x| x.name);
```

Variants

`order_by_asc_null_first` and `order_by_desc_null_first` also exist to help with null values in your returning dataset
