# lively-next-direct-renderer-dev

## What? Why?

This repository was used to develop a renderer for the [`lively.next`](https://github.com/LivelyKernel/lively.next) environment.

`lively.next` had a renderer that was based on the [`vdom`](https://github.com/Matt-Esch/virtual-dom) library by Matt-Esch. 
However, this had several downsides most importantly a paradigm mismatch between the declarative approach of the `vdom` and the imperativeness of `morphic` as well as the fact that due to the asynchronousness of the `vdom` and its decoupling from the actual DOM, `asnyc` and `await` were scattered everywhere through the code and we ended up more or less fighting against our own renderer.

To mitigate these problems and obtain more control over the rendering process, we decided to rewrite the renderer without `vdom`.

Instead, this new renderer should directly manipulate the DOM. We took some inspiration from the [stage0 library](https://github.com/Freak613/stage0).

---

This repository exists mainly for the reason that it was easier to develop a renderer outside of the main `lively.next` codebase.
The code for the renderer alone is not sufficient to run anything and there is no recoverable relationship between this repository and specific commits inside of the `lively.next` repository.
However, this repository remains, so that the development history of the renderer is still available.


This also means, that this repository probably does not contain the most up-to-date of the renderer code anymore. Instead, it would be in the `lively.next` repository. The new renderer was merged into `lively.next` with [this Pull Request](https://github.com/LivelyKernel/lively.next/pull/448).

## High-level Concept

When not using something like `vdom`, we need to execute DOM operations ourself. However, DOM operations are costly.
Due to the *virtual* part of itself, `vdom` comes away with rebuilding everything, every render cycle - because only a fraction of operations will end up in the DOM in the end.

We do not have this luxury, and thus, we need a mechanism to detect changes on a fine-granular level and patch them on the same level as well.

Where previously, we only had one flag indicating that indicated that a morph was changed, we now have several.
For example, there is a flag that indicates that a morph has a submorph added/removed, or one of its properties changed.

In every render cycle, we iterate over all existing morphs in the world, build queues dependent on the set flags and then perform the necessary operations in the DOM.

Due to the render cycle being bound to `requestAnimationFrame()`, it is still asynchronous. However, since we are no longer bound by building a complete virtual dom but can just look-up a concrete DOM node for every morph in the world, we can also synchronously push changes directly into the DOM.


## Interface to customize the rendering

Morphs can implement the following methods to customize their rendering:

- `renderStyles (styleProps)` gets passed the `styleProps` generated by feeding the morph to `defaultStyles` in the renderer. This object can then be customized. The return object of this function is then applied for the node style props.
- `renderAttributes(attrs)` see above, but for the other attributes of the node.
- `getNodeForRenderer(renderer)` can be implemented and cause a special kind of dom node to be created for the rendering of the morph. The methods returning the dom nodes are to be implemented in the renderer. I.e., the `getNodeForRenderer` implementation on `Morph` just does `return renderer.nodeForMorph();`, with `nodeForMorph()` returning a `DIV` node.
- `patchSpecialProps(node)` gets passed the node which represents the rendered morph in the DOM. This method can be used to perform/trigger special updating mechanisms that a morph needs. SVGs and the text-morph make use of this heavily, e.g., in order to check whether the selection inside of a text morph has been changed since the last rendering and then triggering the update mechanisms as appropriate. 

## LICENSE

MIT
