---
title: Getting Started with React
date: 2017-04-21 18:36:00
tags:
- React
- Webpack
- developer experience
- js
---
This will be the fast course to get a productive React development flow going. The only real prerequisite (besides a working computer and internet connection) is having a recent version of Node.js installed (as written, this is version 7.9.0, and LTS 6.10.2).

What this guide does not do is teach you Javascript or React. This just gets the environment in place.

## Let's go!
First thing to do is create your new project folder (I'll be using `react-project` in this guide), and run `npm init`. The requested information doesn't directly impact what we are going to be doing, so use whatever values you want.

We need an html page to attach our app to, so create a `public/` folder, and inside, create `index.html`. Paste this content into it:
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Basic React App</title>
</head>
<body>
    <div id="app"></div>
    <script src="bundle.js"></script>
</body>
</html>
```
If you are looking at this and saying, "man, that's a pretty sparse web page", well, you're right. For a "real" app, you are going to want to put your extra meta tags in, maybe Google Analytics, a link to your favicon, and whatever else you need. We'll get to the content next.

### Application source

Create a `src/` folder in the root of your project. All of your project specific source will be in this folder. For now, just create a `main.js` file in `src/`, and copy these contents into it:
```js
import React from 'react';
import ReactDOM  from 'react-dom';

ReactDOM.render(
    <h1>Hello, world!</h1>,
    document.getElementById('app')
);
```

So now your project folder should look like this:
```
- react-project
 |- package.json
 |- src/
 | |- main.js
 |- public/
   |- index.html
```

Well, we can see from above that we need at least a few dependencies from npm installed, so lets get started on those.

Run `npm install --save react react-dom`.

The `react` package is the ReactJS library, and `react-dom` is a sister library which lets you render a React virtual DOM to the real DOM.

That actually covers the entire source of the application. Now all we need to do is build it.

### Setting up Webpack

So, we primarily need two tools for building a React application: Webpack, and Babel.

Babel is a Javascript compiler (sometimes called a transpiler). It takes JSX and modern (ES2015, ES2016, and beyond) Javascript syntax, and compiles it to ES5 Javascript, which allows your modern code to run in older browsers.

>The compiled code potentially can work with even IE8 (with some extra effort). However, IE9 is the oldest browser which "normally" works. In 2017, IE11 and Android Browser are the targets of primary concern.

Webpack is a module bundler. You give it an entry point file which uses ES Modules or CommonJS (which is the same syntax that Node uses for its modules), and it finds and "bundles" all the required files into a single file, which you can then include in your web page.

So, lets install Webpack and Babel. Run `npm install --save-dev webpack babel-core babel-loader babel-polyfill`. Wait, what is babel-loader? Webpack plugins which change the bundled code are called loaders. babel-loader is then the Webpack plugin which allows Babel to be run against the files to be bundled. And babel-polyfill? That's to fill in additions to Javascript in ES2015+ that do not require new syntax, such as methods like `Object.includes` and objects like `Promise`.

"That's it right? now we just run `webpack` and we're done, right?" Not quite.

Our next step is to configure webpack. Technically, we could forgo configuring, and just use command line arguments to the Webpack cli interface. That would look something like `webpack src/main.js public/bundle.js --module-bind 'js=babel'`. However, using a configuration file is much more expressive, gives you more options, and is easier to maintain. So create `webpack.config.js` in the root of your project, and copy this into it:
```js
const webpack = require('webpack');
const path = require('path');

const BUILD_DIR = path.resolve(__dirname, 'public');
const APP_DIR = path.resolve(__dirname, 'src/');

const config = {
    entry: ['babel-polyfill', APP_DIR + '/main.js'],
    output: {
        path: BUILD_DIR,
        filename: 'bundle.js'
    },
    module : {
        rules : [
            {
                test : /\.jsx?/,
                include : APP_DIR,
                use : 'babel-loader',
            }
        ]
    },
};

module.exports = config;
```
Wow, that's a lot of configuration for a simple app! Let's take a closer look though. The first two lines are just requiring Webpack and the Node "path" module, and the next two lines are just setting some constants.

After that we have the actual configuration for Webpack. One nice point to note is that Webpack's configuration file is a Javascript file, not JSON like many others are (allowing comments, trailing commas, etc). In our configuration we set the `entry` property, which is the file that Webpack starts at (more on that in a moment); the `output`, which is where Webpack saves the compiled code; and the `module` property.

The `entry` configuration can take a string, array, or object. If you use a string, it should point at the single file that starts your application. If you use an array, Webpack will concatenate the files. Using an object allows you to have multiple entry points, which is for building multiple separate apps. In our case, we need our main.js file and babel-polyfill (which is loaded first).

Our `module` property has a `rules` property, which is an array of the defined loaders. For now, we are just defining babel-loader. `test` is a regex to match against file extensions. I'm using a pattern which matches `.js` and `.jsx` files. You could also choose to set it to only compile files with the JSX extension, by using `/\.jsx/`, or if you decide not to use .jsx at all, just use `/\.js/` to .js files. The `include` property tells Webpack that you only want to compile files in a specific folder. This is because you should never need to compile files from `node_modules`, since libraries should already be compiled with Babel (if they needed to be). This only stops Webpack from running Babel on files outside of APP_DIR, it doesn't stop Webpack from bundling those files into the output. The `use` property defines our loader, which is Babel.

The last line is simply exporting the configuration. By default, when Webpack runs, it uses `webpack.config.js` if it exists.

And that's it for our Webpack configuration. Now we just need to add our command to package.json:
```json
{
  ...
  "scripts": {
    ...
    "dev": "webpack -d --watch",
    "build": "webpack -p"
  },
  ...
}
```
We add two npm scripts, "dev" and "build". "dev" will run Webpack with "-d --watch", which sets webpack in debug mode (more verbose, and outputs code maps for the browser), and in watch mode, which means it continuously runs, re-compiling whenever a bundled file changes. "build" runs Webpack with "-p", which enables production mode. In production mode, the output is uglified (using Uglify), and any instances of `process.env.NODE_ENV` are replaced with `"production"`. That last part is important because it allows Uglify to remove React's internal debugging tools (which lets it run faster).

### Setup Babel

Ok, we're almost there. We have everything in place except for a little bit of Babel configuration. This one is easy. Create `.babelrc` in your project root, and copy this into it:
```json
{
    "presets" : ["latest", "react"]
}
```
What this does is tell Babel to compile ES2015+ and JSX to ES5 Javascript. These presets don't come with Babel, so install them with `npm install --save-dev babel-preset-latest babel-preset-react`.

This is the simple approach to the Babel config. For a more explicit configuration (such as for targeting ES2015 compatible browsers), check out @brigand's [config-wizard](https://brigand.github.io/config-wizard/?babelrc).

### Run!

That pretty much covers it. And now you know why the React workflow is notorious for being difficult to setup. While it's not actually that complex, there are many steps to the process, at least the first time through.

So, finally, to start your development environment, run `npm run dev`. You should see some output from Webpack, and two more files should appear in `public/`: `bundle.js` and `bundle.js.map`.

Now you just need a server to get these files into your browser. My go-to mini-server is `http-server`, available with npm. I install very few npm packages globally, but this is one. Install it with `npm install -g http-server`, then run it from your project root with `http-server public`. Now you should be able to visit `localhost:8080` in your browser, and see the web app you just created!

## Just give me the final product already

[Here it is, the whole thing wrapped up in a Github Repository.](https://github.com/samsch/basic-react)
