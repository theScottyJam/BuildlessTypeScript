# Buildless TypeScript Formatting Guide

This formatting guide is for you if:
* experimenting with different ways to format comment delimiters in order to come up with a consistent coding style doesn't sound fun to you, and you'd rather just run with someone else's opinions.
* you'd like multiple maintainers to follow similar formatting rules, and you don't care much what those rules are, just as long as everyone is consistent with them.

It should go without saying, but you're not required to use this guide. Maybe you don't like the rules and prefer coding in a different style, or maybe it goes against your linter's configuration, or maybe you just don't like being told what to do - whatever the case may be, feel free to just ignore some or all of this guide.

Nothing in this guide is set in stone. It may be amended over time, but most changes will be minor.

## Terminology

"inline TS comment" - a TS comment that's part of a larger JavaScript statement.

```javascript
const num = value /*:: as number */;
```

"single-line TS comment" - A TS comment that contains an entire line of code.

```javascript
/*:: Type JsonPrimitive = string | number | boolean | null; */
```

"multi-line TS comment" - A TS comment that contains multiple lines of code.

```javascript
/*::
interface Coordinate {
  readonly x: number
  readonly y: number
}
```

```javascript
/*::
Type NonNullJsonPrimitive = string | number | boolean;
type JsonPrimitive = NonNullJsonPrimitive | null;
*/
```

## General Formatting Tips

Always attach the colons to the comment delimiter.

```javascript
// ✓ - Correct
const num = value /*:: as number */;
let str: /*: string */;

// ✕ - Incorrect
const num = value /* :: as number */;
let str: /* : string */;
```

If you have a function with two or more parameters, it is recommended to put each parameter on a line of its own.

```javascript
// ✓ - Correct
function addTwoNumbers(
  num1 /*: number */,
  num2 /*: number */,
) {
  return num1 + num2;
}

// ✓ - Slightly discouraged, but it's also correct
function addTwoNumbers(num1 /*: number */, num2 /*: number */) {
  return num1 + num2;
}
```

## Inline TS comments

TS comments that are part of a larger JavaScript statement should be padded on the inside with a space.

```javascript
// ✓ - Correct
const num = value /*:: as number */;
let str: /*: string */;
obj/*:: ! */.prop;

// ✕ - Incorrect
const num = value /*::as number*/;
let str: /*:string*/;
obj/*::!*/.prop;
```

The only time you can break this rule is if your TS comment starts with a comma - in this case it is ok for the comma to hug the comment delimiter.

```javascript
// ✓ - Correct
import { createCoordinate /*::, type Coordinate */ } from './coordinate.js';

// ✕ - Incorrect
import { createCoordinate /*:: , type Coordinate */ } from './coordinate.js';

// ✓ - Also correct (you can instead reorder the imports if you prefer)
import { /*:: type Coordinate, */ createCoordinate } from './coordinate.js';
```

The outside of a TS comment should only be padded with a space if you would write the same code in TypeScript with a space there. For example, if you typically write generic classes in TypeScript like this:

```typescript
class Container<T> {
  ...
}
```

Then when converting the `<T>` to buildless TypeScript, you should only pad the right side with a space since that's how it was padded in the original code. Notice how in both code examples, the word `Container` does not have a space afterward, and in both examples there is a space before the opening bracket `{`.

```javascript
class Container/*:: <T> */ {
  ...
}
```

Type annotations are the exception to this rule - whenever a type annotation is used, a space should be added to the left side of the opening comment.

```javascript
// ✓ - Correct
// If you're writing these examples in TypeScript, you'd typically
// make the colon hug the preceding variable name.
// When converting to Buildless TypeScript, you'd
// always add an extra space between the two.
let numb /*: number */;
let bool /*:: !: boolean */;
function fn(str /*:: ?: string */) {}

// ✕ - Incorrect
let numb/*: number */;
let bool/*:: !: boolean */;
function fn(str/*:: ?: string */) {}
```

## Single-line/Multi-line TS comments

For multi-line TS comments, the opening and closing delimiters should be on their own lines.

```javascript
// ✓ - Correct
/*::
interface Coordinate {
  readonly x: number
  readonly y: number
}
*/

// ✕ - Incorrect
/*:: interface Coordinate {
  readonly x: number
  readonly y: number
} */
```

For TS comments with only one line of content, you are encouraged to put the delimiters on the same line as the content (but this isn't required). TS comments that are on the same line as their content should be padded on the inside with a space.

```javascript
// ✓ - Correct
/*:: type JsonPrimitive = string | number | boolean | null; */

// ✓ - Slightly discouraged, but also correct
/*::
type JsonPrimitive = string | number | boolean | null;
*/

// ✓ - Incorrect
// The inside is not padded with a space
/*::type JsonPrimitive = string | number | boolean | null;*/
```

## Combining TS comments

If two single-line/multi-line TS comments are next to each other, you are encouraged to combine them into a single TS comment (but this isn't required).

```javascript
// ✓ - Correct

/*::
Type NonNullJsonPrimitive = string | number | boolean;
type JsonPrimitive = NonNullJsonPrimitive | null;
*/

// ✓ - Correct
/*::
function add(x: string, y: string): string
function add(x: number, y: number): number
*/
function add(
  x /*: number | string */,
  y /*: number | string */,
) /*: number | string */ {
  return (x /*:: as any */) + (y /*:: as any */);
}

// ✓ - Slightly discouraged, but it's also correct

/*:: Type NonNullJsonPrimitive = string | number | boolean; */
/*:: type JsonPrimitive = NonNullJsonPrimitive | null; */

/*:: function add(x: string, y: string): string */
/*:: function add(x: number, y: number): number */
function add(
  x /*: number | string */,
  y /*: number | string */,
) /*: number | string */ {
  return (x /*:: as any */) + (y /*:: as any */);
}
```

Don't do weird combinations of TS comments. Don't combine a single-line/multi-line TS comment with an inline one.

```javascript
// ✓ - Correct
const num = value /*:: as number */;
/*:: Type JsonPrimitive = string | number | boolean | null; */

/*:: type

// ✕ - Incorrect
const num = value /*:: as number
Type JsonPrimitive = string | number | boolean | null; */
```

Some TypeScript code is strongly related to JavaScript code next to it, such as function overloads next to the function definition, or type imports next to normal imports. Don't combine the TS comments containing this TypeScript code with other unrelated TypeScript code.

```javascript
// ✓ - Correct

/*:: Type JsonPrimitive = string | number | boolean | null; */

/*::
function add(x: string, y: string): string
function add(x: number, y: number): number
*/
function add(
  x /*: number | string */,
  y /*: number | string */,
) /*: number | string */ {
  return (x /*:: as any */) + (y /*:: as any */);
}

// ✕ - Incorrect

/*::
Type JsonPrimitive = string | number | boolean | null;

function add(x: string, y: string): string
function add(x: number, y: number): number
*/
function add(
  x /*: number | string */,
  y /*: number | string */,
) /*: number | string */ {
  return (x /*:: as any */) + (y /*:: as any */);
}

// ✓ - Correct

import fs from 'node:fs';
/*:: import type Coordinate from './coordinate.js'; */

/*::
interface CoordinateIn3d extends Coordinate {
  readonly z: number
}
*/

// ✕ - Incorrect

import fs from 'node:fs';
/*::
import type Coordinate from './coordinate.js';

interface CoordinateIn3d extends Coordinate {
  readonly z: number
}
*/
```
