# Bulk

One truth across all databases is single database operations are slow.
Using operations that effect multiple rows simultaneously is a much faster way of interacting with the database.

This is unfortunate for ORMs since ORMs usually edit/create only one row at a time. `Welds` has a solution to this problem.

With welds you get [Bulk Insert](./bulk_insert.md), [Bulk Update](./bulk_update.md), and [Bulk Delete](./bulk_delete.md).

<br/>
<br/>

***NOTE: bulk operations do NOT trigger model hooks***






