# Custom Select

So far in the book, our models have been One-to-One matches with the underlying tables.
But what do we do if we want a subset of columns, or maybe data across tables.

Introducing `select` and `select_as`

`select` and `select_as` allow for the writing of queries that are disconnected from the concept of the underlying table.

This means:
- They can span multiple tables via joins
- They can rename columns
- They can pick only the parts of the data you care about.

## Introduction

A *Custom Select* involves two basic steps.
1) Define a struct that represents the data you want out of the database.
2) Write a Query selecting parts from your existing models to extract that data.

Lets assume we want this data. 

```rust
#[derive(WeldsModel)]
struct PlayerInfo {
    pub name: String,
}
```
Note: This model is not connected to a table and there isn't a table that looks like this.


## Start Querying (single column)

To get just the Player name we can start querying off of our existing Player Model that is wired up to the table. 
Once we have a basic query we can select out only the columns we care about.

A query with a call to `select` returns generic rows, so we must call `collect_into()`
to collect them into our `PlayerInfo` struct.

```rust
let player_query = Player::where_col(|p| p.goals.gt(3) );
let info_query = player_query.select(|p| p.name );
let infos: Vec<PlayerInfo> = info_query.run(&client).await?.collect_into()?;
```

## With more data (Joining table)

What about if we want a list of player names, and their team names.
Lets first start by updating our output struct.

```rust
#[derive(WeldsModel)]
struct PlayerInfo {
    pub player_name: String,
    pub team_name: String,
}
```

This time around both models have a column called name. so we will need to rename them with `select_as`.
We can then start querying like before, but this time we will Join using our relationships we defined on the models `Player` and `Team`.


```rust
let player_query = Player::where_col(|p| p.goals.gt(3) );

let info_query = order_query
    .select_as(|x| x.name, "player_name")
    .join( |p| p.team, Team::select_as(|p| p.name, "team_name") );

let infos: Vec<PlayerInfo> = info_query.run(&client).await?.collect_into()?;
```


