# Editing and Creating

When you get data back from the database it is typically wrapped in DbState like this.

```
let pet: DbState<Pet> = Pet::find_by_id(32).run(client.as_ref()).await?;
```

DbState is how welds knows what to do with your models. 
