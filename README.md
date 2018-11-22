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
