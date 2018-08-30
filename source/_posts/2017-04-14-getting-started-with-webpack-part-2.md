---
title: Getting Started with Webpack - Part 2
date: 2017-04-14 10:45:02
tags:
- React
- Webpack
- developer experience
- js
---
In [Part 1](https://samsch.org/2017/04/08/getting-started-with-webpack/), we setup basic production ready functionality in Webpack. We can organize our code into modules, build them into a single file bundle, and have the build process run automatically, refreshing our project in a webpage.

Our next step is to have our build system compile modern Javascript (ES2015, ES2016, ES2017) into code which can be run in current browsers (ES5). We also need to add in polyfills for some of the additional non-syntax language features.

We also will enable JSX compilation for use with React.

## Loaders and our starting point

Webpack has a couple ways to extend its functionality. There is a [plugin system](https://webpack.js.org/plugins/), which used for tools such as UglifyJS for code minification, and there is the [loader system](https://webpack.js.org/loaders/). Loaders allow you to pre-process files before they are added to the bundle.

We will use the final product of Part 1 as our starting point. You can clone the [basic-webpack-no-loaders](https://github.com/samsch/basic-webpack-no-loaders) repo to follow along, or if you followed Part 1, the project you ended up with.

## Final result

If you don't want to build the whole thing, or want to look ahead, the final result is in [basic-webpack-react](https://github.com/samsch/basic-webpack-react).

## Adding Babel

Our first step is to add and configure Babel. Babel is a Javascript and near-Javascript compiler. The Webpack loader is called babel-loader, and depends on babel-core as a peer dependency. Install both with `npm install -D babel-core babel-loader`.

Next we need to setup a babelrc file in the project root. This file configures the Babel plugins we need.

To setup this file, I used [@brigand](https://github.com/brigand)'s [config-wizard](https://brigand.github.io/config-wizard/?babelrc), with options "Play it safe", "None/Other", ["IE11", "iOS", "Chrome", "Firefox"]. This instructs us to install babel-preset-env with `npm install -D babel-preset-env`, and gives us our file content for `.babelrc`:
```json
{
  "presets": [
    [
      "env",
      {
        "targets": {
          "browsers": [
            "ie 11",
            "last 2 Chrome versions",
            "last 2 Firefox versions",
            "last 2 iOS versions"
          ]
        }
      }
    ]
  ]
}
```
Now we need to add babel-loader to `webpack.config.js`. We're adding a `modules` property with some options:
```js
  module: {
    rules: [
      {
        test : /\.js/,
        include: path.resolve(__dirname, 'src/'),
        use: ['babel-loader'],
      },
    ],
  },
```
>[View diff of file change](https://gist.github.com/samsch/15a058bd059b1bfe57d2400003348736)

With these changes, we can now use the same commands as before (`npm start`, `npm run dev`, `npm run build`), and it will work just the same, except that now if we use ES2015/16/17 code, it will be compiled to ES5 Javascript.

### Polyfills

Now we need to add the Babel polyfill (which is actually core-js and regenerator). Install it with `npm install -D babel-polyfill`. There are a couple ways to include polyfills in your code. The simplest is to just add `import 'babel-polyfill';` to the top of your entry file (`src/main.js`). A slightly cleaner approach is to include them via webpack config, which also makes the polyfills part of the build, rather than part of your code. We need to change the `entry` property in `webpack.config.js` to be `entry: ['babel-polyfill', './src/main.js'],`.
>[webpack.config.js diff](https://gist.github.com/samsch/0df1433b029dda87d73f092cf119d837)

## Test modern to ES5 compilation

Let's show that our code is being compiled properly. Change `srv/main.js` to be:
```js
import foo from './foo';

foo();

const j = () => console.log('Babel compilation is working!');

j();
```
If we compile with `npm run build`, and search in the (minified) `public/bundle.js` for "Babel compilation is working!", we find `function(){console.log("Babel compilation is working!")}`, which is using the `function` keyword rather than using an arrow function like our source.

## React... and more!

What we have now gives us great modern Javascript workflow, while supporting common browsers. Our next step takes us to features outside of Javascript. To work with React, most developers use JSX, which is an HTML-like language that compiles to Javascript. To add JSX support, all we need to do is install babel-preset-react, add it to `.babelrc`, and possibly make a minor change to our webpack config.

First add babel-preset-react with `npm install -D babel-preset-react`. Then change `.babelrc` to be:
```json
{
  "presets": [
    [
      "env",
      {
        "targets": {
          "browsers": [
            "last 2 Chrome versions",
            "last 2 Firefox versions",
            "last 2 iOS versions",
            "ie 11"
          ]
        }
      }
    ],
    "react"
  ]
}
```
>Generated with config-wizard similar to before, this time replacing "None/Other" with "React".

With just those changes, we can now compile JSX in our js files. Some developers prefer to use a different file extension for React components though: `.jsx`. We can support this by making a small change to our `test` line in `webpack.config.js`. Replace `test : /\.js/,` with `test : /\.jsx?/,`. Now Wepack will use babel-loader for `.js` and `.jsx` files.
>[webpack config diff](https://gist.github.com/samsch/54abd031b6ca088abe652371c8dd8754)

## Test React compilation

To see that we can now compile JSX, lets change `src/foo.js` to run some React code:
```js
import React from 'react';
import ReactDOM from 'react-dom';

const foo = function() {
  ReactDOM.render(
    <div>
      <h1>Hello, world!</h1>
      <p>This was rendered with React.</p>
    </div>,
    document.getElementById('app-body')
  );
}

export default foo;
```
Since we're importing React and ReactDOM, we also need to install them, so run `npm install -D react react-dom`.

Now we can run `npm start`, and we should see our "Hello, world!" and the rendered by React message.

## Next steps

We have a fully functional Webpack and Babel workflow now, which supports modern Javascript and React. The final product can be found in [basic-webpack-react](https://github.com/samsch/basic-webpack-react).

For moving beyond what's here, there are many Webpack loaders, and more features which can't be covered in a basic guide. Some popular topics:
- [Loading styles](https://webpack.js.org/loaders/#styling)
- [Code splitting](https://webpack.js.org/guides/code-splitting-import/)
- [Hot modules replacement (HMR)](https://webpack.js.org/guides/hmr-react/)

Happy coding!
