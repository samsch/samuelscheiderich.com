---
title: How to Setup Webpack v4
date: 2018-09-02 00:00:00
tags:
- React
- Webpack
- developer experience
- js
---
> These docs are written for Webpack version 4, and Babel version 7.

If you write JavaScript code you should be using a module system. For the browser, this means bundling, since built-in module support isn't yet ready to be used in production.

> Bundler: A tool which compiles a module-based codebase into a single (or a few) large file to be linked from your HTML.

The industry standard tool for bundling is Webpack. In the past, Webpack had a bad rep for being difficult to configure because it's documentation was lacking, and because new users were often shown bloated examples including features they might never use. Since then, the documentation has dramatically improved. In this guide, we'll focus on what you need to get started!

> The [Webpack documentation](https://webpack.js.org/concepts/) is really quite good, but it can be hard to filter to just what you need. This guide is meant to be the minimal parts you really need for development and production.

# Prerequisites

Primarily, you need to have Node.js installed. This guide assumes you are using current Node v10+ and npm 6+.

For Linux and MacOS, the easiest way to install and work with Node is using nvm. Instructions for installing are on the [nvm project page](https://github.com/creationix/nvm#installation).

> If using Windows, you can download an installer from the [Node.js website](https://nodejs.org/en/).
>
> The instructions are the same for Webpack in Windows, except you might use backslashes instead of slashes for paths on the command line. For example, `webpack src/main.js public/bundle.js` might be `webpack src\main.js public\bundle.js` in Windows.

# Create a project

First, create a project folder (such as `myproject`). In the project folder open a terminal and run `npm init`, and answer the questions. (You can also run `npm init -y` to skip all the questions and use defaults.)

> From here on out, all command line snippets will assume you are currently in the project folder root.

Add Webpack to your development dependencies: `npm install --save-dev webpack webpack-cli`.

> From here on in this guide, we'll use the shorthand `npm` commands like `npm i` (`npm install`) and `npm i -D` (`install --save-dev`).

Create a folder in your project called `src`.

Inside `src/`, add two JavaScript files:

`src/index.js`:
```js
import foo from './foo';

console.log('Running in index.js!');

foo();
```

`src/foo.js`:
```js
import camelCase from 'camelcase';

function foo () {
  console.log(camelCase('Running in foo.js!'));
}

export default foo;
```

And do `npm i camelcase`.

> The `camelcase` module is being used here as an example to show that you can import and bundle npm modules.

> There are things missing that you should have in a "real" project, like linting. Feel free to add them, but they are out of scope for this article to cover.

# Bundling

Webpack has some defaults which allow you to do very basic bundling without any configuration. It will try to use `src/index.js` as an *entrypoint*, and it will output the bundle(s) to `dist/`.

> *Entrypoints* are files you directly link to with script tags. Webpack can bundle multiple entrypoints in a single build, and can share common dependencies between them if configured to do so. Each entrypoint will have it own output bundle file.

We can just run `npx webpack` and it will bundle our files into `dist/main.js`. While not necessary for Webpack, we'll create an `index.html` file and start a static server.

Add `dist/index.html`
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
</head>
<body>
  Check the console! (F12 to open dev tools)
  <script src="/main.js"></script>
</body>
</html>
```

> You can start a basic file server with `npx live-server ./dist` or `npx http-server -c-1 ./dist`, or setup your own server to serve the files in `dist/`. (Open the browser console to view the output.)
>
> [live-server](https://www.npmjs.com/package/live-server) automatically reloads the page for any file changes, while [http-server](https://www.npmjs.com/package/http-server) does not. We use `-c-1` with `http-server` to disable it's aggressive caching. Both of these servers are only suitable for development and testing, and should not be used in production.
>
> Run the server in a separate console so you don't need to stop and restart it to run Webpack. `npx` will re-download `live-server` or `http-server` every time, so to avoid this slowdown when running it, you can install it to the project with `npm i -D <package>`, or install it globally `npm i -g <package>`. After that, `npx` will use the previously installed module.

## Build modes

When you ran `npx webpack`, it printed a warning about missing the `mode` option. Let's do it correctly by running `npx webpack --mode development`. Without a specific `mode` set, Webpack defaults to "production", which mostly means that it *uglifies* the output.

> *Uglifying* is basically minification, and it makes the file as small as possible by removing unnecessary whitespace and replacing variable names with shorter versions where safe. It can also remove static conditions like `if (false === false) { ... }`.

After running in development mode, take a look at `dist/main.js`. There is a lot of code in this file! Webpack does include some overhead to properly handle modules, but there is also inline sourcemaps of the code, which can be used by the browser to show you the original code files when debugging.

Finally, to build in production mode without a warning, run `npx webpack --mode production`. This will output the same code as `npx webpack`, but without the warning.

## Using npm scripts

You can use Webpack with `npx` like we have so far, but you can make it simpler to keep the correct build modes straight by setting up npm scripts for each mode.

In `package.json`, replace this:
```json
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
```
with this:
```json
  "scripts": {
    "dev": "webpack --mode development",
    "build": "webpack --mode production",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
```

Now you can run `npm run dev` for development mode, and `npm run build` for production mode. This exact build command is quite suitable for real projects, but you will probably want some other features for development.

# Configuration

In most cases, you will end up needing to configure some options for Webpack. A lot of options can be provided as command line arguments, but the much cleaner standard approach is to use a Webpack config file.

Webpack will automatically look for a file called `webpack.config.js` in the folder it is being run in (generally the project root), or you can tell it exactly the file to use with the `--config` command line argument.

Create `webpack.config.js` with these contents:
```js
const path = require('path');

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'main.js',
    path: path.resolve(__dirname, 'dist')
  }
};
```

> This is a regular Node.js module, using require() to load modules.

This configuration says to use the same entrypoint and output location and name as the Webpack defaults. You can change these options now to be whatever you want. If your project needs to use `public` as the folder for static assets, then you could change `dist` to `public`, or even use a subfolder.

> There are many options for both [entry](https://webpack.js.org/configuration/entry-context/) and [output](https://webpack.js.org/configuration/output/) in the Webpack docs.

# Automatically building

In most circumstances, you want to build automatically when you save a file in your project which is being bundled while developing. There are two ways to do this, and which one you want depends on how you want to serve your files in development.

> In Linux, the default file system max listener count is often too low to be able to watch the files in a project. This will cause an error like `Error: ENOSPC` in the console, which appears to suggest you are out of disk space. To fix this, you need to increase the max listener count with this command: `echo fs.inotify.max_user_watches=524288 | sudo tee -a /etc/sysctl.conf && sudo sysctl -p`. [More Info](https://stackoverflow.com/questions/22475849/node-js-error-enospc).

## Watch mode (your own server)

If you already have a server setup and can have it serve the files in the `dist/` folder, then you can simply use Webpack's built-in watch mode.

To do this, change your `"dev"` script in `package.json` to:
```json
    "dev": "webpack --mode development --watch",
```

When you run this, instead of just building and exiting, Webpack continues to run, and any time you make a change and save a file that is being bundled, the bundle will get rebuilt.

If your static server doesn't automatically refresh the page when files change (such as `http-server`), you will need to refresh the page after you save your files.

> You can setup [LiveReload plugin](https://www.npmjs.com/package/webpack-livereload-plugin) which can refresh the page for you when your bundle is rebuilt. You should make sure you only do this for development. [Setting up development-only Webpack config]().

## Dev Server (use webpack-dev-server)

If you don't have a local file server to serve `dist/` with, you can use webpack-dev-server, which combines Webpack with a small file server that has useful development features.

Install webpack-dev-server with `npm i -D webpack-dev-server`.

Add a `devServer` property to your Webpack config as below:
```js
const path = require('path');

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'main.js',
    path: path.resolve(__dirname, 'dist')
  },
  devServer: {
    contentBase: path.join(__dirname, 'dist'),
  },
};
```

Replace your `"dev"` script in `package.json` with:
```json
    "dev": "webpack-dev-server --mode development --open",
```

> If you are still running a static server like `http-server` or `live-server`, you can stop that process since webpack-dev-server includes that functionality.

Now when you run `npm run dev`, webpack-dev-server will start a file server with the contents of `dist/`, Webpack will run, and automatically run again any time a file that is bundled is saved. Additionally, webpack-dev-server includes some extra code which causes the browser to refresh any time the code rebuilds. With the `--open` option included, it will also open your browser to show the index.html file.

> [webpack-dev-server configuration options](https://webpack.js.org/configuration/dev-server/) include a proxy option so that all requests which don't match a bundle file are sent to another server.

# More Webpack config

Webpack has two primary ways to extend it's functionality. It supports a rich plugin ecosystem, which can provide features such as automatically injecting the paths to bundle into a generated html file, or displaying in-depth statistics about bundles. It also supports *loaders*, which are tools that run against imported files, and can run transforms on the code, or even move the original file to the output folder and make the imported value a path to that file.

The number one use-case for loaders is to run [Babel](https://babeljs.io) transforms against your JavaScript code. Babel transforms can replace JSX code with JavaScript, remove Typescript type annotations, or most commonly: replace modern syntax with older syntax to support more browsers.

In most cases, you will want to run Babel against your code with the ["env" preset](https://babeljs.io/docs/en/babel-preset-env), which compiles current JavaScript to older JavaScript.

## Setup Babel and babel-loader

First, lets modify our entrypoint file to use a new syntax feature, so that we can see that it gets replaced.

Change `src/index.js` to:
```js
import '@babel/polyfill';
import foo from './foo';

const obj1 = { a: '1' };
const obj2 = { ...obj1, b: '1' };
console.log('obj2.a === obj2.b', obj2.a === obj2.b);

foo();
```

> The `...` operator is part of ES2018, and needs to be compiled for Edge, and non-latest Safari.

Install Babel and babel-loader with `npm i -D babel-loader @babel/core @babel/preset-env `. Then install Babel Polyfill with `npm i @babel/polyfill`.

We need to add babel-loader into our Webpack config:
```js
const path = require('path');

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'main.js',
    path: path.resolve(__dirname, 'dist')
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
        },
      },
    ],
  },
};
```

Then we need to setup a Babel config in `babel.config.js`:
```js
module.exports = {
  presets: [
    ['@babel/preset-env', {
      useBuiltIns: 'entry',
      modules: false,
      targets: [
        'last 2 firefox versions',
        'last 2 chrome versions',
        'last 2 edge versions',
        'last 2 ios versions',
      ],
    }]
  ],
};
```

> This is an optimistic set of browsers, often you will need to support IE 11 as well.

Now when you build your project, any modern syntax is compiled to code that all browsers in your support list can run. This is specifically only for files in your project that are not in `node_modules` (this is what the `exclude` line in the babel-loader config does).

Syntax isn't the only thing older browsers might not support though, so in addition to having Babel transform syntax, we also need to include Polyfills for newer standard library features. This is the `import '@babel/polyfill';` in `index.js`. With the `useBuiltIns: 'entry'` option in the babel env config, that single polyfill import is transformed to only import the polyfills needed for the browser support list.

> Currently there is an experimental option `useBuiltIns: 'usage'` which will only include polyfills for features that you actually use. In the future this could become the best practice.

# Overview

What we have so far is a reasonable set of tools for building JavaScript browser applications.

With `npm run build` we transform a tree of source code modules from using modern JavaScript to being a single bundle file of broadly supported older JavaScript which we can include in our html.

# Onward and Upward

Thus far, our build configuration is fairly simple, and is focused only on JavaScript and building a single application.

From the basic configuration created here, we can expand to support a lot of helpful features:

- Code splitting (Separate a tree of your application).
- Importing stylesheets and other static assets, to be included in the output folder.
- Transforming non-JS code into JS (such as JSX, Typescript, Flow, and JavaScript proposal features).
- Create a map (manifest) of output files for programmatic consumption.
- Development only configuration options.

To explore these topics and more, head to the [Getting started with Webpack landing page](https://github.com/samsch/webpack-guide/blob/master/README.md).
