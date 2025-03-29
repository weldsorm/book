# Advanced - (cross table query)

****Not*** data from multiple tables. See [Custom Select](./query_select.md) for that*

Requirements: the following section requires that you have relationships setup on your models.

## Overview

In general when working with collections of data in code it is very common to see heavy use of
`map` and `filter` to transform a collection of data.

```rust
let good_players: Vec<&str> = players
    .filter(|p| p.goals > 3)
    .map(|p| p.name)
    .collect();
```

This style of programming can be pleasant to write, and easy to read. However it is limited to the data you have already loaded into memory.
Wouldn't it be nice if you could write this style of code that ran in the database? 
Welds tries to emulate this with the two functions `map_query` and `where_relation`. 
While it isn't an exact one to one, It is a good mindset to help understand these two concepts.

## map_query()

Lets say I want to fetch data on one table, and use that data to lookup another table.

A ***very bad*** way of doing this might be.
```rust
let people = People::all().run(&client).await?;
let addresses = people.map(|p| Address::find_by_id(&client, p.address_id).await.unwrap() ).collect();

// or

let people = People::all().run(&client).await?;
for person in people {
    let address = Address::find_by_id(&client, p.address_id).await?;
}
```

This will have very bad performance and will make many, many, many, database calls.
This is sometimes referred to as an "N+1 query".
Instead lets do the mapping in the database.

A ***much better*** way would be
```rust
let addresses = People::all().map_query(|p| p.addresses).run(&client).await?;
```

This does the equivalent mapping, but is much faster and easier to read.

This can of course be combined with other queries.
```rust
// get Bob's addresses
let addresses = People::where_col(|p| p.name.ilike("bob"))
    .map_query(|p| p.addresses).run(&client).await?;
```



## where_relation() 
where_relation is the other side of this coin. It is a way to filter your data based on another query.

Lets say I have a complicated query of People.
```rust
// get Bob's addresses
let people_query = People::all()
    ...
    .where_col(|p| p.created_at.gt(started_date))
    .where_col(|p| p.name.ilike("%b%"));
```

It would be nice to be able to filter addresses on that list of people too. 
You can do this with where_relation

for example:
```rust
let addresses_query = Address::all().where_relation(|a| a.person, people_query);
```

Again this can be combined with other queries.
```rust
let addresses_query = Address::where_col(|a| a.active)
    .where_relation(|a| a.person, people_query);
```



