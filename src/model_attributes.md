# Attributes

This is a list of Attributes you can add to your welds model to help control how it is wired up.

## Struct Level Attributes



### Table Links
```rust
#[welds(table = "tablename")]
// or 
#[welds(schema="schemaname", table = "tablename")]
```
Link a model to a table.  
If a model is a one-to-one, or a subset of a table, add this on to it so that You can query directly off of the model.

### Readonly
```rust
#[welds(readonly)]
```
Mark a model as being readonly. Useful when wiring up views or other read only objects. This prevents the save code from being generated in the derive


### Relationships
```rust
#[welds(BelongsTo(team, Team, "team_id"))]
#[welds(HasMany(players, Player, "team_id"))]
#[welds(HasOne(profile, Profile, "profile_id"))]
#[welds(BelongsToOne(user, User, "profile_id"))]
```
See the [Relations section](./model_relationships.md) for more details


### Hooks
```rust
#[welds(BeforeCreate(func))]
#[welds(AfterCreate(func))]
#[welds(BeforeUpdate(func))]
#[welds(AfterUpdate(func))]
#[welds(BeforeDelete(func))]
#[welds(AfterDelete(func))]
```
Hooks. See the [Hooks section](./model_hooks.md) for more details



## Field Level Attributes

### Primary key
```
#[welds(primary_key)]
```
Lets welds know which field is the primary_key of the table/view.

This is needed for:
- find_by_id
- Creating / Updating / Deleting
- Relationships

### Rename
```
#[welds(rename="new_name")]
```
Add This to a field when the field's name doesn't match the name in the database

### Ignore
```
#[welds(ignore)]
```
Sometimes you need to have a little extra state or other stuff attached to your model.
This tells welds that this field has nothing to do with the database and should be ignored.

If you can, It is useful to make this field `Default`. That way welds can really ignore it all the way.

