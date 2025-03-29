# Query

In Welds a query or `QueryBuilder<T>` is an un-executed action to be preformed in the database. It is the root; the starting point for interactions with the database.

Bulk operations and single column selects are both accessible from a Query, so it really is the first place you should turn.

QueryBuilders can be accessed off of your models directly using functions such as  `where_col(...)` or `all()`
