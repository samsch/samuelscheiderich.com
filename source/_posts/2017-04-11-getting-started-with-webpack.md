---
title: Getting Started with Webpack
date: 2017-04-08 10:49:29
tags:
- React
- Webpack
- developer experience
- js
---
Modern Javascript best practice is to use either ES modules or CommonJS modules for front end code. However, not all [browsers currently support a module system](http://caniuse.com/#feat=es6-module) directly, so to allow this practice, we use a tool called a bundler, which parses modules and creates a single file containing the full source.

There are a couple options for bundlers, but the popularity contest is largely being won by Webpack. There is some notoriety around Webpack, mostly due to the ability to create monstrous configurations for complex builds. However, Webpack itself is fairly straightforward, and this post will walk through the basic setup and configuration.

>In a hurry? The final product of this guide can be found in my [basic-webpack-no-loaders repository](https://github.com/samsch/basic-webpack-no-loaders).

## Webpack basics
The simplest usage of Webpack is very easy. Let's get a basic project started.

>Don't forget to reference the [Webpack documentation](https://webpack.js.org/configuration/) for more details, and to explore the options that Webpack provides!

### Prerequisites
You need to have Node.js installed.

For Linux and MacOS, the easiest way to install and work with Node is using nvm. Instructions for installing are on the [nvm project page](https://github.com/creationix/nvm#installation).

>If using Windows, you can download an installer from the [Node.js website](https://nodejs.org/en/).
>
>The instructions are the same for Webpack in Windows, except you may need to use backslashes instead of slashes for paths on the command line. For example, `webpack src/main.js public/bundle.js` might need to be `webpack src\main.js public\bundle.js` in Windows.

>You can of course use Yarn instead of npm for all the command below. Just remember to use `yarn add -D <package>` instead of `npm install -D <package>`.

### Create a new project

Make a new folder for your project (I'm using `basic-webpack`). Then open this folder in a terminal.

Run `npm init`. It will prompt you for project details, but you can just hit enter for each option to use the default. In a real project, you would probably want to fill in actual details.

Next run `npm install -D webpack`. This will install Webpack as a development dependency.

### Add source code

Let's add some source code modules that we want to bundle!

Create a `src/` folder in your project, and then create a `src/foo.js` file. Add the following to the file:
```js
const foo = function() {
  console.log('Ran foo()!');
}

export default foo;
```
Then create a `src/main.js` file, with this content:
```js

import foo from './foo';

foo();
```

Now we can see that main.js is reliant on foo.js. Now we need to create a *bundle*, which is a single file with all the code in it, neatly packaged to be used in the browser.

### Use Webpack

The simplest way to use Webpack is from the command line. Webpack, like most runnable npm modules, can be installed globally so that you could just run `webpack <options>` directly. However, the best practice is to install locally to the project. So then to run Webpack, we either need to use `./node_modules/.bin/webpack` OR we can setup an **npm script**. We will be doing the latter.

In your `package.json` file (which was created by `npm init` above), add a line under the **scripts** property like this:
```json
  "scripts": {
    "webpack": "webpack"
  },
```
Now we can run Webpack with `npm run webpack -- <options>`.

Webpack runs on the command line with the form `webpack <entry> <output>`. We want to output our bundle to `public/`, and `src/main.js` is our *entry* file.

Run `npm run webpack -- src/main.js public/bundle.js`. (It will create the `public/` folder automatically if it doesn't exist.)

### Profit!

That's it! That's all there is to running Webpack in it's most basic form. You can change your `package.json` file to include the Weback command option so that you can just run `npm run webpack`. Change the line `"webpack": "webpack",` to be `"webpack": "webpack src/main.js public/bundle.js",`.

## Actually...

However, this isn't the standard way to use Webpack. There are a couple missing features which we usually want. You can actually do most of this from the command line, but the recommended path is to use a Webpack config file.

### Basic config-driven Webpack

If you changed your `package.json` file to include the paths in the Webpack script command, revert that change so that it's just:
```json
  "scripts": {
    "webpack": "webpack"
  },
```
Now create a `webpack.config.js` file in the project root, with these contents:
```js
const path = require('path');

module.exports = {
  entry: './src/main.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'public'),
  },
};
```
This is the bare basics to do the same thing as the command line. While it is a bit more verbose, it also is fairly clear.

Now you can run `npm run webpack`, and it will automatically find the config file (we used the default name) and build the bundle.

### Setup basic development and production modes

Webpack comes with some built-in development and production tool which can be enabled with command line options. Edit scripts in `package.json` again to be like:
```json
  "scripts": {
    "dev": "webpack -d --watch",
    "build": "webpack -p"
  },
```
Now we can do a production build with `npm run build`. For development, we can run `npm run dev`, and it will re-compile any time a file changes which will affect the bundle.

The `-d` and `-p` are shortcut options. `-d` enables sourcemaps and turns on debug mode for *loaders* (more on loaders later). `-p` sets the `NODE_ENV` environment variable to "production" and replaces usages of process.env.NODE_ENV with '"production"' in your code. `-p` also turns on UglifyJSPlugin, which does minification and dead-code removal. With these two combined, code such as the following will be removed in production builds:
```js
  if(process.env.NODE_ENV !== "production") {
    console.log('Debug mode!');
  }
```

The sourcemaps are not generated as a separate file by default. You can choose a different sourcemap output with a setting in the webpack config file. [Webpack devtool documentation](https://webpack.js.org/configuration/devtool/).

## Using the bundle, and automation!

What we have now is all you need to bundle Javascript files in development and production. All that needed is a server to serve the Javascript bundle, and a page that includes it.

In the [basic-webpack-no-loaders repo](https://github.com/samsch/basic-webpack-no-loaders), you can run `checkout basic-production-ready` to see the code as it should be if you followed along.

A quick look at what we have:

- We can put ES module and CommonJS module source files in `src/`, and also include modules from `node_modules/` installed with npm or Yarn.
- We can do a development build on change with `npm run dev`, and we can create a production-ready bundle with `npm run build`.

Our next step is to make the developer experience (DX) even better, by setting up webpack-dev-server.

### webpack-dev-server

Webpack-dev-server does two things: It acts as a static file server (by default), and it watches and bundles your source with some added code which will refresh your browser page when the source changes and the bundle is rebuilt.

Install webpack-dev-server with `npm install webpack-dev-server`.

Next add a start script to `package.json`:
```json
  "scripts": {
    "start": "webpack-dev-server -d --open",
    "dev": "webpack -d --watch",
    "build": "webpack -p"
  },
```

If we had an `index.html` file in our project root, and it used a script with a source that pointed to `/bundle.js`, then we could just run `npm start` and serve that. However, we don't really want our app to serve from the project root, we want to serve from the `public/` folder. Let's add a bit of configuration to `webpack.config.js`:
```js
const path = require('path');

module.exports = {
  entry: './src/main.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'public'),
  },
  devServer: {
    contentBase: path.join(__dirname, "public"),
    https: true,
  },
};
```
>The https setting is optional. These days HTTPS is strongly preferred for *all* websites. This option will set the dev server to automatically create a self-signed cert, and open the page via with https. You can also set a cert, key, and ca file. Check the [devServer documentation for instructions](https://webpack.js.org/configuration/dev-server/#devserver-https).

Next create `public/index.html`, and put this content in it:
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Webpack bundle test page</title>
  <script src="/bundle.js" defer></script>
</head>
<body id="app-body">
  <h1>Waiting for Javascript...</h1>
</body>
</html>
```
That's all we need, but to give us a slightly better first look, lets update `src/foo.js` to modify the html:
```js
const foo = function() {
  console.log('Ran foo()!');
  const date = new Date().toDateString();
  document.getElementById('app-body').innerHTML = `<h1>Hello, world!</h1><p>This is bundled Javascript! Today is: ${date}</p>`;
  if(process.env.NODE_ENV !== "production") {
    console.log('Debug mode!');
  }
}

export default foo;
```
Now with these changes, run `npm start`. This should open a new page in your browser with the above html.

>Since it will be using a self-signed cert, you will need to click through the "Advanced" option to allow it. In Chrome, you click "Proceed to <hostname> (unsafe)", and in Firefox you click "Add Exception..." and I recommend unchecking the permanent option (since the cert won't last very long anyway).

Congratulations! You now have webpack-dev-server running! You can test it out by changing `src/main.js` or `src/foo.js`. When you save changes to either file, the webpage in your browser will refresh with the new bundle.

The final product is available to clone in [basic-webpack-no-loaders](https://github.com/samsch/basic-webpack-no-loaders).

## Next steps

What we have now is a production ready Javascript build system. To make it your own, you can replace, change, and remove the files in `src/`, just make sure that the `entry` property in `webpack.config.js` points to your entry file (some common files are `src/main.js`, `src/index.js`, and `lib/index.js`).

The most common extra configuration for Webpack is to include babel-loader with preset to compile modern Javascript to ES5 Javascript for browser compatibility. That will be explored in the [next post](https://samsch.org/2017/04/14/getting-started-with-webpack-part-2/).

Happy coding!
