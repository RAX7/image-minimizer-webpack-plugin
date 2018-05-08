# imagemin-webpack

[![NPM version](https://img.shields.io/npm/v/imagemin-webpack.svg)](https://www.npmjs.org/package/imagemin-webpack)
[![Travis Build Status](https://img.shields.io/travis/itgalaxy/imagemin-webpack/master.svg?label=build)](https://travis-ci.org/itgalaxy/imagemin-webpack)
[![dependencies Status](https://david-dm.org/itgalaxy/imagemin-webpack/status.svg)](https://david-dm.org/itgalaxy/imagemin-webpack)
[![devDependencies Status](https://david-dm.org/itgalaxy/imagemin-webpack/dev-status.svg)](https://david-dm.org/itgalaxy/imagemin-webpack?type=dev)
[![peerDependencies Status](https://david-dm.org/itgalaxy/imagemin-webpack/peer-status.svg)](https://david-dm.org/itgalaxy/imagemin-webpack?type=peer)
[![Greenkeeper badge](https://badges.greenkeeper.io/itgalaxy/imagemin-webpack.svg)](https://greenkeeper.io)

<!--lint disable no-html-->

<div align="center">
  <a href="https://github.com/webpack/webpack">
    <img width="200" height="200" hspace="10"
      src="https://cdn.rawgit.com/webpack/media/e7485eb2/logo/icon.svg">
  </a>
  <h1>Imagemin Webpack</h1>
  <p>
    Plugin and Loader for <a href="http://webpack.js.org/">webpack</a> to optimize (compress) all images using <a href="https://github.com/imagemin/imagemin">imagemin</a>.
    Do not worry about size of images, now they are always compressed.
  </p>
</div>

<!--lint enable no-html-->

## Why

* No extra dependencies (`imagemin-gifsicle`, `imagemin-pngquant`) in `dependencies` section into `package.json`.
  You decide for yourself what plugins to use.

* This loader and plugin will optimize ANY images regardless of how they were added to webpack.
  `image-webpack-loader` don't optimize some images generating `favicons-webpack-plugin` or `copy-webpack-plugin`.
  `ImageminWebpackPlugin` don't optimize inlined images with `url-loader`.

* Images optimized when inlined with `url-loader` or `svg-url-loader`. This can't be done with `imagemin-webpack-plugin`.

* Throttle asynchronous images optimization (using `maxConcurrency` plugin option).
  This allows you to not overload a server when building.

* All tested.

* Persistent cache.

* (Optional) Don't crash building process if your have corrupted image(s).

## Install

```shell
npm install imagemin-webpack --save-dev
```

### Optionals

Images can be compressed in two modes:

1.  [Lossless](https://en.wikipedia.org/wiki/Lossless_compression) (without loss of quality).
2.  [Lossy](https://en.wikipedia.org/wiki/Lossy_compression) (with loss of quality).

Note:

* [imagemin-mozjpeg](https://github.com/imagemin/imagemin-mozjpeg) can be configured in lossless and lossy mode.
* [imagemin-svgo](https://github.com/imagemin/imagemin-svgo) can be configured in lossless and lossy mode.

Explore the options to get the best result for you.

**Recommended basic imagemin plugins for lossless optimization**

```shell
npm install imagemin-gifsicle imagemin-jpegtran imagemin-optipng imagemin-svgo --save-dev
```

**Recommended basic imagemin plugins for lossy optimization**

```shell
npm install imagemin-gifsicle imagemin-mozjpeg imagemin-pngquant imagemin-svgo --save-dev
```

## Usage

### Basic

If you want to use `loader` or `plugin` standalone see sections below, but this is not recommended.

**Make sure that plugin place after any plugins that add images or other assets which you want to optimized.**

```js
const { ImageminWebpackPlugin } = require("imagemin-webpack");

// Before importing imagemin plugin make sure you add it in `package.json` (`dependencies`) and install
const imageminGifsicle = require("imagemin-gifsicle");

const plugins = [imageminGifsicle()];

module.exports = {
  module: {
    rules: [
      {
        test: /\.(jpe?g|png|gif|svg)$/i,
        use: [
          {
            loader: "file-loader" // Or `url-loader` or your other loader
          }
        ]
      }
    ]
  },
  plugins: [
    // Make sure that the plugin is after any plugins that add images
    new ImageminWebpackPlugin({
      imageminOptions: {
        plugins
      }
    })
  ]
};
```

### Standalone Loader

[Documentation: Using loaders](https://webpack.js.org/concepts/loaders/)

In your `webpack.config.js`, add the `imagemin-webpack`,
chained with the [file-loader](https://github.com/webpack/file-loader)
or [url-loader](https://github.com/webpack-contrib/url-loader):

```js
const { imageminLoader } = require("imagemin-webpack");

// Before importing imagemin plugin make sure you add it in `package.json` (`dependencies`) and install
const imageminGifsicle = require("imagemin-gifsicle");

module.exports = {
  module: {
    rules: [
      {
        test: /\.(jpe?g|png|gif|svg)$/i,
        use: [
          {
            loader: "file-loader" // Or `url-loader` or your other loader
          },
          {
            loader: imageminLoader,
            options: {
              cache: true,
              bail: false, // Ignore errors on corrupted images
              imageminOptions: {
                plugins: [imageminGifsicle()]
              }
            }
          }
        ]
      }
    ]
  }
};
```

### Standalone Plugin

[Documentation: Using plugins](https://webpack.js.org/concepts/plugins/)

```js
import { ImageminWebpackPlugin } from "imagemin-webpack";

// Before importing imagemin plugin make sure you add it in `package.json` (`dependencies`) and install
import imageminGifsicle from "imagemin-gifsicle";

const imageminManifest = {};

export default {
  //
  module: {
    rules: [
      {
        loader: "file-loader",
        options: {
          emitFile: true, // Don't forget emit images
          name: "[path][name].[ext]"
        },
        test: /\.(jpe?g|png|gif|svg)$/i
      }
    ]
  },
  plugins: [
    // Make sure that the plugin is after any plugins that add images
    new ImageminWebpackPlugin({
      cache: true,
      bail: false, // Ignore errors on corrupted images
      loader: false,
      imageminOptions: {
        plugins: [imageminGifsicle()]
      },
      manifest: imageminManifest, // This object will contain source and interpolated filenames
      maxConcurrency: os.cpus().length - 1,
      name: "[hash].[ext]",
      test: /\.(jpe?g|png|gif|svg)$/i,
      include: undefined,
      exclude: undefined
    })
  ]
};
```

## Options

### Loader Options

|         Name          |        Type         |         Default         | Description                  |
| :-------------------: | :-----------------: | :---------------------: | :--------------------------- |
|      **`cache`**      | `{Boolean\|String}` |         `false`         | Enable file caching          |
|      **`bail`**       |     `{Boolean}`     | `compiler.options.bail` | Emit warnings instead errors |
| **`imageminOptions`** |     `{Object}`      |    `{ plugins: [] }`    | Options for `imagemin`       |

#### `cache`

##### `{Boolean}`

**webpack.config.js**

```js
[
  new ImageminWebpackPlugin({
    cache: true
  })
];
```

Enable file caching.
Default path to cache directory: `node_modules/.cache/uglifyjs-webpack-plugin`.

##### `{String}`

**webpack.config.js**

```js
[
  new ImageminWebpackPlugin({
    cache: "path/to/cache"
  })
];
```

#### `bail`

**webpack.config.js**

```js
[
  new ImageminWebpackPlugin({
    bail: true
  })
];
```

#### `imageminOptions`

**webpack.config.js**

```js
const imageminGifsicle = require("imagemin-gifsicle");

[
  new ImageminWebpackPlugin({
    imageminOptions: {
      plugins: [
        imageminGifsicle({
          interlaced: true,
          optimizationLevel: 3
        })
      ]
    }
  })
];
```

### Plugin Options

<!--lint disable no-html-->

|         Name          |           Type            |                  Default                  | Description                                                                                                               |
| :-------------------: | :-----------------------: | :---------------------------------------: | :------------------------------------------------------------------------------------------------------------------------ |
|      **`test`**       | `{RegExp\|Array<RegExp>}` | <code>/\.(jpe?g\|png\|gif\|svg)$/i</code> | Test to match files against                                                                                               | gif | svg)$/i</code> | Test to match files against | gif | svg)$/i</code> | Test to match files against |
|     **`include`**     | `{RegExp\|Array<RegExp>}` |                `undefined`                | Files to `include`                                                                                                        |
|     **`exclude`**     | `{RegExp\|Array<RegExp>}` |                `undefined`                | Files to `exclude`                                                                                                        |
|      **`cache`**      |    `{Boolean\|String}`    |                  `false`                  | Enable file caching                                                                                                       |
|      **`bail`**       |        `{Boolean}`        |          `compiler.options.bail`          | Emit warnings instead errors                                                                                              |
| **`imageminOptions`** |        `{Object}`         |             `{ plugins: [] }`             | Options for `imagemin`                                                                                                    |
|     **`loader`**      |        `{Boolean}`        |                  `true`                   | Automatically adding `imagemin-loader` (require for minification images using in `url-loader`, `svg-url-loader` or other) |
| **`maxConcurrency`**  |        `{Number}`         |          `os.cpus().length - 1`           | Maximum number of concurrency minification processes in one time                                                          |
|      **`name`**       |        `{Number}`         |              `[hash].[ext]`               | The target asset name                                                                                                     |
|    **`manifest`**     |        `{Number}`         |              `[hash].[ext]`               | Contain optimized list of images from other plugins                                                                       |

<!--lint enable no-html-->

#### `test`

**webpack.config.js**

```js
[
  new ImageminWebpackPlugin({
    test: /\.(jpe?g|png|gif|svg)$/i
  })
];
```

#### `include`

**webpack.config.js**

```js
[
  new ImageminWebpackPlugin({
    include: /\/includes/
  })
];
```

#### `exclude`

**webpack.config.js**

```js
[
  new ImageminWebpackPlugin({
    exclude: /\/excludes/
  })
];
```

#### `cache`

**Be careful** when your enable `cache` and change options for imagemin plugin (example for `imagemin-gifsicle`) you should remove cache manually.
You can use `rm -rf ./node_modules/.cache/imagemin-webpack` command. This is due to the fact that `imagemin-plugin` is `Function` and we don't know her arguments to invalidate cache.
Note: if somebody know how we can fix it PR welcome!

##### `{Boolean}`

**webpack.config.js**

```js
[
  new ImageminWebpackPlugin({
    cache: true
  })
];
```

Enable file caching.
Default path to cache directory: `node_modules/.cache/uglifyjs-webpack-plugin`.

##### `{String}`

**webpack.config.js**

```js
[
  new ImageminWebpackPlugin({
    cache: "path/to/cache"
  })
];
```

#### `bail`

**webpack.config.js**

```js
[
  new ImageminWebpackPlugin({
    bail: true
  })
];
```

#### `imageminOptions`

**webpack.config.js**

```js
const imageminGifsicle = require("imagemin-gifsicle");

[
  new ImageminWebpackPlugin({
    imageminOptions: {
      plugins: [
        imageminGifsicle({
          interlaced: true,
          optimizationLevel: 3
        })
      ]
    }
  })
];
```

#### `loader`

**webpack.config.js**

```js
const imageminGifsicle = require("imagemin-gifsicle");

[
  new ImageminWebpackPlugin({
    loader: false
  })
];
```

#### `maxConcurrency`

**webpack.config.js**

```js
const imageminGifsicle = require("imagemin-gifsicle");

[
  new ImageminWebpackPlugin({
    maxConcurrency: 3
  })
];
```

#### `name`

**webpack.config.js**

```js
[
  new ImageminWebpackPlugin({
    name: "[hash]-compressed.[ext]"
  })
];
```

#### `manifest`

**Manifest will be contain list of optimized images only after `emit` event.**

**webpack.config.js**

```js
const manifest = {};

[
  new ImageminWebpackPlugin({
    manifest
  }),
  new OtherPlugin({
    manifest
  })
];
```

## Related

* [imagemin](https://github.com/imagemin/imagemin) - API for this package.
* [image-webpack-loader](https://github.com/tcoopman/image-webpack-loader) - inspiration, thanks.
* [imagemin-webpack-plugin](https://github.com/Klathmon/imagemin-webpack-plugin) - inspiration, thanks.

## Contribution

Feel free to push your code if you agree with publishing under the MIT license.

## [Changelog](CHANGELOG.md)

## [License](LICENSE)
