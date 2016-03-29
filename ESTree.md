This document specifies the extensions to the [ESTree ES6 AST][] types to
support the "export-ns-from" proposal.


# Modules

## ExportNamedDeclaration

```js
extend interface ExportNamedDeclaration {
    specifiers: [ ExportSpecifier | ExportNamespaceSpecifier ];
}
```

Extends the [ExportNamedDeclaration][], e.g., `export {foo} from "mod";` to
allow a new type of specifier: *ExportNamespaceSpecifier*.

_Note: When `source` is `null`, having `specifiers` include
*ExportNamespaceSpecifier* results in an invalid state._

_Note: Having `specifiers` include more than one *ExportNamespaceSpecifier*
results in an invalid state._

_Note: Having `specifiers` include both *ExportSpecifier* and
*ExportNamespaceSpecifier* results in an invalid state._


## ExportNamespaceSpecifier

```js
interface ExportNamespaceSpecifier <: Node {
    type: "ExportNamespaceSpecifier";
    exported: Identifier;
}
```

An exported binding `* as ns` in `export * as ns from "mod";`. The `exported`
field refers to the name exported in this module. That name is bound to the
ModuleNameSpace exotic object from the `source` of the parent *ExportNamedDeclaration*.


[ESTree ES6 AST]: https://github.com/estree/estree/blob/master/es6.md
[ExportNamedDeclaration]: https://github.com/estree/estree/blob/master/es6.md#exportnameddeclaration
