# WIP

- Start Date: 2019-03-06
- RFC PR: (leave this empty, to be filled in later)
- Author: Moshe Katz

# Redesign Tira's Product Development

## Summary

This change would introduce a new way to develop products in Tira.
The proposal would simplify creating, modifying and maintaining products in Tira, resulting in a better developer experience and better products.
It would also offer new layouts that weren't possible with the current implementation (goodbye splitter).

The goals of this proposal are:

- Offer developers a better way to create a Tira product.
- Allow developers to design their product independently.
- Reduce the maintenance complexity of the abstraction layer.

## Motivation

Currently, managing products in Tira comes with a lot overhead for developers, both consumers and maintainers:

- You have to create a project in the server and override a lot of methods, sometimes in a naive way just for the compiler to be happy.
- You have to create a bunch of insert statements without knowing what you gain in return.
- The client performs multiple requests to the server just to recieve data that rarely changes (e.g: products name).
- While creating a view from the available templates is quick, it introduce a trade off, and changing the view becomes non trivial.
- Encourages bad practices so that the code you write is very hard to reuse outside the context of Tira's infrastructure.

This leads to several problems:

- The ROI (return on investment) isn't effective for a lot of bugs/features, reducing the product's quality and leads to mediocrity.
- Large knowledge gap - a lot of the problems developers encounter aren't google searchable and very hard to debug/locate in the codebase - so the easiest/fastest way to solve those is to call a senior.
- The on-boarding proccess for new developers includes a few weeks of learning Tira's infrastructure and vocabulary - with minimal correlation to what they previously learned.

## Detailed Design

### Creating a product

As a consumer, this is the only section you have to understand in order to be productive.

#### 1. Create a product directory

Create a directory with your product name under `src/products/{your-product-name}`.
All of your product logic should be placed here as a convention.

#### 2. Create a `productDefinition`

A product definition is an object that includes all the necessary information of a product type.
Inside your product's directory, create a file named `{your-product-name}-definition.ts` and export as default the following object:

```js
export default { metadata, mount, unmount };
```

##### `metadata`

An object that includes the following:

1. **name** : string - the products name
2. **type** : subdp - the product type, generated from tira's subdp sequence
3. **displays** : display[] - an array of displays. each display includes an id and a name.

```js
// OpRep example
const metadata = {
    name: 'דיווח מבצעי',
    type: 710022
    displays: [{id: 1, name: 'דיווחים'}]
}
```

##### Why Tira subdp as a type?

It will reduce the cost of integrating with Tira's current infra, and will help with our "no big rewrites" strategy.

##### `mount({container, id})`

A function that accept a DOM node (container) and an id,
and mounts the component to the DOM.

```js
// React example
function mount({ container, id }) {
  ReactDOM.render(<ReportView reportId={id} />, container);
}

// Native DOM API example
function mount({ container, id }) {
  const button = document.createElement('button');
  button.textContent = `ahalan ${id}`;
  container.appendChild(button);
}
```

##### `unmount({container})`

A function that accept a DOM node (container),
and perform a clean up before final removal of the container from the DOM.

```js
// React example
function unmount({ container }) {
  ReactDOM.unmountComponentAtNode(container);
}
```

#### 3. Add a mapping inside `src/products/products-map.ts`

```js
// 1. Import your product definition
import opRepDefinition from './OpRep/oprep-definition';

const productsMap = new Map();

// 2. Map between the product type and definition
productsMap.set(opRepDefinition.metadata.type, opRepDefinition);
```

**Note:** This step can be automated using babel-plugin-macros

##### Why mapping by type?

In order to support products that renders different content for each id (e.g - spotfire, tati..).
This is also the reason we pass the "id" as a param to mount.

### Changes to the flow

##### Current

![Current Flow](images/current-flow.png?raw=true 'Current Flow')

##### Post Proposal

![Post Proposal Flow](images/post-proposal-flow.png?raw=true 'Post Proposal Flow')

##### End Game

![End Game Flow](images/end-game-flow.png?raw=true 'End Game Flow')

### Changes to the core logic

Codesandbox example: https://codesandbox.io/s/1v87rqvxp3

### Map Integration

### Serialization Integration

## Documentation

TBD

## Drawbacks

TBD

## Backwards Compatibility Analysis

TBD

## Alternatives

#### Full blown micro front ends

1. Adding complexity for no

## Open Questions

##### Support for `onEnter()` as part of the product definition?

A function that will get called when entering the product.

##### Support for `onLeave()` as part of the product definition?

A function that will get called when leaving the product.

##### Normalizing the state shape?

The state isn't nested so this might be an overkill.
More info can be found here: https://redux.js.org/recipes/structuring-reducers/normalizing-state-shape.

##### Using `immer` for the reducer?

Immer helps write immutable functions with regular readable syntax.
More info about immer can be found here: https://hackernoon.com/introducing-immer-immutability-the-easy-way-9d73d8f71cb3

##### client side vs. server side permissions?

##### `api` folder or seperated folders?

## Related Discussions

TBD
