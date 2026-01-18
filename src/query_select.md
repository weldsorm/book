# Custom Columns, Joins, and Aggregation

So far in the book, our models have been One-to-One matches with the underlying tables.
But what do we do if we want a subset of columns, or maybe data across tables.

Introducing `select`, `select_as`, `group_by` and `select_count`

These functions allow you to write queries that are disconnected from the concept of the underlying table.

This means they can:
- span multiple tables via joins
- rename columns
- pick only the parts of the data you care about.
- introduce new columns for counting the matches

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

To get just the Player name
we can start querying off of our existing Player Model that is wired up to the table.
Once we have a basic query we can select out only the columns we care about.

A query with a call to `select` returns generic rows, so we must call `collect_into()`
to collect them into our `PlayerInfo` struct.

An example that returns the names of the "good" players might look like:

```rust
let player_query = Player::where_col(|p| p.goals.gt(3) );
let info_query = player_query.select(|p| p.name );
let infos: Vec<PlayerInfo> = info_query.run(&client).await?.collect_into()?;
```

If we want to know more information about the player,
we can add it with `select` (but remember to update `PlayerInfo`!):

```rust

#[derive(WeldsModel)]
struct PlayerInfo {
    pub name: String,
    pub jersey_number: u8,
}
let player_query = Player::where_col(|p| p.goals.gt(3) );
let info_query = player_query
  .select(|p| p.name )
  .select_as(|p| p.number, "jersey_number");

let infos: Vec<PlayerInfo> = info_query.run(&client).await?.collect_into()?;
```

Note: in this example Player has a column `number`, but in our output model we want to call it `jersey_number`. 
We can support this by renaming the column as it is being selected from the database using `select_as`.

## With more data (Joining table)

What about if we want a list of player names, and their team names.
Lets first start by updating our output struct.

```rust
#[derive(WeldsModel)]
struct PlayerInfo {
    pub player_name: String,
    pub team_name: String,
    pub number: u8
}
```

This time around both models have a column called name,
so we will need to rename them with `select_as`.

We can then start querying like before,
but this time we will Join using our relationships we defined on the models `Player` and `Team`.


```rust
let player_query = Player::where_col(|p| p.goals.gt(3) );

let info_query = order_query
    .select_as(|x| x.name, "player_name")
    .select(|x| x.number)
    .join( |p| p.team, Team::select_as(|p| p.name, "team_name") );

let infos: Vec<PlayerInfo> = info_query.run(&client).await?.collect_into()?;
```

## Group By

Extending on the query above for getting all players and their teams,
let's say you want to get the *good* players on a team (i.e. those with high number of goals).

You can do this with `select_count` and `group_by`:

```rust

#[derive(WeldsModel)]
pub struct TeamsWithPlayerCount {
    pub name: String,
    pub good_player_count: i32,
}

let teams_and_good_player_count: Vec<TeamsWithPlayerCount> = Team::all()
    .select(|t| t.name)
    .left_join(
        |t| t.players,
        Player::where_col(|p| p.goals_scored.gt(3))
            .select_count(|p| p.id, "good_player_count")
            .group_by(|p| p.team_id),
    )
    .run(&client)
    .await?
    .collect_into()?;
```

## Distinct

And if you wanted to see a list of all player names
(assuming some player plays on multiple teams and thus has multiple entries):

```rust

#[derive(WeldsModel)]
pub struct PlayerName {
    pub name: String
}

let distinct_player_names = Player::select(|p| p.name)
    .distinct()
    .run(&client)
    .await?;
```
