# Aggregate Functions (Group By)

Aggregate Functions are a continuation of [Custom Select](./query_select.md). It is recommended to have a understanding of custom selects before continuing.

## Introduction

When working with custom selects, it is possible to use welds to create queries that result in `group by` statements.
These are intended to be used with custom output structs just like custom selects.

Normal group by requirements apply with welds group bys.
If you are using a group by, all columns must be either aggregated or in the group by.



## Basic example

Lets start with a simple example. Counting.

```rust
// Struct for collecting group by output
#[derive(Debug, WeldsModel)]
pub struct PeopleCount {
    pub name: String,
    pub count: i32,
}
```

To count the people by their names, typically you would select the names grouping by the name column.
Something like this in SQL
```sql
SELECT count(id), name FROM people GROUP BY name
```

This can be written in welds in a similar way

```rust
    let query = Player::all()
        .select(|p| p.name)
        .select_count(|p| p.id, "count")
        .group_by(|p| p.name);
    let data: Vec<PeopleCount> = query.run(&client).await?.collect_into()?;
```


All the aggregate functions in welds follow the pattern `select_*( lambda, output_name)`

You are required to include a name for your output data column because welds populates your struct by column name not index.



