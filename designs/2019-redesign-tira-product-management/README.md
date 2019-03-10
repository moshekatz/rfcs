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
* Offer developers a better way to create a Tira product.
* Allow developers to design their product independently.
* Reduce the maintenance complexity of the abstraction layer.

## Motivation

Currently, managing products in Tira comes with a lot overhead for developers, both consumers and maintainers:

* You have to create a project in the server and override a lot of methods, sometimes in a naive way just for the compiler to be happy.
* You have to create a bunch of insert statements without knowing what you gain in return.
* While creating a view from the available templates is quick, it introduce a trade off, and changing the view's logic or replace it becomes non trivial.
* Encourages bad practices so that the code you write is very hard to reuse outside the context of Tira's infrastructure.

This leads to several problems:

* The ROI (return on investment) isn't effective for a lot of bugs/features, reducing the product's quality and leads to mediocrity.
* Large knowledge gap - a lot of the problems developers encounter aren't google searchable and very hard to debug/locate in the codebase - so the easiest/fastest way to solve those is to call a senior.
* The on-boarding proccess for new developers includes a few weeks of learning Tira's infrastructure and vocabulary - with minimal correlation to what they previously learned.

## Detailed Design

codesandbox example: https://codesandbox.io/s/1v87rqvxp3

### Creating a product

#### 1. Create a product directory
Create a directory with your product name under `src/products/{your-product-name}`.
All of your product logic should be placed here as a .

#### 2. Create a `productDefinition`

A product definition is an object that includes all the necessary information of a product type.
Inside your product's directory, create a file named `{your-product-name}-definition.ts` and export as default the following object:

```js
// TODO: add a type definition
export default { metadata, mount, unmount, onenter, onleave }
```

##### `metadata`
An object that includes the following: 
1. __name__ : string - the products name
2. __type__ : subdp - the product type, generated from tira's subdp sequence
3. __displayName__ : string - a single display name

```js
// OpRep example
const metadata = {
    name: 'דיווח מבצעי',
    type: 710022
    displayName: 'דיווחים' 
}
```

##### `mount({container, id})`
A function that accept a DOM node (container) and id,
and mounts the component to the DOM. 

```js
// React example
function mount({container, id}) {
    ReactDOM.render(<ReportView reportId={id} />, container);
}

// Native DOM API example
function mount({container, id}) {
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
function unmount({container}) {
    // Remove a mounted React component from the DOM 
    // and clean up its event handlers and state. 
    ReactDOM.unmountComponentAtNode(container);
}
```

##### `onEnter()` - open question
A function that will get called when entering the product.

##### `onLeave()` - open question
A function that will get called when leaving the product.

#### 3. Add a mapping inside `products-map.ts`

```js
import opRepDefinition from "./OpRep/oprep-definition";

const productsMap = new Map();

productsMap.set(opRepDefinition.metadata.type, opRepDefinition);
```
__Note:__ This step can be automated using babel-plugin-macros

### Changes to the workflow

TBD

### Changes to the core logic

TBD


## Documentation

TBD

## Drawbacks

TBD

## Backwards Compatibility Analysis

TBD

## Alternatives

TBD

## Open Questions

TBD

## Related Discussions

TBD
