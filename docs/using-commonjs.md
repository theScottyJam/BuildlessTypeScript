# So you want to use CommonJS with Buildless TypeScript?

The problem is that a typical TypeScript CommonJS project actually uses ES module syntax, and the TypeScript compiler will transpile the ES module syntax into CommonJS. Since there's no transpilation step when using Buildless TypeScript, this typical method of supporting CommonJS won't work. There are work-arounds available, but they're pretty hairy.

After a little bit of tinkering, I was able to come up with the following example that shows how you can export values and types from one module and import them in another. It's not pretty, but it works.

```javascript
// ~~ coord.js ~~
// This defines a createCoordinate() function and a
// Coordinate interface that we'd like to export.

/*::
interface Coordinate {
  readonly x: number
  readonly y: number
}
*/

function createCoordinate(
  x /*: number */,
  y /*: number */,
) {
  return { x, y };
}

// All values that we wish to export
// go in this variable.
const valueExports = {
  createCoordinate
};

// All types that we wish to export
// go in this interface.
// We need to make sure we export the type
// of our valueExports variable as well.
/*::
interface TypeExports {
  contents: typeof valueExports
  Coordinate: Coordinate
}
*/

// Export the values
module.exports = valueExports;

// Export the types
/*:: export = TypeExports; */

// ~~ main.js ~~
// This module will import the createCoordinate() function
// and Coordinate interface from coord.js

/*::
// Import the types and pluck off some types that we care about.
type CoordModule = import('./coord');
type Coordinate = CoordModule['Coordinate'];
*/
// Import the values and set the type of the value, so
// that its not inferred as the "any" type.
const { createCoordinate } /*: CoordModule['contents'] */ = (require /*:: as any */)('./coord.js');

// The below line will correctly generate a type error, since we're passing
// in a string where a number was expected.
const yourCoord /*: Coordinate */ = createCoordinate(2, '3');
```

I'm sure there's a variety of different ways to accomplish this task, some of which may be simpler than what is being presented above, but none of them are going to be very nice.

In short, avoid commonJS if you can. If you can't, perhaps Buildless TypeScript isn't the tool for the job.
