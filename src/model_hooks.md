# Hooks

Welds allows you to attach functions (called "hooks") that run at specific points in the lifecycle of a model's interaction with the database. 
These hooks enable you to perform actions such as validation, logging, or side-effectful tasks (e.g., sending notifications) before or after a model is created, updated, or deleted.

***Warning: hooks do not run on bulk operations***

## Overview

Hook functions are regular Rust functions (or async functions) you register on your model using ```#[welds(...)]``` attributes. 
Each function has access to the current model instance so it can inspect or even mutate fields (for "before" hooks). 
In Welds, you can define hooks at these points:

- ***BeforeCreate***
  Runs just before inserting the model as a new row in the database.

- ***AfterCreate***
Runs immediately after the model is inserted in the database.

- ***BeforeUpdate***
Runs right before an existing model is updated in the database.

- ***AfterUpdate***
Runs right after an existing model is updated in the database.

- ***BeforeDelete***
Runs just before an existing model is deleted from the database.

- ***AfterDelete***
Runs immediately after an existing model is deleted from the database.

<br/>

Because these hooks can alter or inspect your model right before or after it touches the database, they are a powerful way to inject your application-specific logic. Typical uses include:
- ***Validation***: Validate fields before insertion or update.
- ***Audit Logging***: Output or record a log of changes.
- ***Timestamps***: Automatically update timestamps or record user info.
- ***Notifications***: Email or push notifications in the "after" hooks.
- ***Transformation***: Adjust or sanitize data before persisting.

## Synchronous vs. Asynchronous Hooks
By default, Welds treats hook functions as synchronous. You can, however, specify async = true in the attribute if your function is async:

```rust
#[welds(AfterCreate(my_async_callback, async = true))]
```

This allows you to perform asynchronous tasks, such as making web requests, sending notifications, etc., without blocking the thread. 

***Warning: If you use an async hook within a transaction, you will be locking the table for the life of that async call. This should be avoided***


## Hook Function Signatures

Each hook receives different parameters depending on whether it’s a "before" or "after" hook:

<br/>

Before Hooks accept a mutable reference to the model, allowing you to change the model before writing to the database:

```rust
fn before_create(model: &mut MyModel) -> welds::errors::Result<()> {
    // Possibly mutate fields or validate
    Ok(())
}
```

Returning errors
If a "before" hook fails and returns an error, Welds will stop the associated operation (create, update, or delete) and return the error to the caller. 

<br/>

After Hooks accept an immutable reference to the model, as the change has already happened in the database:

```rust
fn after_create(model: &MyModel) {
    // Log or send a notification
}
```

