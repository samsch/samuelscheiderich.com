---
title: React is a Leaky Abstraction (and that's ok)
date: 2020-03-30 13:38:42
tags:
  - React
---
## What React Abstracts

React is a view rendering and state framework, designed to abstract over the idea of making your view a function of your state. For web, that means it abstracts over the DOM. What this means is that React gives you tools to work with the view as a higher-level idea and through a higher-level API than the browser's built-in interfaces for creating, updating, and deleting elements.

> React Native takes the same high-level abstraction and applies it to native mobile view primitives. The ability to do this is a key feature of a high-level abstraction. I'm going to talk about React in the context of DOM for this article though.

Abstracting the DOM makes writing a view much easier. Instead of calling `document.createElement()` for each node, manually mutating each property, setting event listeners, holding all the element references, updating properties, deleting elements, removing event listeners, and so on and so forth; with React you describe exactly the final result of what you want the view markup to be, and React does all the work to make the DOM conform to that request.

## How it leaks

That last little nugget of how it works is an important point! You describe the *markup* of the view you want. But you are still describing the view as "html" (JSX), which translates to DOM. So React *isn't* hiding the DOM from you entirely, you still see it, describe it, and can get access to elements directly with a ref. Not only that, but the properties you pass are the actual DOM properties React will set the elements to (with some transformations applied to some properties, such as styles and event listeners).

This makes React a "leaky" abstraction. It doesn't completely remove the idea of the DOM from your code, and the DOM rules still influence how you have to write your code. This becomes immediately clear when you do compare React for DOM to React Native. In React DOM you describe html markup in render functions, but in React Native you use "View" representations that map to ideas in Android and iOS.

## Why that's ok

So React isn't a perfect abstraction of the view. This is ok! Even though it leaks, React still provides a much more powerful interface for creating views than the DOM primitive methods, and *parts* of using React *are* fully abstracted. You never need to directly create, add, move, or remove DOM elements when using React (with only a very few small exceptions which can be easily isolated, like using third party libraries which themselves manipulate the DOM directly).

Other parts of React will commonly require some knowledge of DOM, html, and how they work. A really good example of this is styles. You still have to work with css while using React DOM for styles. React doesn't provide any styling considerations beside the (transformed) `style` property and className property. To style elements in React, you must work with css (through the DOM).

## How does this information help me?

Understanding your tools better helps you use them more effectively. Knowing how React is a leaky abstraction allows you to better make decisions about using other abstractions with it, when to rely on the framework, and when you can safely work around the framework with standard DOM tools.

For example, using React without any extra styling tools means either passing style props with css properties, or passing className props. But because you know React doesn't abstract around styles at all means you can easily use style abstraction tools that aren't specifically designed for React, such as `emotion`'s `css` template function which returns a class name string (which can be directly passed), but that you can also use style abstractions which are built around React as well, leveraging the React component abstraction, such as styled-components's `styled.*` template functions.

## How will this change?

The status of building views on the web right now requires knowledge of html and css, and generally DOM. It's possible in the future we might see a perfect abstraction of building views, which can then be completely portable to other view systems. There are attempts to do this now, with tools like React Native Web, which attempts to turn the React Native primitives into a general abstraction over both Native views and DOM views. But for at least a while longer, you will still need to work with the leaks.