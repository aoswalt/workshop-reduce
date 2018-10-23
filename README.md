# Workshop: Reduce

## Introduction

```javascript
const nums = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```

```javascript
const add = (a, b) => a + b
```

## simple sum

```javascript
nums.reduce(add)
// 55
```

with accumulator

```javascript
nums.reduce(add, 0)
// 55
```

with accumulator of initial value

```javascript
nums.reduce(add, 10)
// 65
```

## more complex sum

string

```javascript
const string = 'This is a string of text to use to count the occurances of letters.'
```

split into letter array

```javascript
const letters = string
  .toLowerCase()
  .replace(/\W/g, '')
  .split('')
```

function to increment count for a letter

```javascript
const countLetter = (counts, letter) => ({
  ...counts,
  [letter]: (counts[letter] || 0) + 1,
})
```

intial accumulator is important without making fn more complex

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

## other array methods

map

```javascript
const addToArray = (arr, v) => [...arr, v]
const addDoubledToArray = (arr, n) => addToArray(arr, n * 2)

nums.reduce(addDoubledToArray, [])
// [ 2, 4, 6, 8, 10, 12, 14, 16, 18, 20 ]
```

filter

```javascript
const keepIfEven = (arr, n) => (n % 2 ? arr : [...arr, n])

nums.reduce(keepIfEven, [])
// [ 2, 4, 6, 8, 10 ]
```

chainable because returning another array

```javascript
nums
  .reduce(keepIfEven, [])
  .reduce(addDoubledToArray, [])
// [ 4, 8, 12, 16, 20 ]
```

## more complex arrays

a set of menu options

```javascript
const options = [
  { label: 'First', value: 1 },
  { label: 'Second', value: 2 },
  { label: 'Third', value: 3 },
  { label: 'Fourth', value: 4 },
  { label: 'Fifth', value: 5 },
]
```

getting an array only the values

```javascript
options.reduce((arr, { value }) => [...arr, value], [])
// [ 1, 2, 3, 4, 5 ]
```

the same but with the labels instead

```javascript
options.reduce((arr, { label }) => [...arr, label], [])
// [ 'First', 'Second', 'Third', 'Fourth', 'Fifth' ]
```

changing to object for O(1) lookup
turning the array into an object with the label property as keys

```javascript
options.reduce(
  (acc, o) => ({ ...acc, [o.label]: o }),
  {},
)
/*
{
  First: { label: 'First', value: 1 },
  Second: { label: 'Second', value: 2 },
  Third: { label: 'Third', value: 3 },
  Fourth: { label: 'Fourth', value: 4 },
  Fifth: { label: 'Fifth', value: 5 },
}
*/
```

turning the array into an object with the value property as keys

```javascript
options.reduce(
  (acc, o) => ({ ...acc, [o.value]: o }),
  {},
)
/*
{
  '1': { label: 'First', value: 1 },
  '2': { label: 'Second', value: 2 },
  '3': { label: 'Third', value: 3 },
  '4': { label: 'Fourth', value: 4 },
  '5': { label: 'Fifth', value: 5 },
}
*/
```
