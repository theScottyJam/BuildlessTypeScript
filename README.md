# Buildless TypeScript

Buildless TypeScript is a TypeScript fork that allows you to use TypeScript syntax inside of comments in your JavaScript files.

Place your TypeScript syntax inside of a double-colon block comment (`/*:: ... */`), and use a single-colon block comment (`/*: ... */`) for type annotations. `/*: ... */` is really just a shorthand for doing `/*:: : ... */`.

```javascript
/*::
interface User {
  id: number
  firstName: string
  lastName: string
  role: string
}
*/
 
function updateUser(
  id /*: number */,
  update /*: Partial<User> */,
) {
  const user = getUser(id);
  const newUser = { ...user, ...update };
  saveUser(id, newUser);
}
```

Oh - and let's get this out of the way - if you are using CommonJS, then you probably do not want to use Buildless TypeScript. [You can try](https://github.com/theScottyJam/buildlesstypescript/blob/main/docs/using-commonjs.md), but it won't be pretty. If you use `require()`/`module.exports`, then you are using CommonJS. Even if you use ES module import syntax instead (`import ... from ...`), if your project is currently built with TypeScript, TypeScript may be transpiling your code down to CommonJS - check the `"module"` config field in your `tsconfig.json` file to see if it is set to `"commonjs"` or not.

What's in this document
* [Getting Started](#getting-started)
* [Editor support](#editor-support)
* [Auto-convert your entire TypeScript project to Buildless TypeScript](#converting-an-existing-project)
* [Ejecting back to TypeScript](#ejecting)
* [Limitations](#limitations)
* [Q&A](#QA)

## Getting Started

If you have an existing project that you would like to convert, then follow the [converting an existing project](#converting-an-existing-project) section before continuing on with these instructions.

Add Buildless Typescript as a dependency with:

```sh
npm install --save-dev buildlesstypescript
```

It is recommended that you add `"type": "module"` to your `package.json` config file, which ensures that all JavaScript files in your project are treated as ES modules instead of CommonJS modules.

If you don't have a `tsconfig.json` file yet, you can run `npx tsc --init` to generate one. You'll want to make sure the following fields are set in your `tsconfig.json` file:

```javascript
{
  "compilerOptions": {
    ...
    "allowJs": true,                                    /* Allow JavaScript files to be a part of your program. Use the 'checkJS' option to get errors from these files. */
    "checkJs": true,                                    /* Enable error reporting in type-checked JavaScript files. */
    "noEmit": true,                                     /* Disable emitting files from a compilation. */
    ...
  }
}
```

Alternatively, you can leave `"checkJs"` as `false` and instead place [`// @ts-check`](https://www.typescriptlang.org/docs/handbook/intro-to-js-ts.html) at the top of each module that you would like to enable type-checking on.

If you want to configure your project to emit declaration files (e.g. maybe you are building a library, and you want to provide types definitions for your consumers), you can configure your `tsconfig.json` like this instead:

```javascript
{
  "compilerOptions": {
    ...
    "allowJs": true,                                    /* Allow JavaScript files to be a part of your program. Use the 'checkJS' option to get errors from these files. */
    "checkJs": true,                                    /* Enable error reporting in type-checked JavaScript files. */
    "noEmit": false,                                    /* Disable emitting files from a compilation. */
    "declaration": true,                                /* Generate .d.ts files from TypeScript and JavaScript files in your project. */
    "emitDeclarationOnly": true,                        /* Only output d.ts files and not JavaScript files. */
    ...
  }
}
```

Lets test it out. Create a new file called `main.js` and add the following to it:

```javascript
/*::
interface Coordinate {
  readonly x: number
  readonly y: number
}
*/

function createCoordinate(
  x /*: number */,
  y /*: number */,
) /*: Coordinate */ {
  return { x, y };
}

createCoordinate(3, 'not a number');
```

For now, ignore any errors that your editor may be showing - those may be incorrect, and will be addressed in the [editor support](#editor-support) section.

Run `npx tsc`. It should report an error that looks like this:

```
main.js:15:21 - error TS2345: Argument of type 'string' is not assignable to parameter of type 'number'.

15 createCoordinate(3, 'not a number');
                       ~~~~~~~~~~~~~~
```

If you configured your project to emit declaration files, you should see `.d.ts` files appear next to your `.js` files as well.

Now fix the error by changing `'not a number'` into a real number then re-run `npx tsc`.

## Editor Support

The Buildless TypeScript library installed inside of your project provides a language server that your editor can talk to. This language server provides your editor with syntax highlighting support, type error information, refactoring helpers, and more - you just have to get your editor to use the custom language server.

In the case of VS code, the VS Code editor comes with its own internal TypeScript installation that it uses by default, but [you can ask it to switch to the workspace's version instead](https://code.visualstudio.com/docs/typescript/typescript-compiling#_using-the-workspace-version-of-typescript). Add a `.vscode` folder to the root of your project with a `settings.json` file containing the following:

```
{
  "typescript.tsdk": "node_modules/buildlesstypescript/lib"
}
```

Next, open your command palette (typically `ctrl+shift+P`) and run the command "TypeScript: Select TypeScript Version". An option should appear to let you use the workspace version of TypeScript.

Most editors should have similar features available.

## Converting an Existing Project

Buildless TypeScript comes with a CLI tool that can be used to convert your existing TypeScript project into Buildless TypeScript. The only thing the tool does is rename files from `.ts` to `.js` and add TS comment delimiters to it (the `/*::`, `/*:`, and `*/` stuff) - this means it has some important limitations to be aware of:
* Most TypeScript syntax has no effect on the runtime behavior of your codebase, but there are a few pieces of older syntax that do, including enums, namespaces, and more. The auto-conversion tool is not smart enough to handle syntax like this - it'll treat it the same as all other TypeScript syntax - sticking it inside of TS comments.
- TypeScript can be used to transpile a project into an older version of JavaScript. Check your project's `package.json`'s `"target"` field to see what version of JavaScript it is currently configured to transpile to. Once you've switched to Buildless TypeScript, there won't be a transpilation step anymore, and that `"target"` field will be ignored.
* TypeScript can automatically convert ES module syntax (`import ... from ...`) into CommonJS (`require(...)`). Check your `package.json`'s `"module"` field - if it is set to `"commonjs"` that means it is automatically performing this conversion for you. Buildless TypeScript works best with ES modules - you can support CommonJS, but [it is a pain](https://github.com/theScottyJam/buildlesstypescript/blob/main/docs/using-commonjs.md), and this converter tool will not automatically convert your ES import/export syntax into CommonJS.
* No adjustments will be made to your build setup. For example, if your `package.json` expects files to be in a `build/` folder, and you've switched to Buildless TypeScript, then you won't have a `build/` folder anymore and you'll need to make the appropriate adjustments. Depending on how your unit tests are set up, their configuration may need tweaking as well.
* While it does a fairly good job at formatting the comment delimiters, it is not perfect and may make some odd choices here and there.

To convert a project, simply open a shell at the root of your project and run the command `npx --package=buildlesstypescript tsc --buildlessConvert`. This will automatically convert all files in your project that would typically get type-checked. If you want to control which files are being converted, the easiest option is to temporarily [add/edit the include/exclude fields](https://www.typescriptlang.org/tsconfig/#include in `tsconfig.json`) before running the conversion command.

## Ejecting

An ejection tool is provided to automatically convert your Buildless TypeScript files into normal TypeScript files. The tool does nothing more than remove TypeScript comment delimiters (the `/*::`, `/*:`, and `*/` stuff) and rename your `.js` files to `.ts`. As such, there are a couple of things to be aware of:
* It will not configure your build step for you, you will have to configure that after you eject.
* For the most part it should do fine at formatting the final TypeScript file, especially if you loosely follow [the Buildless TypeScript style guide](https://github.com/theScottyJam/buildlesstypescript/blob/main/docs/style-guide.md) - but don't feel pressured into following that guide, its completely optional.

To eject a project, simply open a shell at the root of your project and run the command `npx tsc --buildlessEject`. This will automatically eject all files in your project that would typically get type-checked. If you want to control which files are being converted, the easiest option is to temporarily [add/edit the include/exclude fields](https://www.typescriptlang.org/tsconfig/#include in `tsconfig.json`) before running the conversion command.

If you want to eject to JavaScript instead of TypeScript, that can be done through a two-step process - first convert to TypeScript, then build the TypeScript project. You can then replace your source code with the build artifacts.

## limitations

Known low-priority issues. There are plans to fix these issues in the future.
* You must use the `as` keyword to do type assertions. Old-style angle-bracket assertions (e.g. `<number>myNumber`) do not work.
* It may allow you to put JavaScript syntax inside of TS comments without reporting a syntax error.
* If you put TypeScript syntax in a JavaScript file, it will tell you to move it to a TypeScript file. What it should say instead is to move it into a TS comment.

Known limitations. There are currently no plans to change these - mostly in an effort to keep this fork from getting too complicated.
* The single-colon and double-colon comments only work with block comments. You can not use line comments. e.g. `//: ...` and `//:: ...` do not work.
* There are no plans to support JSX - JSX requires a build-step anyways, so might as well use tsx instead if you want TypeScript support.

## Q&A

### How do you write JSDoc comments inside of a TS comment?

For example, say you are defining an interface and you'd like to document some of its properties. In TypeScript you would be able to write the following:

```typescript
interface User {
  readonly username: string
  readonly birthday: Date
  /** @deprecated */
  readonly age: number
}
```

With Buildless TypeScript, you can add JSDoc in the middle of your interface definition in the exact same way, you just have to be wary of the fact that the closing block comment delimiter (`*/`) is going to close both the JSDoc comment and your TS comment. To fix that, just re-open your TS comment right afterwards, like this:

```javascript
/*::
interface User {
  readonly username: string
  readonly birthday: Date
  /** @deprecated *//*::
  readonly age: number
}
*/
```

### What's up with the huge version numbers?

This project's version numbers can be parsed as follows - If you omit the last two digits of each segment, you will get a TypeScript version number, so a Buildless TypeScript version number of `500.301.304` really means "TypeScript version 5.1.4". The remaining digits are to allow Buildless TypeScript to release semver compatible releases between TypeScript versions. With the ``500.301.304`` example again, that version number says that `4` patch releases and `1` minor release have come out for this tool since it provided the TypeScript `5.1.4` release. Every time this fork incorporates a new TypeScript version, it will reset the last two digits back to `00`.

### How should I format the TS comments?

For those who like being told how to format their code, [a style guide is available](https://github.com/theScottyJam/buildlesstypescript/blob/main/docs/style-guide.md). If you don't like being told what to do, don't click on the link.

### This project is awesome, how can I contribute?

The best way to contribute is by going to [this TypeScript feature request](https://github.com/microsoft/TypeScript/issues/48650) and adding a thumbs up to the proposal, asking for TypeScript-in-comments to become a native feature.
