# Manual SQL


Custom SQL in WHERE Clauses
Sometimes you need more flexibility than the standard .where_col method provides. 
For instance, you might want to perform a comparison between columns or use a database function in a way the query builder doesn’t directly support. 
Welds offers two helper methods `where_manual` and `where_manual2` that let you drop in custom SQL snippets in your WHERE clauses.

## Introduction

Welds typically handles SQL generation for you. However, in advanced scenarios, you might need direct control over part of the WHERE clause. 

The two functions covered here:
- where_manual
- where_manual2

Both let you inject custom SQL strings. They differ slightly in how they reference columns and in how the snippet is anchored in the `WHERE` condition.

Important Notes:

1) ***Use `?` for parameters.*** Welds will substitute them with the correct syntax (`$1`, `?`, or `@p1` depending on your database).
2) ***Use `$` as a placeholder for the table alias.*** At runtime, Welds replaces `$` with the actual table alias (e.g., `t1`).
        For instance, `$.price1` might become `t1.price1`.

where_manual
Description

## where_manual

where_manual adds custom SQL to the right side of a clause while anchoring it to a specific column in your model.
Internally, it references the column you choose (in the lambda) and appends a user-supplied custom snippet.

Usage:

Imagine you have a table thing with two integer fields price1 and price2. 
You want to return rows where price1 is greater than price2 + 5. 
We can’t express this directly with standard comparisons, so we use where_manual:

```rust
let query = Thing::all().where_manual(|c| c.price1, " > ($.price2 + ?) * ?", (1, 0.19) );
```

The params for where_manual are automatically converted from a tuple of whatever size.



## where_manual2

where_manual2 gives you full control over a custom SQL snippet in the WHERE clause without requiring an anchor column.
This is useful for comparing multiple columns or more elaborate function calls.

```rust
let query = Thing::all().where_manual2("$.price1 + $.price2 > ?", (33,) );
```

The params for where_manual2 are automatically converted from a tuple of whatever size.


## Other

There are a couple other `*_manual` functions in the QueryBuilder with the purpose of allowing small manual SQL snippets in the query.
