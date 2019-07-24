---
title: Effects are not lifecycles
date: 2019-07-23 16:58:11
tags:
  - react
  - hooks
  - lifecycles
---
You can't write lifecycles with useEffect.

With React hooks being widely regarded as "better" than using classes in the React community, both for new users and for experienced developers, there's a wide pattern of developer migration to learn the new tools.

Most of these developers are bringing with them the concepts they've gotten used to with React classes and even from non-React frameworks or tools. Some of these are easy to directly transfer across: It's not terribly hard to pick up `useState` if you are used to class state, and `useRef` is fairly straight forward for many as well, once they get the basic concept of how hooks hold on to state.

## Lifecycles are "when" you do things

React class component authors are used to writing functionality in lifecycles, and lifecycles don't exist with hooks. You can *emulate* them if you're careful, maybe using some `useRef` instances to reference changing props because of closures. But emulating lifecycles is a bad idea, and the reason why is this: Effects are a higher-level abstraction than lifecycles.

When you use a lifecycle like componentDidMount, or componentDidUpdate (let alone the older deprecated lifecycles which ran at different stages), you must think in terms of *when* something should happen. "I want the data loaded when the component mounts." "I want to load data if when the component updates with a new X prop." This idea of "when" is procedural thinking. The "when" concept isn't actually important, but because the tool for completing these tasks is lifecycles, you need to map the "what" that you want to do, to the "when" of a specific lifecycle.

Well I'm here to tell you to forget all of that. Seriously, forget the concept of "when" entirely. You don't care *when* something happens. You really don't. You think you might for this specific thing? You don't.

## Effects are "what", not "when"

React is a strict model. It's part of why it's so powerful and flexible. The model says "given X state, the view should be viewFunction(X)". For a long time, we had to break this model for anything that wasn't direct view output. Instead of "given X state, do effectFunction(X)", we had to break down *when* we wanted those things to happen, and sort them into lifecycle methods.

With `useEffect`, you say "given X state, do effectFunction(x)". What's important now is just *what* your state is, and *what* you should do given that state. "When" doesn't matter anymore. If we apply this to a simple case, it matches up to making sense practically. With lifecycles, you would do async loads of your data in componentDidMount. You did it at mount, because you know it's not previously been done then. But do you *actually* care about it being at mount? Isn't what really matters that you load the data *when it hasn't already been loaded?* So we just boiled it down to the important part: When our state is that data is not yet loaded, we want to load the data.

That concept is how `useEffect` works. We don't care that the component is mounting, we just write in our useEffect that we want the data to be loaded if it hasn't been already. What's more, from a high level, we don't usually even care if it loads the data multiple times, just that the data gets loaded.

## What it looks like in code

Now we've boiled down the *what* that we want to do. "When data isn't loaded, load the data."

The naive approach looks like this:

```jsx
const [isLoaded, setLoaded] = useState(false);
const [data, setData] = useState(null);

useEffect(() => {
  if (isLoaded === false) {
    loadData().then(data => {
      setData(data);
      setLoaded(true);
    });
  }
});
```
This code *works*. It's the most naive approach given our concept of what we *want*, but it works perfectly fine.

> Arguably, there are more naive approaches, but we're making the assuming here that we already know *how* hooks work, so we aren't taking into consideration putting the `useEffect()` inside the condition, since that is a known error.

Let's compare that to what the code looks like if you emulate `componentDidMount` using `[]` as a second argument.
```jsx
const [data, setData] = useState(null);

useEffect(() => {
  loadData().then(data => {
    setData(data);
    setLoaded(true);
  });
}, []);
```
At first glance, there is less code involved, which you might argue is a good thing. But this code doesn't describe the situation as well. We have *implicit* state. It *looks* like the `loadData()` function should run every time, because there is no *semantic* code which says it won't. In other words, we aren't *describing* what the code is actually supposed to do. If you remove the `[]`, then this code looks almost identical, but simply doesn't work properly (it always loads data, instead of just when we need it). What's more, we very likely need the loading state in render anyway, and while you can assume that `null` data means it's not loaded, you are breaking single responsibility principle by overloading the meaning of a variable.

This is a very common stumbling block that people trip over when learning hooks, because they try to emulate lifecycles.

## Optimizing

Now, for practical purposes, we *don't* actually want the `loadData` function called more than once. If you follow the simplest application of what should be in the second `useEffect` argument dependencies (every outside reference), this is automatically fixed:
```jsx
const [isLoaded, setLoaded] = useState(false);
const [data, setData] = useState(null);

useEffect(() => {
  if (isLoaded === false) {
    loadData().then(data => {
      setData(data);
      setLoaded(true);
    });
  }
}, [isLoaded, loadData, setData, setLoaded]);
```

The two setters won't change, but they are semantically deps of the function, and maybe down the road they get replaced by something that might change. We'll assume for now that `loadData` won't change (if it did, it will only trigger a new call *if* `isLoaded` is still `false`). Our key dependency here is `isLoaded`. In the first pass, we automatically run the effect, and `isLoaded` is false, so `loadData` is called. If the component renders again for some reason while `isLoaded` is still false, the deps won't have changed, so the effect won't run again.

Once `loadData()` resolves, `isLoaded` is set true. The effect runs again, but this time the condition is false, so `loadData()` isn't called.

What's important to take away from this is that the dependency argument *didn't change* our functionality at all, it just made it so we make fewer useless calls to a function.

## But what about things that *shouldn't* be loaded more than once!

Ah, right. Maybe it's making a call which changes something somewhere else. It should *only* be called once when needed.

This means our "what" changed. It's no longer "if not loaded, load data", it's now: "if not loaded, *and not already loading*, load data." Because our "what" changed, our semantic code should change too.

We could simply add a `isLoading` state, but then we could have something confusing happen like `isLoading` and `isLoaded` both true! Since these state should be *exclusive*, that means they are also *related*. And more than related, they are actually the *same* state item (the data status), just different values.

So now we change our state code to reflect our new "what":
```jsx
const [dataStatus, setDataStatus] = useState('empty');
const [data, setData] = useState(null);

useEffect(() => {
  if (dataStatus === 'empty') {
    loadData().then(data => {
      setData(data);
      setDataStatus('available');
    });
    setDataStatus('loading');
  }
});
```

Now we have code which *only* calls `loadData` when we need it and it isn't already loading, AND it doesn't use the dependency argument of `useEffect`.

Additionally, the different parts of our state are all explicitly included here.

## Tell me what to do!

So, forget about lifecycles, mounting, updates, and generally "when" thing happen. Just completely put it out of your head.

Think about *what* you need to do, and *what* the states are that should cause those things to happen.

Model those states explicitly in your code, and model the effects to be called for certain state conditions.

Your code should *always* work without using the second argument to `useEffect`. If you *need*, the second argument, you are probably incorrectly coding your functionality.
