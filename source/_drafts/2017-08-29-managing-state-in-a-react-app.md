---
title: Managing State in a React app
date: 2017-08-29 07:59:53
tags:
- React
- state
- patterns
- js
---
How to manage state in React.

## Fully top-down
This is easily the most straight forward way to use React. You pass state and update functions as props to the top level render, and re-render when the state changes.

A basic (simplified) example:
```jsx
const Input = props =>
	<input value={props.value} onChange={e => props.onChange(e.currentTarget.value)} />;

const Output = props =>
	<div>
	  Output: {props.value}
	</div>;

let value = '';

const containerEl = document.getElementById('container');

const render = () => {
	ReactDOM.render(
 		<div>
 		  <Input value={value} onChange={newValue => {
        value = newValue;
        render();
      }} />
      <Output value={value} />
 		</div>,
    containerEl
  );
};

render();
```
> [On JSFiddle](https://jsfiddle.net/samsch/skom7z2e/)

For a slightly more realistic example, check out [this fiddle](https://jsfiddle.net/samsch/nsdrt8r4/).

## Internally
React class component include some features not available in function components, including a simple state store.

> One of the biggest catches which causes new users trouble is that the setState function can't be treated as a synchronous update (it's also not strictly async, but can usually be treated as if it is). You can read more about how it works and how to use it in [the documentation](https://facebook.github.io/react/docs/react-component.html#setstate).

A basic input example:
```jsx
class App extends React.Component {
	constructor(props) {
  	super(props);
    this.state = {
      text: '',
    };
  }
  render() {
    return (
    	<div>
    	  <div>
          <input
            value={this.state.text}
            onChange={e => {
            	// events can't be directly stored in React, so we need to "cache" the value.
            	const newText = e.currentTarget.value;
            	this.setState(prev => ({ text: newText }));
            }}
          />
        </div>
        <div>
          Output: {this.state.text}
        </div>
    	</div>
    );
  }
}

ReactDOM.render(
  <App />,
  document.getElementById('container')
);
```
> [On JSFiddle](https://jsfiddle.net/samsch/4h2ryo3u/)

## With context
Lastly, React includes a shortcut (called context) which can pass data down the component tree without having to be included in the props at each level. This relies on some other actual data store, and also custom update logic, since context updates don't guarantee that any child using that context will update.

React context is also considered an "unstable" feature, meaning the React team is not actually happy with how it works, and if they can find a better solution it will likely get replaced. However, React also follows semver, so any non-backward compatible changes will only happen in major versions.

Because of the "unstableness" of context, it's recommended that you use a library which wraps the functionality, rather than use the API directly in your app. Another possiblity is to create such a wrapper yourself.

> To learn more, check out [How safely use React context](https://medium.com/@mweststrate/how-to-safely-use-react-context-b7e343eff076) on medium.

# When to use what
If you are just getting into React, you should only do fully top down rendering. (You should also only use function components here, which helps enforce this.) Using React strictly top down will help solidify how React is meant to be used.

Keeping everything rendered top down allows for the strongest application logic and state vs view separation. It's easier to reason about your application when your view is irrelavant, and it's easier to reason about your view when your application is only relevant in that it provides state and functions which you use in render and event handlers.

React component state is handy tool for some situations, but should almost always be only used for "view state" purposes, never business logic. You might make an exception for a small "widget" where you only have a couple components and the functionality is trivial, but even then, it's often going to be easier to move the functionality outside of React and just have React render the view.

## Using context
In general, you don't need to use context for most apps. The advantage to using it is that you don't need to pass everything needed for every component through all of its parents. It also allows you to delegate selecting what state is needed to the component which use it.

However, there are downsides too. It can be more difficult to trace state usage in your view when components further down the tree