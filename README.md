# Debugger Operands for JS

Bradley Farias

Stage 0

This document proposes adding an optional operand to the <a href="https://tc39.github.io/ecma262/#prod-DebuggerStatement"><code>DebuggerStatement</code></a> production of JS.

## A guiding example: grouping breakpoints

The following would be an example of allowing a debugging facility to interpret the operand and provide an interface to group breakpoints into logical units.

```js
const log = (v) => {
  debugger { group: 'logging' };
  console.log(v);
};
```

## Why not a meta function.

Presenting this syntax as a meta function would imply that the operand is always evaluated. This proposal leaves it up to the host to determine if it would like to evaluate the operand.

### Detecting Debugging facilities

It should be noted that if the operand only runs when the debugging facility is active it potentially exposes this fact to JS code. Steps can be taken that avoid this such as: always evaluating the operand, preventing side effects while evaluating the operand, or never evaluating the operand.

Various libraries already are able to detect the code injection that debugging facilities use so this ability to detect debugging facilities would not be a new feature.

## Use cases

### Userland breakpoint grouping

```js
const log = (v) => {
  debugger { group: 'logging' };
  console.log(v);
};
```

This allows debugging facilities to group breakpoints rather than leaving them
opaque.

#### Alternatives

Debugging facilities could read labels of breakpoints to similar effect.

```js
const log = (v) => {
  logging:debugger;
  console.log(v);
};
```

However this only allows string based naming. With an operand based approach libraries could use Symbols or Objects as a means to be unique.

### Userland breakpoint conditions

```js
const route = (httpRequest) => {
  debugger {
    test: () => httpRequest.tracing || !isProduction
  };
  console.log(v);
};
```

#### Alternatives

Code can be wrapped in branches to determine if it should fire a breakpoint.

```js
const route = (httpRequest) => {
  if (httpRequest.tracing || !isProduction) {
    debugger;
  }
  console.log(v);
};
```

In order to reuse this logic the branch must be duplicated, or abstracted to a function call.

```js
const routeA = (httpRequest) => {
  if (httpRequest.tracing || !isProduction) {
    debugger;
  }
  console.log(v);
};
const routeB = (httpRequest) => {
  breakOnTracing(httpRequest);
  console.log(v);
};
```

### Userland debugging assistance

Lots of tools have browser extensions in order to achieve enhanced experiences such as [React Devtools](https://github.com/facebook/react-devtools). These tools rely on code injection of the main Realm usually and are not able to handle cases where code is not exposed to the main realm. Providing a means for debugging facilities to determine the type of the breakpoint also exposes the possibility for them to read the group of breakpoints and provide extra assistance to the developer.
