---
title: How to Add css-loader to Webpack
date: 2017-07-14 21:16:00
tags:
  - webpack
  - react
  - css
  - css-loader
---
One of the benefits of using Webpack is that is allows you to load non-JavaScript files using the standard JavaScript import statements. Specifically, this is regularly used to load stylesheets on a per-module basis.

> The final results can be found in a [branch from the base repo](https://github.com/samsch/basic-webpack-react/tree/css-loader-extract-text-plugin).

# What we need to get started
Our baseline is a working Webpack project. A good starting point is my [basic-webpack-react tutorial results](https://github.com/samsch/basic-webpack-react). Run `git clone https://github.com/samsch/basic-webpack-react.git`, `cd basic-webpack-react`, and `npm install`.

# Loaders for styles
Loading css in a useful way requires either two Webpack loaders, or a loader and a plugin. The easier setup to get started with is using `style-loader` and `css-loader`.
  - [style-loader](https://webpack.js.org/loaders/style-loader/) takes an imported css file, and makes it output as a `<style>` element with the css from the file.
  - [css-loader](https://webpack.js.org/loaders/css-loader/) parses a css file for `@import`, `url()`, and translates those into something Webpack can understand to import.

With these, you can have imports in your JavaScript modules like:
```js
import './style.css';
```
Webpack will take that stylesheet -- and anything it `@import`s -- and include JavaScript in your bundle to put it in `<style>` tags in the output when the app starts.

The downside of this method is that styles can't be cached by the browser separately. Later, we'll use the other method for a better production setup.

# Configuration
First install the loaders by running `npm install -D style-loader css-loader`.

Then we need to modify [webpack.config.js](https://github.com/samsch/basic-webpack-react/blob/f57ca65db1bb8462a9415a065a628b52163b34d0/webpack.config.js). Add the new loader rule to the `rules` property as shown below.
```diff
   module: {
     rules: [
       {
         test : /\.jsx?/,
         include: path.resolve(__dirname, 'src/'),
         use: ['babel-loader'],
       },
+      {
+        test: /\.css$/,
+        use: [ 'style-loader', 'css-loader' ]
+      },
     ],
   },
```

That's it! Now you can run `npm start`, and it will act exactly as before, but it's ready to import css files.

# Add some test styles
Let's add `import './style.css'` to the top of `src/foo.js`.
```diff
+import './style.css';
+
 import React from 'react';
 import ReactDOM from 'react-dom';
 
 const foo = function() {
```
The running Webpack process throws and error because style.css isn't found. So let's add `src/style.css`:
```css
h1 {
  border-bottom: solid 2px gray;
}
```
Now Webpack compiles, and the page should refresh with a border applied to the h1 tag.

Let's add an imported style. Make `src/paragraph.css`:
```css
p {
  background: #ddd;
  padding: 1rem;
}
```

And add an import line to `src/style.css`:
```diff
+@import "paragraph.css";
+
 h1 {
   border-bottom: solid 2px gray;
 }
```
> Note that we can use "paragraph.css" here, rather than "./paragraph.css" which Webpack would normally need for relative files. To import a file from an npm package, you can use a `~` prefix, like `~some-css-package/dist/style.css`.

Now we have css styles imported from our modules, and css import bundling!

# Separate style sheets
The above system actually works pretty well for smaller projects and development. But for larger projects and production, we often want separate stylesheets for caching.

To pull our styles into a new file, we use `extract-text-webpack-plugin`, so install that with `npm i -D extract-text-webpack-plugin`, and remove `style-loader` with `npm remove style-loader`. We need to make a couple changes in `webpack.config.js`.

At the top of the file:
```diff
 const path = require('path');
+const ExtractTextPlugin = require("extract-text-webpack-plugin");
 
 module.exports = {
   entry: ['babel-polyfill', './src/main.js'],
```
And near the end of the file:
```diff
       {
         test : /\.jsx?/,
         include: path.resolve(__dirname, 'src/'),
         use: ['babel-loader'],
       },
       {
         test: /\.css$/,
-        use: [ 'style-loader', 'css-loader' ]
+        use: ExtractTextPlugin.extract({
+          use: 'css-loader',
+        }),
       },
     ],
   },
+  plugins: [
+    new ExtractTextPlugin({
+      filename: 'style.css',
+      allChunks: true,
+    }),
+  ],
 };
```
Now if we re-run `npm start`, we don't have our styles anymore! This is because the styles output into a new file, which isn't automatically loaded. So we add a stylesheet link in `public/index.html`:
```diff
   <meta http-equiv="X-UA-Compatible" content="ie=edge">
   <title>Webpack bundle test page</title>
+  <link rel="stylesheet" href="/style.css">
   <script src="/bundle.js" defer></script>
 </head>
```
Refresh the page and your styles are back. If you check the network requests tab in your browser, you can see the stylesheet was loaded in separately.

This is now a mostly production-ready stylesheet management system. 
> The main pain-point for this configuration now is that files always have the same name, which means that you would need to do external versioning to bust the browser file cache. Fixing this is possible with a Webpack plugin, but it's a bit out of scope for this guide.

The final results can be seen as a [branch of the base repo](https://github.com/samsch/basic-webpack-react/tree/css-loader-extract-text-plugin).
