# webpack-blocks

[![Build Status](https://travis-ci.org/andywer/webpack-blocks.svg?branch=master)](https://travis-ci.org/andywer/webpack-blocks)
[![JavaScript Style Guide](https://img.shields.io/badge/code%20style-standard-brightgreen.svg)](http://standardjs.com/)
[![Gitter chat](https://badges.gitter.im/webpack-blocks.svg)](https://gitter.im/webpack-blocks)

Functional building blocks for your webpack config: easier way to configure webpack and to share configuration between projects.

Ready to use blocks to configure popular tools like *Babel*, *PostCSS*, *Sass*, *TypeScript*, etc., as well as best practices like extracting CSS — all with just one line of configuration.

>"Finally, webpack config done right. (...) Webpack clearly wants to stay low-level. So it makes total sense to outsource configuring it to well designed blocks instead of copy-paste."
>
>[Dan Abramov](https://github.com/gaearon) via [twitter](https://twitter.com/dan_abramov/status/806249934399881216) (Co-author of Redux, Create React App and React Hot Loader)


## Table of contents

<!-- To update run: npx markdown-toc --maxdepth 2 -i README.md -->

<!-- toc -->

- [Installation](#installation)
- [Upgrade from v0.4](#upgrade-from-v04)
- [Example](#example)
- [More examples](#more-examples)
- [Custom blocks](#custom-blocks)
- [Available webpack blocks](#available-webpack-blocks)
- [Helpers](#helpers)
- [Shorthand setters](#shorthand-setters)
- [Third-party blocks](#third-party-blocks)
- [Design principles](#design-principles)
- [FAQ](#faq)
- [Like what you see?](#like-what-you-see)
- [License](#license)

<!-- tocstop -->

## Installation

```sh
npm install --save-dev webpack webpack-blocks
# or
yarn add --dev webpack webpack-blocks
```


## Upgrade from v0.4

Check out our [migration guide](./docs/MIGRATION-GUIDE.md) to get started with v1.0 today.


## Example

The following sample shows how to create a webpack config with Babel support, dev server and Autoprefixer.

```js
const webpack = require('webpack')
const {
  createConfig,
  match,

  // Feature blocks
  babel,
  css,
  devServer,
  file,
  postcss,
  uglify,

  // Shorthand setters
  addPlugins,
  defineConstants,
  entryPoint,
  env,
  setOutput,
  sourceMaps
} = require('webpack-blocks')
const autoprefixer = require('autoprefixer')
const path = require('path')

module.exports = createConfig([
  entryPoint('./src/main.js'),
  setOutput('./build/bundle.js'),
  babel(),
  match('*.css', { exclude: path.resolve('node_modules') }, [
    css(),
    postcss([
      autoprefixer({ browsers: ['last 2 versions'] })
    ])
  ]),
  match(['*.gif', '*.jpg', '*.jpeg', '*.png', '*.webp'], [
    file()
  ]),
  defineConstants({
    'process.env.NODE_ENV': process.env.NODE_ENV
  }),
  env('development', [
    devServer(),
    devServer.proxy({
      '/api': { target: 'http://localhost:3000' }
    }),
    sourceMaps()
  ]),
  env('production', [
    uglify(),
    addPlugins([
      new webpack.LoaderOptionsPlugin({ minimize: true })
    ])
  ])
])
```

See shorthand setters and helpers [documentation](packages/webpack#exports).

All blocks, like `babel` or `postcss` are also available as their own [small packages](./packages), `webpack-blocks` package wraps these blocks, shorthand setters and helpers as a single dependency for convenience.

## More examples

CSS modules:

```js
const { createConfig, match, css } = require('webpack-blocks')

// ...

module.exports = createConfig([
  // ...
  match('*.css', { exclude: path.resolve('node_modules') }, [
    css.modules()
  ]
])
```

TypeScript:

```js
const { createConfig } = require('webpack-blocks')
const typescript = require('@webpack-blocks/typescript')

// ...

module.exports = createConfig([
  // ...
  typescript()
])
```

## Custom blocks

Need a custom block? A simple block looks like this:

```js
module.exports = createConfig([
  // ...
  myCssLoader([ './styles' ])
])

function myCssLoader () {
  return (context, { merge }) => merge({
    module: {
      rules: [
        Object.assign(
          {
            test: /\.css$/,
            use: [ 'style-loader', 'my-css-loader' ]
          },
          context.match     // carries `test`, `exclude` & `include` as set by `match()`
        )
      ]
    }
  })
}
```

If we use `myCssLoader` in `match()` then `context.match` will be populated with whatever we set in `match()`. Otherwise there is still the `test: /\.css$/` fallback, so our block will work without `match()` as well.

Check out the [sample app](./test-app) to see a webpack config in action or read [how to create your own blocks](./docs/BLOCK-CREATION.md).


## Available webpack blocks

- [assets](./packages/assets)
- [babel](./packages/babel)
- [dev-server](./packages/dev-server)
- [elm](./packages/elm)
- [extract-text](./packages/extract-text)
- [postcss](./packages/postcss)
- [sass](./packages/sass)
- [tslint](./packages/tslint)
- [typescript](./packages/typescript)
- [uglify](./packages/uglify)

## [Helpers](./packages/webpack#helpers)

Helpers allow you to structure your config and define settings for particular environments (like `production` or `development`) or file types.

- group
- env
- match

## [Shorthand setters](./packages/webpack#shorthand-setters)

Shorthand setters gives you easier access to common webpack settings, like plugins, entry points and source maps.

- addPlugins
- customConfig
- defineConstants
- entryPoint
- performance
- resolve
- setContext
- setDevTool
- setOutput
- sourceMaps

## Third-party blocks

- [webpack-blocks-happypack](https://github.com/diegohaz/webpack-blocks-happypack) — HappyPack
- [webpack-blocks-less](https://github.com/kirill-konshin/webpack-blocks-less) — Less
- [webpack-blocks-purescript](https://github.com/ecliptic/webpack-blocks-purescript) — PureScript
- [webpack-blocks-server-source-map](https://github.com/diegohaz/webpack-blocks-server-source-map) — source map for server bundle
- [webpack-blocks-split-vendor](https://github.com/diegohaz/webpack-blocks-split-vendor) — vendor bundle
- [webpack-blocks-ts](https://github.com/foxbunny/webpack-blocks-ts) — TypeScript using ts-loader instead of awesome-typescript-loader
- [webpack-blocks-vue](https://github.com/foxbunny/webpack-blocks-vue) — Vue

Missing something? Write and publish your own webpack blocks!


## Design principles

- Extensibility first
- Uniformity for easy composition
- Keep everything configurable
- But provide sane defaults

## FAQ

<details>
<summary>How to debug?</summary>

In case the webpack configuration does not work as expected you can debug it using [stringify-object](https://www.npmjs.com/package/stringify-object):

```js
const stringify = require('stringify-object')

module.exports = createConfig([
  // ...
])

console.log(stringify(module.exports))
```
</details>

<details>
<summary>How does env() work?</summary>

`env('development', [ ... ])` checks the `NODE_ENV` environment variable and only applies its contained webpack blocks if it matches the given string.

So make sure you set the NODE_ENV accordingly:

```js
// your package.json
"scripts": {
  "build": "cross-env NODE_ENV=production webpack",
  "start": "cross-env NODE_ENV=development webpack-dev-server"
}
```

If there is no NODE_ENV set then it will treat NODE_ENV as if it was `development`. Use [cross-env](https://github.com/kentcdodds/cross-env) to make it work on all platforms.
</details>

<details>
<summary>What does defineConstants() do?</summary>

`defineConstants()` is a small convenience wrapper around webpack's [DefinePlugin](https://webpack.github.io/docs/list-of-plugins.html#defineplugin). It is composable and automatically encodes the values. Use it to replace constants in your code by their values at build time.

So having a `defineConstants({ 'process.env.FOO': 'foo' })` and a `defineConstants({ 'process.env.BAR': 'bar' })` in your config means the resulting webpack config will contain a single `new webpack.DefinePlugin({ 'process.env.FOO': '"FOO"', 'process.env.BAR': '"BAR"' })`, thus replacing any occurrence of `process.env.FOO` and `process.env.BAR` with the given values.
</details>

<details>
<summary>What does a block look like from the inside?</summary>

A webpack block is *a function and requires no dependencies at all* (🎉🎉), thus making it easy to write your own blocks and share them with your team or the community.

Take the `babel` webpack block for instance:

```js
/**
 * @param {object} [options]
 * @param {RegExp|Function|string}  [options.exclude]   Directories to exclude.
 * @return {Function}
 */
function babel (options = { cacheDirectory: true }) {
  return (context, util) => util.addLoader(
    Object.assign({
      // we use a `MIME type => RegExp` abstraction here in order to have consistent regexs
      test: /\.(js|jsx)$/,
      exclude: /node_modules/,
      use: [
        { loader: 'babel-loader', options }
      ]
    }, context.match)
  )
}
```

Add a README and a package.json and you are ready to ship.

For more details see [How to write a block](./docs/BLOCK-CREATION.md).
</details>

<details>
<summary>I need some custom webpack config snippet!</summary>

No problem. If you don't want to write your own webpack block you can use `customConfig()`:

```js
const path = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const { addPlugins, customConfig } = require('@webpack-blocks/webpack')

// ...

module.exports = createConfig([
  // ...
  addPlugins([
    // Add a custom webpack plugin
    new HtmlWebpackPlugin({
      inject: true,
      template: './index.html'
    })
  ]),
  customConfig({
    // Add some custom webpack config snippet
    resolve: {
      extensions: [ '.js', '.es6' ]
    }
  })
])
```

The object you pass to `customConfig()` will be merged into the webpack config using
[webpack-merge](https://github.com/survivejs/webpack-merge) like any other webpack
block's partial config.
</details>

<details>
<summary>How to compose blocks?</summary>

Got some projects with similar, yet not identical webpack configurations? Create a “preset”, a function that returns a `group` of blocks so you can reuse it in multiple projects:

```js
const { createConfig, env, group, babel, devServer } = require('webpack-blocks')

function myPreset (proxyConfig) {
  return group([
    babel(),
    env('development', [
      devServer(),
      devServer.proxy(proxyConfig)
    ])
  ])
}

module.exports = createConfig([
  myPreset({
    '/api': { target: 'http://localhost:3000' }
  }),
  // add more blocks here
])
```

The key feature is the `group()` method which takes a set of blocks and returns a new block that combines all their functionality.
</details>


## Like what you see?

Support webpack-blocks by giving [feedback](https://github.com/andywer/webpack-blocks/issues), publishing new webpack blocks or just by 🌟 starring the project!


## License

MIT
