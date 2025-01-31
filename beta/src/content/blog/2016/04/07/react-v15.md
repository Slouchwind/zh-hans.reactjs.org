---
title: 'React v15.0'
author: [gaearon]
---

We would like to thank the React community for reporting issues and regressions in the release candidates on our [issue tracker](https://github.com/facebook/react/issues/). Over the last few weeks we fixed those issues, and now, after two release candidates, we are excited to finally release the stable version of React 15.

As a reminder, [we’re switching to major versions](/blog/2016/02/19/new-versioning-scheme.html) to indicate that we have been using React in production for a long time. This 15.0 release follows our previous 0.14 version and we’ll continue to follow semver like we’ve been doing since 2013. It’s also worth noting that [we no longer actively support Internet Explorer 8](/blog/2016/01/12/discontinuing-ie8-support.html). We believe React will work in its current form there but we will not be prioritizing any efforts to fix new issues that only affect IE8.

React 15 brings significant improvements to how we interact with the DOM:

- We are now using `document.createElement` instead of setting `innerHTML` when mounting components. This allows us to get rid of the `data-reactid` attribute on every node and make the DOM lighter. Using `document.createElement` is also faster in modern browsers and fixes a number of edge cases related to SVG elements and running multiple copies of React on the same page.

- Historically our support for SVG has been incomplete, and many tags and attributes were missing. We heard you, and in React 15 we [added support for all the SVG attributes that are recognized by today’s browsers](https://github.com/facebook/react/pull/6243). If we missed any of the attributes you’d like to use, please [let us know](https://github.com/facebook/react/issues/1657). As a bonus, thanks to using `document.createElement`, we no longer need to maintain a list of SVG tags, so any SVG tags that were previously unsupported should work just fine in React 15.

- We received some amazing contributions from the community in this release, and we would like to highlight [this pull request](https://github.com/facebook/react/pull/5753) by [Michael Wiencek](https://github.com/mwiencek) in particular. Thanks to Michael’s work, React 15 no longer emits extra `<span>` nodes around the text, making the DOM output much cleaner. This was a longstanding annoyance for React users so it’s exciting to accept this as an outside contribution.

While this isn’t directly related to the release, we understand that in order to receive more community contributions like Michael’s, we need to communicate our goals and priorities more openly, and review pull requests more decisively. As a first step towards this, we started publishing [React core team weekly meeting notes](https://github.com/reactjs/core-notes) again. We also intend to introduce an RFC process inspired by [Ember RFCs](https://github.com/emberjs/rfcs) so external contributors can have more insight and influence in the future development of React. We will keep you updated about this on our blog.

We are also experimenting with a new changelog format in this post. Every change now links to the corresponding pull request and mentions the author. Let us know whether you find this useful!

## Upgrade Guide {/*upgrade-guide*/}

As usual with major releases, React 15 will remove support for some of the patterns deprecated nine months ago in React 0.14. We know changes can be painful (the Facebook codebase has over 20,000 React components, and that’s not even counting React Native), so we always try to make changes gradually in order to minimize the pain.

If your code is free of warnings when running under React 0.14, upgrading should be easy. The bulk of changes in this release are actually behind the scenes, impacting the way that React interacts with the DOM. The other substantial change is that React now supports the full range of SVG elements and attributes. Beyond that we have a large number of incremental improvements and additional warnings aimed to aid developers. We’ve also laid some groundwork in the core to bring you some new capabilities in future releases.

See the changelog below for more details.

## Installation {/*installation*/}

We recommend using React from `npm` and using a tool like browserify or webpack to build your code into a single bundle. To install the two packages:

- `npm install --save react react-dom`

Remember that by default, React runs extra checks and provides helpful warnings in development mode. When deploying your app, set the `NODE_ENV` environment variable to `production` to use the production build of React which does not include the development warnings and runs significantly faster.

If you can’t use `npm` yet, we provide pre-built browser builds for your convenience, which are also available in the `react` package on bower.

- **React**  
  Dev build with warnings: https://fb.me/react-15.0.0.js  
  Minified build for production: https://fb.me/react-15.0.0.min.js
- **React with Add-Ons**  
  Dev build with warnings: https://fb.me/react-with-addons-15.0.0.js  
  Minified build for production: https://fb.me/react-with-addons-15.0.0.min.js
- **React DOM** (include React in the page before React DOM)  
  Dev build with warnings: https://fb.me/react-dom-15.0.0.js  
  Minified build for production: https://fb.me/react-dom-15.0.0.min.js

## Changelog {/*changelog*/}

### Major changes {/*major-changes*/}

- #### `document.createElement` is in and `data-reactid` is out

  There were a number of large changes to our interactions with the DOM. One of the most noticeable changes is that we no longer set the `data-reactid` attribute for each DOM node. While this will make it more difficult to know if a website is using React, the advantage is that the DOM is much more lightweight. This change was made possible by us switching to use `document.createElement` on initial render. Previously we would generate a large string of HTML and then set `node.innerHTML`. At the time, this was decided to be faster than using `document.createElement` for the majority of cases and browsers that we supported. Browsers have continued to improve and so overwhelmingly this is no longer true. By using `createElement` we can make other parts of React faster. The ids were used to map back from events to the original React component, meaning we had to do a bunch of work on every event, even though we cached this data heavily. As we’ve all experienced, caching and in particularly invalidating caches, can be error prone and we saw many hard to reproduce issues over the years as a result. Now we can build up a direct mapping at render time since we already have a handle on the node.

  **Note:** `data-reactid` is still present for server-rendered content, however it is much smaller than before and is simply an auto-incrementing counter.

  <small>[@sophiebits](https://github.com/sophiebits) in [#5205](https://github.com/facebook/react/pull/5205)</small>

- #### No more extra `<span>`s

  Another big change with our DOM interaction is how we render text blocks. Previously you may have noticed that React rendered a lot of extra `<span>`s. For example, in our most basic example on the home page we render `<div>Hello {this.props.name}</div>`, resulting in markup that contained 2 `<span>`s. Now we’ll render plain text nodes interspersed with comment nodes that are used for demarcation. This gives us the same ability to update individual pieces of text, without creating extra nested nodes. Very few people have depended on the actual markup generated here so it’s likely you are not impacted. However if you were targeting these `<span>`s in your CSS, you will need to adjust accordingly. You can always render them explicitly in your components.

  <small>[@mwiencek](https://github.com/mwiencek) in [#5753](https://github.com/facebook/react/pull/5753)</small>

- #### Rendering `null` now uses comment nodes

  We’ve also made use of these comment nodes to change what `null` renders to. Rendering to `null` was a feature we added in React 0.11 and was implemented by rendering `<noscript>` elements. By rendering to comment nodes now, there’s a chance some of your CSS will be targeting the wrong thing, specifically if you are making use of `:nth-child` selectors. React’s use of the `<noscript>` tag has always been considered an implementation detail of how React targets the DOM. We believe they are safe changes to make without going through a release with warnings detailing the subtle differences as they are details that should not be depended upon. Additionally, we have seen that these changes have improved React performance for many typical applications.

  <small>[@sophiebits](https://github.com/sophiebits) in [#5451](https://github.com/facebook/react/pull/5451)</small>

- #### Function components can now return `null` too

  We added support for [defining stateless components as functions](/blog/2015/09/10/react-v0.14-rc1.html#stateless-function-components) in React 0.14. However, React 0.14 still allowed you to define a class component without extending `React.Component` or using `React.createClass()`, so [we couldn’t reliably tell if your component is a function or a class](https://github.com/facebook/react/issues/5355), and did not allow returning `null` from it. This issue is solved in React 15, and you can now return `null` from any component, whether it is a class or a function.

  <small>[@jimfb](https://github.com/jimfb) in [#5884](https://github.com/facebook/react/pull/5884)</small>

- #### Improved SVG support

  All SVG tags are now fully supported. (Uncommon SVG tags are not present on the `React.DOM` element helper, but JSX and `React.createElement` work on all tag names.) All SVG attributes that are implemented by the browsers should be supported too. If you find any attributes that we have missed, please [let us know in this issue](https://github.com/facebook/react/issues/1657).

  <small>[@zpao](https://github.com/zpao) in [#6243](https://github.com/facebook/react/pull/6243)</small>

### Breaking changes {/*breaking-changes*/}

- #### No more extra `<span>`s

  It’s worth calling out the DOM structure changes above again, in particular the change from `<span>`s. In the course of updating the Facebook codebase, we found a very small amount of code that was depending on the markup that React generated. Some of these cases were integration tests like WebDriver which were doing very specific XPath queries to target nodes. Others were simply tests using `ReactDOM.renderToStaticMarkup` and comparing markup. Again, there were a very small number of changes that had to be made, but we don’t want anybody to be blindsided. We encourage everybody to run their test suites when upgrading and consider alternative approaches when possible. One approach that will work for some cases is to explicitly use `<span>`s in your `render` method.

  <small>[@mwiencek](https://github.com/mwiencek) in [#5753](https://github.com/facebook/react/pull/5753)</small>

- #### `React.cloneElement()` now resolves `defaultProps`

  We fixed a bug in `React.cloneElement()` that some components may rely on. If some of the `props` received by `cloneElement()` are `undefined`, it used to return an element with `undefined` values for those props. In React 15, we’re changing it to be consistent with `createElement()`. Now any `undefined` props passed to `cloneElement()` are resolved to the corresponding component’s `defaultProps`. Only one of our 20,000 React components was negatively affected by this so we feel comfortable releasing this change without keeping the old behavior for another release cycle.

  <small>[@truongduy134](https://github.com/truongduy134) in [#5997](https://github.com/facebook/react/pull/5997)</small>

- #### `ReactPerf.getLastMeasurements()` is opaque

  This change won’t affect applications but may break some third-party tools. We are [revamping `ReactPerf` implementation](https://github.com/facebook/react/pull/6046) and plan to release it during the 15.x cycle. The internal performance measurement format is subject to change so, for the time being, we consider the return value of `ReactPerf.getLastMeasurements()` an opaque data structure that should not be relied upon.

  <small>[@gaearon](https://github.com/gaearon) in [#6286](https://github.com/facebook/react/pull/6286)</small>

- #### Removed deprecations

  These deprecations were introduced nine months ago in v0.14 with a warning and are removed:

  - Deprecated APIs are removed from the `React` top-level export: `findDOMNode`, `render`, `renderToString`, `renderToStaticMarkup`, and `unmountComponentAtNode`. As a reminder, they are now available on `ReactDOM` and `ReactDOMServer`.  
    <small>[@jimfb](https://github.com/jimfb) in [#5832](https://github.com/facebook/react/pull/5832)</small>

  - Deprecated addons are removed: `batchedUpdates` and `cloneWithProps`.  
    <small>[@jimfb](https://github.com/jimfb) in [#5859](https://github.com/facebook/react/pull/5859), [@zpao](https://github.com/zpao) in [#6016](https://github.com/facebook/react/pull/6016)</small>

  - Deprecated component instance methods are removed: `setProps`, `replaceProps`, and `getDOMNode`.  
    <small>[@jimfb](https://github.com/jimfb) in [#5570](https://github.com/facebook/react/pull/5570)</small>

  - Deprecated CommonJS `react/addons` entry point is removed. As a reminder, you should use separate `react-addons-*` packages instead. This only applies if you use the CommonJS builds.  
    <small>[@gaearon](https://github.com/gaearon) in [#6285](https://github.com/facebook/react/pull/6285)</small>

  - Passing `children` to void elements like `<input>` was deprecated, and now throws an error.  
    <small>[@jonhester](https://github.com/jonhester) in [#3372](https://github.com/facebook/react/pull/3372)</small>

  - React-specific properties on DOM `refs` (e.g. `this.refs.div.props`) were deprecated, and are removed now.  
    <small>[@jimfb](https://github.com/jimfb) in [#5495](https://github.com/facebook/react/pull/5495)</small>

### New deprecations, introduced with a warning {/*new-deprecations-introduced-with-a-warning*/}

Each of these changes will continue to work as before with a new warning until the release of React 16 so you can upgrade your code gradually.

- `LinkedStateMixin` and `valueLink` are now deprecated due to very low popularity. If you need this, you can use a wrapper component that implements the same behavior: [react-linked-input](https://www.npmjs.com/package/react-linked-input).  
  <small>[@jimfb](https://github.com/jimfb) in [#6127](https://github.com/facebook/react/pull/6127)</small>

- Future versions of React will treat `<input value={null}>` as a request to clear the input. However, React 0.14 has been ignoring `value={null}`. React 15 warns you on a `null` input value and offers you to clarify your intention. To fix the warning, you may explicitly pass an empty string to clear a controlled input, or pass `undefined` to make the input uncontrolled.  
  <small>[@antoaravinth](https://github.com/antoaravinth) in [#5048](https://github.com/facebook/react/pull/5048)</small>

- `ReactPerf.printDOM()` was renamed to `ReactPerf.printOperations()`, and `ReactPerf.getMeasurementsSummaryMap()` was renamed to `ReactPerf.getWasted()`.  
  <small>[@gaearon](https://github.com/gaearon) in [#6287](https://github.com/facebook/react/pull/6287)</small>

### New helpful warnings {/*new-helpful-warnings*/}

- If you use a minified copy of the _development_ build, React DOM kindly encourages you to use the faster production build instead.  
  <small>[@sophiebits](https://github.com/sophiebits) in [#5083](https://github.com/facebook/react/pull/5083)</small>

- React DOM: When specifying a unit-less CSS value as a string, a future version will not add `px` automatically. This version now warns in this case (ex: writing `style={{width: '300'}}`. Unitless _number_ values like `width: 300` are unchanged.  
  <small>[@pluma](https://github.com/pluma) in [#5140](https://github.com/facebook/react/pull/5140)</small>

- Synthetic Events will now warn when setting and accessing properties (which will not get cleared appropriately), as well as warn on access after an event has been returned to the pool.  
  <small>[@kentcdodds](https://github.com/kentcdodds) in [#5940](https://github.com/facebook/react/pull/5940) and [@koba04](https://github.com/koba04) in [#5947](https://github.com/facebook/react/pull/5947)</small>

- Elements will now warn when attempting to read `ref` and `key` from the props.  
  <small>[@prometheansacrifice](https://github.com/prometheansacrifice) in [#5744](https://github.com/facebook/react/pull/5744)</small>

- React will now warn if you pass a different `props` object to `super()` in the constructor.  
  <small>[@prometheansacrifice](https://github.com/prometheansacrifice) in [#5346](https://github.com/facebook/react/pull/5346)</small>

- React will now warn if you call `setState()` inside `getChildContext()`.  
  <small>[@raineroviir](https://github.com/raineroviir) in [#6121](https://github.com/facebook/react/pull/6121)</small>

- React DOM now attempts to warn for mistyped event handlers on DOM elements, such as `onclick` which should be `onClick`.  
  <small>[@ali](https://github.com/ali) in [#5361](https://github.com/facebook/react/pull/5361)</small>

- React DOM now warns about `NaN` values in `style`.  
  <small>[@jontewks](https://github.com/jontewks) in [#5811](https://github.com/facebook/react/pull/5811)</small>

- React DOM now warns if you specify both `value` and `defaultValue` for an input.  
  <small>[@mgmcdermott](https://github.com/mgmcdermott) in [#5823](https://github.com/facebook/react/pull/5823)</small>

- React DOM now warns if an input switches between being controlled and uncontrolled.  
  <small>[@TheBlasfem](https://github.com/TheBlasfem) in [#5864](https://github.com/facebook/react/pull/5864)</small>

- React DOM now warns if you specify `onFocusIn` or `onFocusOut` handlers as they are unnecessary in React.  
  <small>[@jontewks](https://github.com/jontewks) in [#6296](https://github.com/facebook/react/pull/6296)</small>

- React now prints a descriptive error message when you pass an invalid callback as the last argument to `ReactDOM.render()`, `this.setState()`, or `this.forceUpdate()`.  
  <small>[@conorhastings](https://github.com/conorhastings) in [#5193](https://github.com/facebook/react/pull/5193) and [@gaearon](https://github.com/gaearon) in [#6310](https://github.com/facebook/react/pull/6310)</small>

- Add-Ons: `TestUtils.Simulate()` now prints a helpful message if you attempt to use it with shallow rendering.  
  <small>[@conorhastings](https://github.com/conorhastings) in [#5358](https://github.com/facebook/react/pull/5358)</small>

- PropTypes: `arrayOf()` and `objectOf()` provide better error messages for invalid arguments.  
  <small>[@chicoxyzzy](https://github.com/chicoxyzzy) in [#5390](https://github.com/facebook/react/pull/5390)</small>

### Notable bug fixes {/*notable-bug-fixes*/}

- Fixed multiple small memory leaks.  
  <small>[@sophiebits](https://github.com/sophiebits) in [#4983](https://github.com/facebook/react/pull/4983) and [@victor-homyakov](https://github.com/victor-homyakov) in [#6309](https://github.com/facebook/react/pull/6309)</small>

- Input events are handled more reliably in IE 10 and IE 11; spurious events no longer fire when using a placeholder.  
  <small>[@jquense](https://github.com/jquense) in [#4051](https://github.com/facebook/react/pull/4051)</small>

- The `componentWillReceiveProps()` lifecycle method is now consistently called when `context` changes.  
  <small>[@milesj](https://github.com/milesj) in [#5787](https://github.com/facebook/react/pull/5787)</small>

- `React.cloneElement()` doesn’t append slash to an existing `key` when used inside `React.Children.map()`.  
  <small>[@ianobermiller](https://github.com/ianobermiller) in [#5892](https://github.com/facebook/react/pull/5892)</small>

- React DOM now supports the `cite` and `profile` HTML attributes.  
  <small>[@AprilArcus](https://github.com/AprilArcus) in [#6094](https://github.com/facebook/react/pull/6094) and [@saiichihashimoto](https://github.com/saiichihashimoto) in [#6032](https://github.com/facebook/react/pull/6032)</small>

- React DOM now supports `cssFloat`, `gridRow` and `gridColumn` CSS properties.  
  <small>[@stevenvachon](https://github.com/stevenvachon) in [#6133](https://github.com/facebook/react/pull/6133) and [@mnordick](https://github.com/mnordick) in [#4779](https://github.com/facebook/react/pull/4779)</small>

- React DOM now correctly handles `borderImageOutset`, `borderImageWidth`, `borderImageSlice`, `floodOpacity`, `strokeDasharray`, and `strokeMiterlimit` as unitless CSS properties.  
  <small>[@rofrischmann](https://github.com/rofrischmann) in [#6210](https://github.com/facebook/react/pull/6210) and [#6270](https://github.com/facebook/react/pull/6270)</small>

- React DOM now supports the `onAnimationStart`, `onAnimationEnd`, `onAnimationIteration`, `onTransitionEnd`, and `onInvalid` events. Support for `onLoad` has been added to `object` elements.  
  <small>[@tomduncalf](https://github.com/tomduncalf) in [#5187](https://github.com/facebook/react/pull/5187), [@milesj](https://github.com/milesj) in [#6005](https://github.com/facebook/react/pull/6005), and [@ara4n](https://github.com/ara4n) in [#5781](https://github.com/facebook/react/pull/5781)</small>

- React DOM now defaults to using DOM attributes instead of properties, which fixes a few edge case bugs. Additionally the nullification of values (ex: `href={null}`) now results in the forceful removal, no longer trying to set to the default value used by browsers in the absence of a value.  
  <small>[@syranide](https://github.com/syranide) in [#1510](https://github.com/facebook/react/pull/1510)</small>

- React DOM does not mistakingly coerce `children` to strings for Web Components.  
  <small>[@jimfb](https://github.com/jimfb) in [#5093](https://github.com/facebook/react/pull/5093)</small>

- React DOM now correctly normalizes SVG `<use>` events.  
  <small>[@edmellum](https://github.com/edmellum) in [#5720](https://github.com/facebook/react/pull/5720)</small>

- React DOM does not throw if a `<select>` is unmounted while its `onChange` handler is executing.  
  <small>[@sambev](https://github.com/sambev) in [#6028](https://github.com/facebook/react/pull/6028)</small>

- React DOM does not throw in Windows 8 apps.  
  <small>[@Andrew8xx8](https://github.com/Andrew8xx8) in [#6063](https://github.com/facebook/react/pull/6063)</small>

- React DOM does not throw when asynchronously unmounting a child with a `ref`.  
  <small>[@yiminghe](https://github.com/yiminghe) in [#6095](https://github.com/facebook/react/pull/6095)</small>

- React DOM no longer forces synchronous layout because of scroll position tracking.  
  <small>[@syranide](https://github.com/syranide) in [#2271](https://github.com/facebook/react/pull/2271)</small>

- `Object.is` is used in a number of places to compare values, which leads to fewer false positives, especially involving `NaN`. In particular, this affects the `shallowCompare` add-on.  
  <small>[@chicoxyzzy](https://github.com/chicoxyzzy) in [#6132](https://github.com/facebook/react/pull/6132)</small>

- Add-Ons: ReactPerf no longer instruments adding or removing an event listener because they don’t really touch the DOM due to event delegation.  
  <small>[@antoaravinth](https://github.com/antoaravinth) in [#5209](https://github.com/facebook/react/pull/5209)</small>

### Other improvements {/*improvements*/}

- React now uses `loose-envify` instead of `envify` so it installs fewer transitive dependencies.  
  <small>[@qerub](https://github.com/qerub) in [#6303](https://github.com/facebook/react/pull/6303)</small>

- Shallow renderer now exposes `getMountedInstance()`.  
  <small>[@glenjamin](https://github.com/glenjamin) in [#4918](https://github.com/facebook/react/pull/4918)</small>

- Shallow renderer now returns the rendered output from `render()`.  
  <small>[@simonewebdesign](https://github.com/simonewebdesign) in [#5411](https://github.com/facebook/react/pull/5411)</small>

- React no longer depends on ES5 _shams_ for `Object.create` and `Object.freeze` in older environments. It still, however, requires ES5 _shims_ in those environments.  
  <small>[@dgreensp](https://github.com/dgreensp) in [#4959](https://github.com/facebook/react/pull/4959)</small>

- React DOM now allows `data-` attributes with names that start with numbers.  
  <small>[@nLight](https://github.com/nLight) in [#5216](https://github.com/facebook/react/pull/5216)</small>

- React DOM adds a new `suppressContentEditableWarning` prop for components like [Draft.js](https://facebook.github.io/draft-js/) that intentionally manage `contentEditable` children with React.  
  <small>[@mxstbr](https://github.com/mxstbr) in [#6112](https://github.com/facebook/react/pull/6112)</small>

- React improves the performance for `createClass()` on complex specs.  
  <small>[@sophiebits](https://github.com/sophiebits) in [#5550](https://github.com/facebook/react/pull/5550)</small>
