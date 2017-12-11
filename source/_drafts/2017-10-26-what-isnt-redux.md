---
title: what-isnt-redux
date: 2017-10-26 04:46:29
tags:
---
It's frequently argued that you should use Redux so you can avoid passing your entire state tree through your entire React component tree.

This is a bit like suggesting that you need a freight train to travel from A to B. `connect()` and getting data without passing props through the tree is **not** part of Redux. It's a [**separate** library called react-redux](https://github.com/reactjs/react-redux), which can actually be used with any store that has a redux-compatible API (namely the `dispatch`, `subscribe` and `getState` functions and immutable state).

If all you want is to not pass props through all components, consider whether you want all the baggage of a strongly opinionated state library (Redux, the freight train), or if it makes more sense to use react-redux with another state manager, or if it makes sense to use another context-based solution all together.

Let's a look at a couple examples:

Here's what using Redux and React looks like without react-redux: https://jsfiddle.net/samsch/899vv6kn/

This is the equivalent with react-redux: https://jsfiddle.net/samsch/ejrLuyvx/

If you don't see the value of the *first* example over something like https://jsfiddle.net/samsch/uuqno72z/ , then you probably shouldn't use Redux. If you see value in only the second example, take a look at using other context-based solutions, since Redux is just going to be noise.



As for the actual "why" of Redux, checkout the design goals from when it was being written: [Redux v0.3.0 design goals](https://github.com/reactjs/redux/tree/v0.3.0#design-goals)

The major theme of Redux was to make hot module reloading easy, which is made much easier by having an external, immutable, state controlled by pure (stateless, deterministic) functions (the reducer(s)).

It was found that Redux makes an ok general state store, and it can stop you from making mistakes which are easy in earlier Flux implementations. There are downsides to the library as well though. Because the entire store system (reducer tree) **must** be pure, you have to separate out impure, asynchronous and non-deterministic work. This works ok for some types of apps, but if you do a lot of async work, you can easily end up having half of your app logic outside of Redux in [Sagas](https://redux-saga.js.org/) or [Thunks](http://redux.js.org/docs/advanced/AsyncActions.html#async-action-creators). Instead, it can be much easier and clearer to write these types of apps using a state management system which doesn't have these limitations.

Using an external state store often makes using React much easier and cleaner. Redux is an available option, but I'd recommend taking a look at some other tools as well:

- Just use JavaScript!
  - [Over simplified example](https://jsfiddle.net/samsch/skom7z2e/)
  - [Basic example](https://jsfiddle.net/samsch/nsdrt8r4/)
  - [More advanced example](https://jsfiddle.net/samsch/8erd7m9m/)
  - Roll your own library! (e.g., I created [subscribe-store](https://github.com/samsch/subscribe-store/))
- Reactive programming!
  - [Tightly integrated with React](https://github.com/rikutiira/react-frp-todomvc)
  - You can also directly use reactive libraries like [RxJS](https://github.com/Reactive-Extensions/RxJS), [Kefir](https://rpominov.github.io/kefir/) and [Bacon](https://baconjs.github.io/); where you would subscribe and pass props similar to the Just use JavaScript examples above.
- Use older model libraries like [Backbone](https://reactjs.org/docs/integrating-with-other-libraries.html#using-backbone-models-in-react-components)
  - *Note that using backbone isn't really recommend by most people these days, but this provides a decent example of using a model that wasn't designed around React*
- Other React-focused state management libraries
  - [MobX](https://mobx.js.org/)
  - A search for "react state management library" on Google will turn up many less-known libraries (as well as some dead libraries, such as [marty.js](https://github.com/martyjs/marty#marty-is-no-longer-actively-maintained-use-alt-or-redux-instead-more-info))
