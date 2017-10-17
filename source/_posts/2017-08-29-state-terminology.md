---
title: State Terminology
date: 2017-08-29 09:20:37
tags:
- React
- state
- patterns
- js
---
What does someone mean when they say "state" in React contexts? Glad you asked! Unfortunately, it's not something with a simple answer. I can break "state" into at least five different meanings, all of which are used in documentation, IRC chats, forums posts, and elsewhere.

### General state
The general idea of "state", is the remembered events, data, and user interactions of the application. It is the part of an application which can change. For a longer definition, check out [State \(computer science\)](https://en.wikipedia.org/wiki/State_%28computer_science%29) on Wikipedia.

### Application state
A subset of the general state in an app is the "application state". This is data relevant to the actual functionality of the application. It may have properties including form data, current user info, and lists of data.

### View state
Another subset of the general state is "view state". This is data which is used in the user interface to improve or change the user experience. This state is never relevant to the application, such that it is possible to have the exact same application functionality without any view state<sup>[1](#view-state-removed)</sup>. Included types of information in the view state may be whether a dropdown is open, the current page, or a selected theme.

### Component state
This is where some confusion usually starts to set in. Many users (including blog, documentation, forum post, and real-time chat authors) also use "state" to refer to the built-in state handling functionality in `React.Component` (and `React.PureComponent`) classes. This functionality is not itself actual "state", but is actually a type of "store". "Component state" is just a tool, and it allows you to store any type of state.

### \<store\> state (e.g., "Redux state")
In contrast to using "state" to refer to "component state", it's also used to refer specifically to state stored in other types of state stores. Currently, a popular state store is Redux, but this also applies to any external store (Mobx, Backbone, other Flux implementations). Sometimes, users will use "application state" to refer to state stored outside of React "component state".

## The Right Way
Of course, The Right Way doesn't exist. Nobody wants to explicitly type "application state" or "component state" all the time, even though that would allow much less ambiguity.

When communicating with someone, I recommend verifying what they mean when they say "state" if it might be ambiguous, and being explicit at least once in a while yourself to help others understand you.

> <a name="view-state-removed">1.</a> Generally this is true, but in most circumstances having a stateless user interface would require vast changes to an application or website, including moving everything into a single page and replacing dropdowns with different types of UIs (radio buttons, checkbox lists, etc). Even then, trying to remove some state such as mouse and scroll position would make it very difficult to maintain the same application functionality. To have a **perfectly** stateless UI for a stateful app would be very difficult or impossible.
