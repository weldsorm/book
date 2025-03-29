# Create, Update, And Delete

When you get data back from the database it is typically wrapped in DbState like this.

```
let mut pet: DbState<Pet> = Pet::find_by_id(42).run(client.as_ref()).await?;
```

DbState is how welds knows what to do with your models.

The compiler knows how to automatically treat a DbState as a borrowed version of its inner type. 
This means for the most part you shouldn't need to refer to DbState much, just the borrowed type.

For example:

```rust
fn give_dog_treat(&mut dog: Pet) {
    dog.wags += 1;
}
```

```rust
let mut pet: DbState<Pet> = Pet::find_by_id(42).run(client.as_ref()).await?;
give_dog_treat(&pet);
```

<br/>
<br/>

***DbState is required to Create, Update, or Delete the instance of your model in the database.***

Saving can always be done by calling. `save()`.
Save will create or update based on the states knowledge of what is in the database.
Additionally `delete` can be called to execute a SQL delete command.

<br/>
<br/>
<br/>
<br/>

## Removing DbState

A very common use case is to fetch some data and then display, or do something with it that doesn't involve the database. 
When this happens it is often nice to not have to worry about DbState. 
Welds provides several ease of use functions to help remove DbState when you want just the Underlying Model.

<br/>

### Single Instances

#### into_inner()
returns the inner Model stripped of DbState.

Usage:
```rust
let pet_with_state: DbState<Pet> = Pet::find_by_id(42).run(client.as_ref()).await?;
let pet: Pet = pet_with_state.into_inner();
```
Typical Use Case:
- Interacting with other code non-welds related.


<br/>

#### into_vm()
returns the inner Model stripped of DbState and wrapped in Arc.

Usage:
```rust
let pet_with_state: DbState<Pet> = Pet::find_by_id(42).run(client.as_ref()).await?;
let pet: Arc<Pet> = pet_with_state.into_vm();
```
Typical Use Case:
- Interacting with other code non-welds related.
- Interacting with view type layers of code that want to share the model. (Yew)

<br/>
<br/>
<br/>

### Vec of DbState
These methods are *Method Extensions* added to `Vec<DbState<T>>`
when you include 
```rust
use welds::prelude::*;
```

#### into_inners()
returns a Vec of Models stripped of DbState.

Usage:
```rust
let pets: Vec<DbState<Pet>> = Pet::all().run(client.as_ref()).await?;
let pet: Vec<Pet> = pets.into_inners();
```
Typical Use Case:
- Interacting with other code non-welds related.

<br/>

#### into_vms()
returns a Vec of Models stripped of DbState and wrapped in Arc.

Usage:
```rust
let pets: Vec<DbState<Pet>> = Pet::all().run(client.as_ref()).await?;
let pet: Vec<Arc<Pet>> = pets.into_vms();
```
Typical Use Case:
- Interacting with other code non-welds related.
- Interacting with view type layers of code that want to share the model. (Yew)
