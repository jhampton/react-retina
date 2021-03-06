# React-RETINA

React-RETINA adds the ability for React components to render to `<canvas>`.
It is an alternative to React-Canvas by Flipboard, and also can be a substitute for the React-ART library.
Much of the React-Canvas documentation also applies to React-RETINA.  This is the original blog post that gave me the inspiration to write my own `<canvas>` implementation for React/Preact:

[60 FPS On The Mobile Web](http://engineering.flipboard.com/2015/02/mobile-web)

However, React-RETINA has a **very different** implementation to React-Canvas/React-ART and is not a drop-in replacement - look at demo code for examples on how to use APIs unique to React-RETINA.

## Copyright and Licensing

(c) Copyright 2017 by X. Phung/Farseek Pty Ltd.
Refer to LICENSE file for use/licensing information.

## Comparison with React-Canvas/React-ART

1.  React-RETINA only uses **standard/public React APIs**.
2.  The React-RETINA design **avoids numerous React anti-patterns** (which React Canvas/React ART are guilty of using).  There are no mixins, string Refs, or even Contexts.  It does not use non standard lifecycle methods 
3.  The React-RETINA component library source code is much easier to understand as a result and is **leaner and cleaner**.
4.  However, React-RETINA has some very limited direct DOM manipulation via function Refs.  The DOM manipulation code is entirely encapsulated into the LayerUtils module, and is invisible to other modules.
5.  All DOM nodes are minimal stub-like children of the `<canvas>` element.  They are ignored by all modern browsers, and hence have almost zero performance impact.  The DOM child nodes exist in React-RETINA for three reasons: (i) the rendering of these DOM nodes is a side effect of the standard React API (ie: React-Canvas and React-ART do not render these DOM nodes because they use non standard mount/receive React APIs), (ii) the standard DOM `title` attribute is optionally supported to provide **accessibility to vision impaired users** (disable this by omitting the `title` property), (iii) extra attributes are added to the React-RETINA DOM nodes to provide a clean separation between React Component-like behaviours - versus hidden/underlying implementation of event management and layer repainting (both done by LayerUtils).
5.  React-RETINA works with Preact!! (Needs preact-compat).  This is a consequence of using only standard React APIs.
6.  Babel/JSX are not required.  Instead React-RETINA's components assign `React.createElement` to the alias `h` and use this like [hyperscript's](https://github.com/hyperhype/hyperscript) `h`.  However, you can use JSX in your own code if you wish.

## Features Overview

1.  **src/components/Surface** is the top level React-RETINA component and is the drawing canvas on which all other components are rendered.  It needs to be the root of all other React-RETINA components.
2.  **src/components/View** is the base 'low level' React-RETINA component.  It provides event management, hooks for updating/repainting and the notion of 'Layers' (rectangular regions identified by style left/top/width/height/translateX/translateY/scale properties).  Layers can be optionally cached into a backing store.  View is an extremely light weight component - it is little more than a notional sub-rectangle of the `<canvas>` bitmap but does not draw anything into the `<canvas>` itself.  As a React component, View inherits the standard React containment functionality via the props.children property, and these children may provide the canvas drawing (Z order is fixed to children above parents, and N-th siblings above (N-1)th siblings).  Alternatively, containers of View may provide a draw callback.
3.  **src/components/Text** is simple single line text string hosting it's own View.  As I don't have any need for React-Canvas' multi-line truncation and text metrics features, these have been left out but you can very easily write your own (see **Custom draw components**) below.
4.  **src/components/Image** is a single image hosted it's own View with a few added features like management of the loading lifecycle, fade-in after loading, and image pooling.
5.  **src/components/Rect** contains canvas drawing code for rectangles with/without borders.
6.  **Custom draw components**: You can create your own custom drawing components by providing a props.draw callback in the same manner as Rect, Image or Text.
7.  **Events**: supports keyboard/wheel/click events in addition to touch events.  One reason I developed React-RETINA instead of using React-ART/React-Canvas is that they do not support keyboard events.  (Note: testing has been primarily been on desktop Chrome and OS X WKWebKit, and touch events have not been fully tested).
8.  **Event bubbling** from child to parent components is supported.  `Event.target` identifies the target of an event, whereas the event can be handled by any parent (and all) parent components with the relevant event handler callback property.  Also supported is the `pointerEvents:none` property to disable a component from being a target of a pointer event (like CSS pointer-events) - the first parent node without `pointerEvents:none` is then the target identified in `Event.pointerTarget`.  See the demo for more detail on how this works (Gallery handles events for which it's children are the targets, and pointer events are disabled for grandchild nodes).
9.  **src/components/Gallery**: Provides a high performance scrollable and zoomable container for a list of items (each item is composed as a tree of Views).  Unlike React-Canvas ListView, variable sized items are supported and items can be dynamically resized.  Try the demo to see the full range of features.
10.  **Backing Store caching of layers** is supported in a clean, simple and elegant manner.  Simply assign a unique ID to the `useBackingStore` property of the view you wish to cache and a backing store Canvas is created for your view.  Once cached into an ID, you can reuse the same backing store pixel data in any view (or the same view) by referencing the same `useBackingStore` ID.  The view can be optionally scaled to fit the required width & height.  However, any changes to the View props/state once it has been assigned to a backing store ID is not guaranteed be reflected in the backing store - ie: each `useBackingStore` ID is essentially a pure function mapping the ID to a specific View repaint execution.  If you wish to invalidate/update the backing store, simply use a new ID - see the demo for an example of how it works.  Or if you leave out the ID, this will force the View to be repainted without use of a backing store.  Backing stores are cached/discarded in a first-in, first-out manner from a fixed size backing store pool, so you can create and assign as many ID's as you like without any memory impact.  

## Dependencies and Requirements

`React`, `ReactDOM` and Node.js `EventEmitter` are the only runtime requirements.  `Preact` + `Preact-compat` is an alternative to `React` + `ReactDOM`.

Gallery uses Redux style state management - note how the demo code passes a Redux `Store.dispatch` callback to the `onChange` property of Gallery.  However, Redux is not essential and you can provide a non-Redux `onChange` callback to Gallery provided your callback can handle Redux-format actions.

## Running the demo

1. Download this git as a zip file or clone this git locally to your computer.  CD (change directory) into the root directory of this git (where `package.json` and `webpack.config.js` are located)

2. Open `demo.html` in Chrome to see the demo.

## Creating your own code

To create your own code, install EventEmitter (the only internal runtime dependency) and webpack (the only development dependency) by running:

```
npm install
```

```
npm install -g webpack
```

Then use webpack to assemble the React-RETINA modules with your own code (execute code above with the current directory set to the directory with `webpack.config.js`:

```
webpack
```

The existing `webpack.config.js` file can be tailored to suit your needs (default output is to `js/React-RETINA.js`. Change this to your own javascript file).  The other runtime dependencies are provided in the `js/` directory, and are included via `<script>` tags in your HTML file.  Webpack is aware of them via the `externals` settings in the `webpack.config.js` file.


## Contributing

Pull requests for bug fixes, new features, and improvements are very welcome!
