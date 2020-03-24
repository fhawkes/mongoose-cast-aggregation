# Mongoose Cast Aggregation

A mongoose plugin that casts aggregation pipelines whenever possible.

## Getting Started


Run:
```
npm install mongoose-cast-aggregation
```

Inject the plugin into mongoose:
```js
const mongoose = require('mongoose');
const { Schema } = mongoose;
const castAggregation = require('mongoose-cast-aggregation');

mongoose.plugin(castAggregation); 
```


Now mongoose will cast the `$match` stage whenever possible. It casts the `$match` stage as long as no stage before it changed the resulting document shape from the original schema (e.g. `$match`, `$limit`, `$sort`, `$skip`, and `$sample`).
```js
// After injecting the plugin
const discountSchema = new Schema({
  expiresAt: Date,
  amount: Number
});

const Discount = mongoose.model('Discount', discountSchema);

const discounts = await Discount.aggregate([
  // Will cast the amount to a number, and the timestamp to a date object
  { $match: { expiresAt: { $lt: Date.now() }, amount: '20' } }
]);
```

This works as well:

```js
const discounts = await Discount.aggregate([
  { $sort: { amount:-1 } },
  { $skip: 20 },
  // Will cast the stage below to a date object, because the document shape hasn't changed yet.
  { $match: { expiresAt: { $lt: Date.now() } } },

  // Will cast this one to numbers as well.
  { $match: { amount: { $gt: '80', $lt: '200' } } },
  { $project: { amountInUSD: '$amount' } },

  // Will ***NOT*** cast this one, because we used a stage that changed the shape of the document.
  // so using the string '100' here will not work, will have to use the correct type of number in order to get results.
  { $match: { amountInUSD: { $gt: 100 } } }
]);
```