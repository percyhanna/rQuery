[![Build Status](https://travis-ci.org/percyhanna/rquery.svg?branch=master)](https://travis-ci.org/percyhanna/rquery)

# rquery
A [React](http://facebook.github.io/react/) tree traversal utility similar to
jQuery, which can be useful for making assertions on your components in your
tests.

## Vision
[`chai-react`](https://github.com/percyhanna/chai-react/) was originally built
to help with test assertions of React components. However, it quickly started
adding too much complexity because it was attempting to solve two problems: 1)
making assertions of properties/rendered content and 2) traversing the rendered
React tree to make those assertions.

`rquery` is meant to take over the rendered tree traversing responsibility from
`chai-react`, which will allow it to be used with any testing framework. It will
also provide convenience wrappers for various common test actions, such as event
dispatching.

## React Version Support

* For React v15, use `rquery` version 5+
* For React v0.14, use `rquery` version 4.x

## Setup

### Node.js, Webpack, Browserify

```javascript
var _ = require('lodash');
var React = require('react');
var ReactDOM = require('react-dom');
var TestUtils = require('react-addons-test-utils');
var $R = require('rquery')(_, React, ReactDOM, TestUtils);
```

### Browser with Scripttags

Include React, lodash, and rquery in the page, then you get the `$R` global.

Sample usage:

```html
<script src="https://fb.me/react-with-addons-0.14.1.js"></script>
<script src="https://fb.me/react-dom-0.14.1.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/lodash.js/3.6.0/lodash.min.js"></script>
<script src="rquery.js"></script>
<script>
var component = React.createClass({
  render: function () {
    return React.createElement('h1', {}, 'Hello, world!');
  }
});

var el = document.createElement('div');
document.body.appendChild(el);

var comp = ReactDOM.render(React.createElement(component), el);

var $r = $R(comp);

console.log($r.text()); // 'Hello, world!'
</script>
```

## Usage

### $R Factory

The `$R` factory method returns a new instance of an `rquery` object.

**Example**:

```javascript
var $r = $R(component);
```

### Class Methods

#### `.extend`

```javascript
$R.extend({
  customMethod: function () {
    // your own custom method
  }
});
```

The `extend` method allows you to add extra methods to the `rquery` prototype. It
does not allow you to override internal methods, though.

#### `.isRQuery`

```javascript
$R.isRQuery('abc'); // false
$R.isRQuery($R([])); // true
```

Returns `true` if the provided argument is an instance of the `rquery`
prototype.

### *Instance Methods*

An instance of the `rquery` class contains an array of components, and provides
an `Array`-like interface to directly access each component.

**Example**:

```javascript
var $r = $R([component1, component2 /* , componentN */]);
$r.length === 2; // true
$r[0] === component1; // true
$r[1] === component2; // true
```

#### `#find`

```javascript
$r.find(selector)
```

Returns a new `rquery` instance with the components that match the provided
selector (see [Selector](#selectors) documentation).

#### `#prop`

```javascript
$r.prop('a')
```

Returns the value for the given prop for the first component in the scope. It
throws an error if the scope has no components.

#### `#style`

```javascript
$r.style('a')
```

Returns the value for the given style for the first component in the scope. The
style value is loaded from the style property. It throws an error if the scope
has no components.

#### `#state`

```javascript
$r.state('a')
```

Returns the value for the given state for the first component in the scope. It
throws an error if the scope has no components.

#### `#nodes`

```javascript
$r.nodes()
```

Returns an array of each DOM node in the current scope.

#### `#text`

```javascript
$r.text()
```

Returns the text contents of the component(s) in the `$r` object. Similar to
jQuery's `text()` method (read-only).

#### `#html`

```javascript
$r.html()
```

Returns the HTML contents of the component(s) in the `$r` object. Similar to
jQuery's `html()` method (read-only).

#### `#simulateEvent`

```javascript
simulateEvent(eventName, eventData)
```

Simulates triggering the `eventName` DOM event on the component(s) in the rquery
object.

#### Event helpers

```javascript
[eventName](eventData)
```

Convenience helper methods to trigger any supported React DOM event. See the
[React documentation](http://facebook.github.io/react/docs/events.html) to read
about the events that are currently supported.

## Selectors

### Scope Selectors

#### Descendant Selector

**Example**:

```javascript
$R(component).find('div p');
```

**Description**:

Finds all elements that are descendants of a parent component/node. Note that
the root element of a component will be matched as a descendant of it
(e.g. `MyComponent > div` will match `<div>` at root of `MyComponent#render`).

#### Child Selector

**Example**:

```javascript
$R(component).find('div > p');
```

**Description**:

Finds all elements that are children (direct descendant) of a parent
component/node.

#### Union Selector

**Example**:

```javascript
$R(component).find('div, p');
```

**Description**:

Matches a union of all selectors on both sides of the `,`.

#### Not Selector

**Example**:

```javascript
$R(component).find('div :not(p)');
```

**Description**:

Matches all elements that do _not_ match the nested selector. **NB**: Note the
difference between `div :not(p)` and `div:not(p)`. The latter will match nothing
as the `:not` is being applied directly on the `div`. However, `div :not(p)`
applies a descendant scope selector first, which means it will match all
elements that are descendants of `div` but not a `p`.

### Element/Component Selectors

#### Component Selector

**Example**:

```javascript
$R(component).find('MyComponentName');
$R(component, 'MyButton');
```

**Description**:

Traverses the tree to find components based on their `displayName` value. *NB*:
the selector must start with an upper-case letter, to signify a
CompositeComponent vs. a DOM component.

#### DOM Tag Selector

**Example**:

```javascript
$R(component).find('div');
$R(component, 'p');
```

**Description**:

Traverses the tree to find DOM components based on their `tagName`. *NB*: the
selector must start with a lower-case letter, to signify a CompositeComponent
vs. a DOM component.

#### DOM Class Selector

**Example**:

```javascript
$R(component).find('.button');
$R(component, '.green');
```

**Description**:

Traverses the tree to find components with `className`s that contain the
specified class.

#### Index Selector

**Example**:

```javascript
$R(component).find('div[2]'); // matches the third div
```

**Description**:

Matches the element at the given index. This is shorthand for `.at(index)` on
the `rquery` object.

#### Attribute Selector

**Example**:

```javascript
$R(component).find('[target]');
$R(component, '[onClick]');
```

**Description**:

Traverses the tree to find components that have a value defined for the given
property name.

*Note:* Although these are labeled as *attribute* selectors, they are really
*property* selectors. In other words, they match properties being passed to a
DOM/Composite component, not actual DOM attributes being rendered.

#### Attribute Value Selectors

**Example**:

```javascript
$R(component).find('[target="_blank"]');
$R(component, '[href="http://www.github.com/"]');
```

**Supported Operators**:

`rquery` supports the [CSS Selectors level 3 spec]
(http://dev.w3.org/csswg/selectors-3/#attribute-selectors):

* `[att="val"]`: equality
* `[att~="val"]`: whitespace-separated list
* `[att|="val"]`: namespace-prefixed (e.g. `val` or `val-*`)
* `[att^="val"]`: prefix
* `[att$="val"]`: suffix
* `[att*="val"]`: substring

**Description**:

Traverses the tree to find components with a property value that matches the
given key/value pair.

*Note:* Although these are labeled as *attribute* selectors, they are really
*property* selectors. In other words, they match properties being passed to a
DOM/Composite component, not actual DOM attributes being rendered. For complex
property values (e.g. arrays, objects, etc.), the value matchers are less useful
as `rquery` doesn't currently support any complex value matching.

*Note:* All values *must* be provided as double-quoted strings. `[att="val"]` is
valid, but `[att=val]` and `[att='val']` are not.

## Usage with Test Suites

The rquery interface is meant to be generic enough to use with any assertion
library/test runner.

Sample usage with Chai BDD style assertions:

```javascript
expect($R(component).find('MyComponent')).to.have.length(1);
