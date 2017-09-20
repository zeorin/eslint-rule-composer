# eslint-rule-composer

This is a utility that allows you to build [ESLint](https://eslint.org/) rules out of other ESLint rules.

## Installation

```
npm install eslint-rule-composer --save
```

Requires Node 4 or later.

## Examples

The following example creates a modified version of the [`no-unused-expressions`](https://eslint.org/docs/rules/no-unused-expressions) rule which does not report lines starting with `expect`.

```js
const ruleComposer = require('eslint-rule-composer');
const eslint = require('eslint');
const noUnusedExpressionsRule = new eslint.Linter().getRules().get('no-unused-expressions');

module.exports = ruleComposer.filterReports(
  noUnusedExpressionsRule,
  (problem, metadata) => metadata.sourceCode.getFirstToken(problem.node).value !== 'expect'
);
```

The following example creates a modified version of the [`semi`](https://eslint.org/docs/rules/semi) rule which reports missing semicolons after experimental class properties:

```js
const ruleComposer = require('eslint-rule-composer');
const eslint = require('eslint');
const semiRule = new eslint.Linter().getRules().get('semi');

module.exports = ruleComposer.joinReports([
  semiRule,
  context => ({
    ClassProperty(node) {
      if (context.getSourceCode().getLastToken(node).value !== ';') {
        context.report({ node, message: 'Missing semicolon.' })
      }
    }
  })
]);
```

## API

### `ruleComposer.filterReports(rule, predicate) and ruleComposer.mapReports(rule, predicate)`

Both of these functions accept two arguments: `rule` (an ESLint rule object) and `predicate` (a function)

`filterReports(rule, predicate)` returns a new rule such that whenever the original rule would have reported a problem, the new rule will report a problem only if `predicate` returns true for that problem.
`mapReports(rule, predicate)` returns a new rule such that whenever the original rule would have reported a problem, the new rule reports the result of calling `predicate` on the problem.

In both cases, `predicate` is called with two arguments: `problem` and `metadata`.

* `problem` is a normalized representation of a problem reported by the original rule. This has the following schema:

    ```
    {
      node: ASTNode | null,
      message: string,
      loc: {
        start: { line: number, column: number },
        end: { line: number, column: number } | null
      },
      fix: Function
    }
    ```

* `metadata` is an object containing information about the source text that was linted. This has a `sourceCode` property, which is a [`SourceCode`](https://eslint.org/docs/developer-guide/working-with-rules#contextgetsourcecode) instance corresponding to the linted text.

### `ruleComposer.joinReports(rules)`

Given an array of ESLint rule objects, `joinReports` returns a new rule that will report all of the problems from any of the rules in the array. The options provided to the new rule will also be provided to all of the rules in the array.

### Getting a reference to an ESLint rule

To get a reference to an ESLint core rule, you can use ESLint's [public API](https://eslint.org/docs/developer-guide/nodejs-api) like this:

```js
// get a reference to the 'semi' rule

const eslint = require('eslint');
const semiRule = new eslint.Linter().getRules().get('semi');
```

To get a reference to a rule from a plugin, you can do this:

```js
// get a reference to the 'react/boolean-prop-naming' rule
const booleanPropNamingRule = require('eslint-plugin-react').rules['boolean-prop-naming'];
```

You can also create your own rules (see the [rule documentation](https://eslint.org/docs/developer-guide/working-with-rules)):

```js
const myCustomRule = {
  create(context) {
    return {
      DebuggerStatement(node) {
        context.report({ node, message: 'Do not use debugger statements.' });
      }
    }
  }
};
```

## License

MIT License
