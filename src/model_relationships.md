# Model Relationships

Welds supports several different types of relationships between models. By specifying these relationships, you can let the ORM know how your models are connected in the database.
This is used when making advanced queries across tables and across table updates

## Overview

Relationships are defined as attributes on your model’s struct. Each relationship attribute includes:
1. A **name** (the field or logical name of the relationship).
2. A **target type** (the related model).
3. A **foreign key** or **reference field** (the column in this model or the related model that links them).

For example:

```rust
#[welds(BelongsTo(team, Team, "team_id"))]
```

<br/>

### 1. BelongsTo

```rust
#[welds(BelongsTo(team, Team, "team_id"))]
```

Use BelongsTo when your model holds the foreign key referencing another model.

- **Name** (first parameter): The name you want to give this relationship in your model (e.g., team).
- **Target model** (second parameter): The Rust struct you’re linking to (e.g., Team).
- **Foreign key** (third parameter): The column in this model that references the parent’s primary key (e.g., "team_id").

Example use case:
```rust
#[derive(welds::WeldsModel)]
#[welds(table = "players")]
#[welds(BelongsTo(team, Team, "team_id"))]
pub struct Player {
    #[welds(primary_key)]
    pub id: i32,
    pub name: String,
    pub team_id: i32,
}
```
    The Player model has a team_id field referencing a Team record’s primary key.

<br/>

### 2. HasMany

```rust
#[welds(HasMany(players, Player, "team_id"))]
```

Use HasMany when the other model holds the foreign key. 

- **Name** (first parameter): The name of the relationship collection (e.g., players).
- **Target model** (second parameter): The child model (e.g., Player).
- **Foreign key** (third parameter): The column on the child model that references this model’s primary key (e.g., "team_id").

Example use case:

```rust
#[derive(welds::WeldsModel)]
#[welds(table = "teams")]
#[welds(HasMany(players, Player, "team_id"))]
pub struct Team {
    #[welds(primary_key)]
    pub id: i32,
    pub name: String,
}
```
    The Team model can have many Player records, each storing team_id.

<br/>

### 3. HasOne

```rust
#[welds(HasOne(image, Image, "user_id"))]
```

Use HasOne when the association is singular and the other model holds the foreign key. 

- **Name** (first parameter): The name of the relationship (e.g., image).
- **Target model** (second parameter): The child model (e.g., Image).
- **Foreign key** (third parameter): The column on the child model that references this model’s primary key (e.g., "user_id").

Example use case where a User has one image:

```rust
#[derive(welds::WeldsModel)]
#[welds(table = "users")]
#[welds(HasOne(image, Image, "user_id"))]
pub struct User {
    #[welds(primary_key)]
    pub id: i32,
    pub name: String,
}
```

```rust
#[derive(welds::WeldsModel)]
#[welds(table = "images")]
#[welds(BelongsTo(user, User, "user_id"))]
pub struct Image {
    #[welds(primary_key)]
    pub id: i32,
    pub user_id: i32,
    pub url: String,
}
```

    A User has one Image, the Image BelongsTo the User and holds the foreign key.

<br/>


