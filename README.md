# ECMAScript Proposal: export ns from

**Stage:** N/A - this instead became a "needs consensus" PR, and is [merged into ECMA-262](https://github.com/tc39/ecma262/pull/1174)!

**Champion:** Valerie Young

**Author:** Lee Byron

**Reviewers:**
  - Caridy Patiño
  - John-David Dalton
  - Leo Balter

**Specification:** https://tc39.github.io/proposal-export-ns-from/

**AST:** [ESTree.md](./ESTree.md)

**Transpiler:** See Babel's [export-namespace-from](https://babeljs.io/docs/en/babel-plugin-proposal-export-namespace-from) plugin.

> NOTE: Closely related to the [export-default-from](https://github.com/tc39/proposal-export-default-from) proposal.

## Problem statement and rationale

The `export ___ from "module"` statements are a very useful mechanism for
building up "package" modules in a declarative way. In the ECMAScript 2015 spec,
we can:

* export through a single export with `export {x} from "mod"`
* ...optionally renaming it with `export {x as v} from "mod"`.
* We can also spread all exports with `export * from "mod"`.

These three export-from statements are easy to understand if you understand the
semantics of the similar looking import statements.

However there is an import statement which does not have corresponding
export-from statement, exporting the ModuleNameSpace object as a named export.

Example:

```js
export * as someIdentifier from "someModule";
```


### Current ECMAScript 2015 Modules:

Import Statement Form         | [[ModuleRequest]] | [[ImportName]] | [[LocalName]]
---------------------         | ----------------- | -------------- | -------------
`import v from "mod";`        | `"mod"`           | `"default"`    | `"v"`
`import * as ns from "mod";`  | `"mod"`           | `"*"`          | `"ns"`
`import {x} from "mod";`      | `"mod"`           | `"x"`          | `"x"`
`import {x as v} from "mod";` | `"mod"`           | `"x"`          | `"v"`
`import "mod";`               |                   |                |


Export Statement Form           | [[ModuleRequest]] | [[ImportName]] | [[LocalName]] | [[ExportName]]
---------------------           | ----------------- | -------------- | ------------- | --------------
`export var v;`                 | **null**          | **null**       | `"v"`         | `"v"`
`export default function f(){};`| **null**          | **null**       | `"f"`         | `"default"`
`export default function(){};`  | **null**          | **null**       | `"*default*"` | `"default"`
`export default 42;`            | **null**          | **null**       | `"*default*"` | `"default"`
`export {x}`;                   | **null**          | **null**       | `"x"`         | `"x"`
`export {x as v}`;              | **null**          | **null**       | `"x"`         | `"v"`
`export {x} from "mod"`;        | `"mod"`           | `"x"`          | **null**      | `"x"`
`export {x as v} from "mod"`;   | `"mod"`           | `"x"`          | **null**      | `"v"`
`export * from "mod"`;          | `"mod"`           | `"*"`          | **null**      | **null**


### Proposed addition:

Export Statement Form           | [[ModuleRequest]] | [[ImportName]] | [[LocalName]] | [[ExportName]]
---------------------           | ----------------- | -------------- | ------------- | --------------
`export * as ns from "mod";`    | `"mod"`           | `"*"`          | **null**      | `"ns"`


## Symmetry between import and export

There's a syntactic symmetry between the export-from statements and the import
statements they resemble. There is also a semantic symmetry; where import
creates a locally named binding, export-from creates an export entry.

As an existing example:

```js
import {v} from "mod";
```

If then `v` should be exported, this can be followed by an `export`. However, if
`v` is unused in the local scope, then it has introduced a name to the local
scope unnecessarily.

```js
import {v} from "mod";
export {v};
```

A single "export from" line directly creates an export entry, and does not alter
the local scope. It is *symmetric* to the similar "import from" statement.

```js
export {v} from "mod";
```

This presents a developer experience where it is expected that replacing the
word `import` with `export` will always provide this symmetric behavior.

However, when there is a gap in this symmetry, it can lead to confusing behavior:

> "I would like to chime in with use-case evidence. When I began using ES6 (via
> babel) and discovered imports could be re-exported I assumed the feature set
> would be symmetrical only to find out quite surprisingly that it was not. I
> would love to see this spec round out what I believe is a slightly incomplete
> feature set in ES6."
>
> - @jasonkuhrt
>
> "I also bumped into this and even thought this was a bug in Babel."
>
> - @gaearon

### Proposed addition:

The proposed addition follows this same symmetric pattern:

Importing a namespace exotic object (existing):

```js
import * as ns from "mod";
```

Exporting that name (existing):

```js
import * as ns from "mod";
export {ns};
```

Symmetric "export from" (proposed):

```js
export * as ns from "mod";
```


### Table showing symmetry

Using the terminology of [Table 40][] and [Table 42][] in ECMAScript 2015, the
export-from form can be created from the symmetric import form by setting
export-from's **[[ExportName]]** to import's **[[LocalName]]** and export-from's
**[[LocalName]]** to **null**.

[Table 40]: http://www.ecma-international.org/ecma-262/6.0/#table-40
[Table 42]: http://www.ecma-international.org/ecma-262/6.0/#table-42

Statement Form                          | [[ModuleRequest]] | [[ImportName]] | [[LocalName]]  | [[ExportName]]
--------------                          | ----------------- | -------------- | -------------- | --------------
`import v from "mod";`                  | `"mod"`           | `"default"`    | `"v"`          |
`import {x} from "mod";`                | `"mod"`           | `"x"`          | `"x"`          |
`export {x} from "mod";`                | `"mod"`           | `"x"`          | **null**       | `"x"`
`import {x as v} from "mod";`           | `"mod"`           | `"x"`          | `"v"`          |
`export {x as v} from "mod";`           | `"mod"`           | `"x"`          | **null**       | `"v"`
`import * as ns from "mod";`            | `"mod"`           | `"*"`          | `"ns"`         |
<ins>`export * as ns from "mod";`</ins> | `"mod"`           | `"*"`          | **null**       | `"ns"`
`export * from "mod";`                  | `"mod"`           | `"*"`          | **null**       | **null** (many)
