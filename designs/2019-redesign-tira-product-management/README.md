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

### Creating new product

#### 1. Create a product directory
Create a directory with your product name under `src/products/{your-product-name}`.

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

e.g: 
```js
const metadata = {
    name: 'דיווח מבצעי',
    type: 710022
    displayName: 'דיווחים' 
}
```


### Changes to the workflow


## Documentation

This change would effectively remove the distinction between "local" and "global" ESLint installations, because the location of the ESLint package would no longer be relevant. As a result, we would need to update the documentation that describes the distinction between local and global installations, and more generally modify the parts of the documentation that describe config loading to ensure that they were up to date.

Our existing advice for config/plugin authors regarding dependencies (namely, that parsers and configs can be dependencies, while plugins must be peerDependencies) would remain the same.

We would also need to update the `generator-eslint` boilerplate for plugins to avoid generating a readme file that describes a distinction between local and global installations.

## Drawbacks

* A lot of existing documentation says that global ESLint installations require global plugin installations. (This includes README files in third-party plugins which we're unable to modify.) As a result, this change could temporarily cause confusion for new users who choose to use global installations.
* This change is backwards-incompatible for end users who use global plugin installations or home-directory configs, which could cause some migration pain. (However, the change is compatible with existing shareable configs and plugins, so end users would be able to migrate without waiting for a third party to migrate.)
* This change does not allow shareable configs to manage their own plugins, as proposed in [RFC #5](https://github.com/eslint/rfcs/pull/5). However, it is intended to be compatible with future solutions to that problem.

## Backwards Compatibility Analysis

* This change has no compatibility impact on shareable configs and plugins. Shareable configs already specify plugins as `peerDependencies`, implying that the plugins must be installed and loadable by the end user. This change simply updates ESLint to use that guarantee.
* This is a breaking change for end users who use global ESLint installations and global plugin installations; the plugins must now be installed locally.
* This is a breaking change for end users who use home-directory configs along with shareable configs or custom parsers; the shareable configs/parsers must be resolvable from the location of the home-directory config.
* This is a breaking change for integration authors who depend on `Linter` loading custom parsers from the filesystem. These integration authors could explicitly use `Linter#defineParser` instead. (Most integration authors who need access to the filesystem are using `CLIEngine` anyway.)

## Alternatives

* Along with configs/parsers, ESLint could load plugins relative to the configs that depend on them. That would introduce the possibility of having two plugins with the same name, requiring a disambiguation mechanism for configuring that plugin's resources; see [RFC #5](https://github.com/eslint/rfcs/pull/5) for an example. This RFC is intended to be a simpler step to solve some problems, while allowing additional work to proceed on that problem if desirable.
* To mitigate some compatibility impact, ESLint could fall back to the existing behavior when it fails to load a package using the new behavior, instead of throwing a fatal error. However, this would likely cause more confusion because users wouldn't understand why ESLint was finding their packages in certain cases, which would make it more difficult to debug problems. This fallback is only provided for the `espree` package (which falls back to the default parser if no user-installed version of the pacakge is found).

## Open Questions

* This proposal uses the CWD as the "project root" from which plugins are loaded. Are there other folders that would serve this purpose better (e.g. in cases where the `--config` flag is used to load a config from a separate project)?

## Related Discussions

* [eslint/eslint#10125](https://github.com/eslint/eslint/issues/10125)
* [eslint/rfcs#5](https://github.com/eslint/rfcs/pull/5)
* [eslint/eslint#3458](https://github.com/eslint/eslint/issues/3458)
* [eslint/eslint#6732](https://github.com/eslint/eslint/issues/6732)
* [eslint/eslint#9897](https://github.com/eslint/eslint/issues/9897)
* [eslint/eslint#10643](https://github.com/eslint/eslint/issues/10643)
