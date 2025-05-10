# Include

*NOTE: when using include your data is read only*

<br />

Fetch Models and their related objects.
Models are wired up to their related objects for ease of access.

Sometimes it's not the data that's important its the relationship.
A good example of this is a `select` with `opgroup`.

In the following Html snippit, we want both the `Food` model and the `FoodCategory` model.
We need to iterate over `FoodCategory` and find the relevant `Food` model.

In this example If we load our data with `include`, welds will automatically wire this up.

<br />
<br />
<label for="hr-select">Your favorite food</label>

<select name="foods" id="hr-select">
  <option value="">Choose a food</option>
  <hr />
  <optgroup label="Fruit">
    <option value="apple">Apples</option>
    <option value="banana">Bananas</option>
    <option value="cherry">Cherries</option>
    <option value="damson">Damsons</option>
  </optgroup>
  <hr />
  <optgroup label="Vegetables">
    <option value="artichoke">Artichokes</option>
    <option value="broccoli">Broccoli</option>
    <option value="cabbage">Cabbages</option>
  </optgroup>
  <hr />
  <optgroup label="Meat">
    <option value="beef">Beef</option>
    <option value="chicken">Chicken</option>
    <option value="pork">Pork</option>
  </optgroup>
  <hr />
  <optgroup label="Fish">
    <option value="cod">Cod</option>
    <option value="haddock">Haddock</option>
    <option value="salmon">Salmon</option>
    <option value="turbot">Turbot</option>
  </optgroup>
</select>



```html
<label for="hr-select">Your favorite food</label> <br />

<select name="foods" id="hr-select">
  <option value="">Choose a food</option>
  <hr />
  <optgroup label="Fruit">
    <option value="apple">Apples</option>
    <option value="banana">Bananas</option>
    <option value="cherry">Cherries</option>
    <option value="damson">Damsons</option>
  </optgroup>
  <hr />
  <optgroup label="Vegetables">
    <option value="artichoke">Artichokes</option>
    <option value="broccoli">Broccoli</option>
    <option value="cabbage">Cabbages</option>
  </optgroup>
  <hr />
  <optgroup label="Meat">
    <option value="beef">Beef</option>
    <option value="chicken">Chicken</option>
    <option value="pork">Pork</option>
  </optgroup>
  <hr />
  <optgroup label="Fish">
    <option value="cod">Cod</option>
    <option value="haddock">Haddock</option>
    <option value="salmon">Salmon</option>
    <option value="turbot">Turbot</option>
  </optgroup>
</select>
```

## Loading the data

Let us assume we have the two models `Food` and `FoodCategory`. 
While you could select either model with the include and make it work,
we are iterating over `FoodCategory` first. Because of this, it is a better model to start with. 

```rust
    use welds::dataset::DataSet;

    let set: DataSet<FoodCategory> = FoodCategory::all()
        .include(|c| c.food)
        .run(client.as_ref())
        .await?;
```

This leads into the next question.
What is a `DataSet` and how do I use it?


## DataSet\<T\>

Dataset is an iterate-able data collection like `Vec`.
It exists to maintain and access the related data between the models.

When you access a model from it, you get your model with a little extra. 
The accessed data is wrapped in a special accessor struct that knows how to lookup its related objects.


```rust
    use welds::dataset::DataSet;

    let set: DataSet<FoodCategory> = FoodCategory::all()
        .include(|c| c.food)
        .run(client.as_ref())
        .await?;

    for category in set.iter() {
        println!("category: {}", category.name);

        // access the food for this spisiffic category
        let food_in_category = category.get(|x| x.food);
        for food in &food_in_category {
            println!("food: {}", food.name);
        }
    }


```


## Filter sub models

If you are interested in fetching a subset of the included models, this can be achieved using `include_where`.
`include_where` works just like include but an included filter is applied to the child model.

```rust
    use welds::dataset::DataSet;

    let set: DataSet<FoodCategory> = FoodCategory::all()
        .include_where(|c| t.food, Food::where_col(|f| f.deleted.equal(false)))
        .run(client.as_ref())
        .await?;
```
