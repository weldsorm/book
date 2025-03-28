# Fundamentals

Before diving to deep into Welds, there are a couple constructs you should be aware of. 
These are the four most common things you will see when interacting with Welds.

Infact, This is what is included when you add the line.
```rust
use welds::prelude::*;
```
which you should do.

## welds::Client

This is the trait that is implemented by All the backend pools and all the backend transaction types.

It is common to see this trait passed into your function that does database calls.    

With this trait you can execute SQL and get data back, however you typically don't call it directly.

## welds::TransactStart

This trait is implemented by All the backend pools.

It is used to start a transaction. It is almost always the same object as the Client.

You can use it to start a transaction by calling `begin()`

NOTE: transactions are automatically rolled back unless you call `commit()`


## welds::DbState\<T\>

This is a wrapper around your model. While you might not use this struct directly, it is what is returned to you from a database query. It works very similar to `std::sync::MutexGuard`, transparently giving you access to the model inside.
It is used by welds to keep track of the changes you make to your model so Welds can create or update it in the database.

If you are finished with Welds and just want your vanilla model back, you can call `into_inner` or `into_vm` to get to the inner object.

## WeldsModel

The Attribute you will add to your models. [More information can be found here](models.md)

