# Workshop: Reduce

Walking through the basics of the `reduce` method (also known as `foldr`) for collections, specifically in JavaScript.

## Introduction

Collections are one of the fundamental building blocks of software. Because of this, languages provide abstracted methods to simplify working with them.

A couple of the common methods are `map` for transforming each element of a collection, and `filter` for obtaining a limited selection of elements of the collection.

The `reduce` method can be thought of as the "Swiss army knife" of collections: any of the other collection methods can be implemented with it.

### Anatomy

There are 2 parts to reduce: the `accumulator` and the `reducer` function.

#### Accumulator

The accumulator is threaded through reduce by 3 aspects:
* Optionally starting with an initial value
* Collecting changes from each step
* The final return value

The initial value for the accumulator is optional. If a value is given, the first operation will be called on the first element with that value as the accumulator. If no initial value is given, the first operation will be called on the second element with the first element as the accumulator.

#### Reducer

The reducer function takes the current accumulator value and an element from the collection and returns the modified accumulator for the next step.

This function should be pure and without side-effects, and it should avoid mutating the passed in values.

## Simple Sum

One of the most straightforward ways to see reduce is in summing a list of numbers.

With an array of numbers 1 through 10:

```javascript
const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```

And a function that adds 2 numbers:

```javascript
const add = (a, b) => a + b
```

We can apply the `add` function to the array with the `reduce` method, "reducing" the collection to a single value.

```javascript
numbers.reduce(add)
// 55
```

Because our resulting value is the same type as the elements in the array, we can skip the initial value. If desired, we could provide an initial value and get the same result.

```javascript
numbers.reduce(add, 0)
// 55
```

However, if we have a starting amount, we could provide it as the initial value instead of adding the values afterwards.

```javascript
numbers.reduce(add, 10)
// 65
```

## Counting Letters

Let's use reduce to count the number of letters in a message.

Any string will do, but I think one relevant to what we are donig is appropriate:

```javascript
const string = 'This is a string of text to use to count the occurances of letters.'
```

By itself, we cannot iterate over a string, so we neet to `split` it. Since we only care about the letters, we can convert the string to lower case and remove any non-letter characters.

Ultimately, this leaves us with an array of all letter characters from the string:

```javascript
const letters = string
  .toLowerCase()
  .replace(/\W/g, '')
  .split('')
```

To count the letters, we can use an object with the letters as keys and counts as values. This will be our accumulator.

Our reducer function should accept the accumulated object of counts and current letter, and it should return a new object with updated values, keeping with the expected immutability of `reduce`.

```javascript
const countLetter = (counts, letter) => ({
  ...counts,
  [letter]: (counts[letter] || 0) + 1,
})
```

Because our reducer function is expected an object as the accumulator, we should provide an empty object as our initial value.

```javascript
letters.reduce(countLetter, {})
/*
{
  t: 10,
  h: 2,
  i: 3,
  s: 6,
  a: 2,
  r: 3,
  n: 3,
  g: 1,
  o: 6,
  f: 2,
  e: 6,
  x: 1,
  u: 3,
  c: 4,
  l: 1,
}
*/
```

## Implementing Other Array Methods

Because of the flexibility of `reduce`, the other array methods can be implemented with it.

### Map

The `map` method constructs a new array, applying an operation to each element in the array.

With `reduce`, each step can spread the current array along with the new element.

```javascript
const addDoubledToArray = (arr, n) => [...arr, n * 2]

numbers.reduce(addDoubledToArray, [])
// [ 2, 4, 6, 8, 10, 12, 14, 16, 18, 20 ]
```

### Filter

Similarily, `filter` returns a new array, but it only keeps the elements passing a condition.

This can be done at each step by either returning the current array or returning an array with the element added.

```javascript
const keepIfEven = (arr, n) => (n % 2 ? arr : [...arr, n])

numbers.reduce(keepIfEven, [])
// [ 2, 4, 6, 8, 10 ]
```

### Chainability

As with the built-in array methods, these reduce methods can be chained since they each return an array.

```javascript
numbers
  .reduce(keepIfEven, [])
  .reduce(addDoubledToArray, [])
// [ 4, 8, 12, 16, 20 ]
```

## Manipulating Arrays of Objects

Frequently, we encounter arrays of objects: api responses, option lists, etc. By making use of `reduce`, we can adapt these collections for when their original shapes are cumbersome to work with.

As an example, let's work with a set of menu options:

```javascript
const options = [
  { label: 'First', value: opt1 },
  { label: 'Second', value: opt2 },
  { label: 'Third', value: opt3 },
  { label: 'Fourth', value: opt4 },
  { label: 'Fifth', value: opt5 },
]
```

When an option is selected, we will typically have only the `value` of the option.

To get the full option, one option is to use the `find` array method; however, this is inefficient because each item must be checked individually until the desired one is found. A more performant approach is to convert the array into an object with each option stored by its `value` property.

```javascript
options.reduce(
  (acc, o) => ({ ...acc, [o.value]: o }),
  {},
)
/*
{
  opt1: { label: 'First', value: opt1 },
  opt2: { label: 'Second', value: opt2 },
  opt3: { label: 'Third', value: opt3 },
  opt4: { label: 'Fourth', value: opt4 },
  opt5: { label: 'Fifth', value: opt5 },
}
*/
```

If we needed to access each option by its `label` instead, it is as trivial as using it as the object key instead of the `value`.

```javascript
options.reduce(
  (acc, o) => ({ ...acc, [o.label]: o }),
  {},
)
/*
{
  First: { label: 'First', value: opt1 },
  Second: { label: 'Second', value: opt2 },
  Third: { label: 'Third', value: opt3 },
  Fourth: { label: 'Fourth', value: opt4 },
  Fifth: { label: 'Fifth', value: opt5 },
}
*/
```

## Processing Actions

So far, we have been working with sets of data. What if we instead want to handle actions that are performed?

As a simple example, let's build a calculator.

### Action Object

Let's define a structure to fit our actions.

Actions will have a `type` to differentiate them.

```javascript
{ type: 'RESET' }
```

Actions may have data associated with them. If we standardize on the key of `data`, we can handle them with a consistent pattern. Different sets of actions can have different types of values under `data`, but it will be a standardized but optional key.

```javascript
{ type: 'ADD', data: 1 }
```

We could add more fields if they are needed, but these will serve the majority of cases.

### Handling an Action

With the thought of building up a set of handlers for the actions, the most straightforward implementation is a switch statement.

First, we should set up a default case as a fallback. It should simply print a message to the console and return the value unchanged.

```javascript
const handleAction = (value, action) => {
  const { type, data } = action

  switch(type) {
    default:
      console.warn('Unhandled action:', action)
      return value
  }
}
```

Let's implement our first action: an addition.

```javascript
const handleAction = (value, action) => {
  const { type, data } = action

  switch(type) {
    case 'ADD':
      return value + data
    default:
      console.warn('Unhandled action:', action)
      return value
  }
}
```

### Processing a Set of Actions

Assuming that we have collected a series of actions, we can use `reduce` to process them all to get the resulting value.

```javascript
[
  { type: 'ADD', data: 1 },
  { type: 'ADD', data: 2 },
  { type: 'ADD', data: 3 },
].reduce(handleAction, 0)
// 6
```

Because we have accounted for actions that are not implemented, the processing does not break if an unexpected action is included.

```javascript
[
  { type: 'ADD', data: 1 },
  { type: 'ADD', data: 2 },
  { type: 'SUBTRACT', data: 1 },
  { type: 'ADD', data: 3 },
].reduce(handleAction, 0)
// Unhandled action: { type: 'SUBTRACT', data: 0 }
// 6
```

### Adding an Action Type

By simply adding another `case` statement, we can account for another action. Let's handle the `SUBTRACT` action.

```javascript
const handleAction = (value, action) => {
  const { type, data } = action

  switch(type) {
    case 'ADD':
      return value + data
    case 'SUBTRACT':
      return value - data
    default:
      console.warn('Unhandled action:', action)
      return value
  }
}
```

With that in place, we no longer get a warning, and calculations are performed as expected.

```javascript
[
  { type: 'ADD', data: 1 },
  { type: 'ADD', data: 2 },
  { type: 'SUBTRACT', data: 1 },
  { type: 'ADD', data: 3 },
].reduce(handleAction, 0)
// 5
```

### Making It Dynamic (Bonus)

Working with a static set of data is straightforward enough, but we can expand the action processing concept to be dynamic while making use of the same reducer function.

Instead of calling `reduce` on a set of actions, we need to build a mechanism to perform the same type of processing.

We need to handle 3 primary things:
* Take an initial value
* Accept ongoing actions
* Get the current value

Ultimately, this can boil down to a function that maintains state, and it returns an object with 2 properties: `takeAction` and `getValue`. We can provide `takeAction` with an action object to update the internal value, and `getValue` will return the value at the current time.

```javascript
const createCalculator = initialValue => {
  let currentValue = initialValue
  const getValue = () => currentValue

  const takeAction = action => {
    currentValue = handleAction(currentValue, action)
  }

  return { takeAction, getValue }
}
```

With this in place, we can implement our dynamic actions.

To start with, we create our stateful data object for our calculator:

```javascript
const calculator = createCalculator(0)
```

Now, we can give it actions from anywhere to change its current value. Let's make use of the same operations that we did before.

```javascript
calculator.takeAction({ type: 'ADD', data: 1 })
calculator.takeAction({ type: 'ADD', data: 2 })
calculator.takeAction({ type: 'SUBTRACT', data: 1 })
calculator.takeAction({ type: 'ADD', data: 3 })
```

At any point, we can call `getValue` to get the current value of the calculator.

```javascript
calculator.getValue()
// 5
```

This is one of the foundational concepts of [redux](https://github.com/reduxjs/redux): make immutable changes through actions while getting the value as needed.
