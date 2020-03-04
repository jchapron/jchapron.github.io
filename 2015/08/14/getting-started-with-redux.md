---
layout: post
title: An early look at Redux - Getting started
date: 2015-08-14
tags:
  -javascript
  -redux
  -react
---

As Redux is about to hit the 1.0 release, I decided to take some time to write an article on the subject to share my experience.
I recently had to pick a "flux implementation" for a client app and so far working with Redux has been a pleasure.

#### Why Redux?

Redux defines itself as a predictable state container for JavaScript apps. Redux is inspired by [Flux][1] and [Elm][2]. If you come from the Flux world, I encourage you to read how Redux is related in the ["Prior Art chapter"][3] of the new (awesome) docs !

Redux makes you think of your application as an initial state being modified by a sequential list of actions, which I found is a really nice way to approach complex web apps and opens up a lot of opportunities.

Hence, Redux enables tools like logging, hot reloading, time travel, record and replay with no extra work from the developer which makes working on a Redux app really fun and easy ! It also means we can implement complex features like undo/redo really easily.
If you're not sold already, I don't know what to tell you !

Of course you can find out more on Redux, its architecture and the role of each component on the [official docs][4].

[1]: https://facebook.github.io/flux/
[2]: https://elm-lang.org/
[3]: https://redux.js.org/introduction/prior-art/
[4]: https://redux.js.org/

## Building a Friend List with React and Redux
Today we will focus on guiding you step by step through your first app using Redux and React by building a simple friend list from scratch so you can grasp the main concepts.

You can find the full code at : [https://github.com/jchapron/redux-friendlist-demo/tree/v1.0](https://github.com/jchapron/redux-friendlist-demo/tree/v1.0)

![the project](../../../../assets/images/getting-started-with-redux/thefriendlist.gif)

#### Who is this for ?

This article is aimed at people with no prior experience of Redux. No prior experience with Flux should be necessary neither. I'll reference the docs whenever we encounter a new term.

## 1. The setup

Redux's author [Dan Abramov](https://twitter.com/dan_abramov) created a nice boilerplate with React, Webpack, ES6/7 and React Hot Loader which you can find at : [https://github.com/gaearon/react-hot-boilerplate](https://github.com/gaearon/react-hot-boilerplate)

There are boilerplates out there with redux already set up but I think it's important to understand the role of each library.

```
$ git clone https://github.com/gaearon/react-hot-boilerplate.git friendlist
$ cd friendlist && npm install
$ npm start
```

You can now access your app at http://localhost:3000. As you can see you get a nice hello world already !

### 1.1 Adding redux, react-redux and redux-devtools

We need to install three packages :

Redux : The library itself
React-redux : The bindings to React
Redux-devtools : Which are optionals but give you some cool tools for development.

```
$ npm install --save redux@1.0.0-rc react-redux
$ npm install --save-dev redux-devtools
```

### 1.2 Directory structure

Although what we are building is fairly simple, let's create a directory structure that would look like what we might have in the real world.

```
.
+-- src
|   +-- actions
|       +-- index.js
|   +-- components
|       +-- index.js
|   +-- constants
|       +-- ActionTypes.js
|   +-- containers
|       +-- App.js
|       +-- FriendListApp.js
|   +-- reducers
|       +-- index.js
|       +-- friendlist.js
|   +-- utils
|   +-- index.js
+-- index.html
+-- app.css
```

We will see in greater details the role of each directory as we build our app. We moved App.js to containers so you will need to adjust the import statement in index.js.

### 1.3 Connecting redux

#### Enabling devtools

We want to enable devTools in development environment only so let's modify our webpack.config.js like so:

```javascript
/* webpack.config.js */

var devFlagPlugin = new webpack.DefinePlugin({
  __DEV__: JSON.stringify(JSON.parse(process.env.DEBUG || 'false'))
});

[...]

plugins: [
    new webpack.HotModuleReplacementPlugin(),
    new webpack.NoErrorsPlugin(),
    devFlagPlugin
]
```

When starting our app with DEBUG=true npm start, this gives a __DEV__ flag which we can use in our app. We can implement devTools this way :

```jsx
/* src/utils/devTools.js */

import React from 'react';
import { createStore as initialCreateStore, compose } from 'redux';

export let createStore = initialCreateStore;

if (__DEV__) {
  createStore = compose(
    require('redux-devtools').devTools(),
    require('redux-devtools').persistState(
      window.location.href.match(/[?&]debug_session=([^&]+)\b/)
    ),
    createStore
  );
}

export function renderDevTools(store) {
  if (__DEV__) {
    let {DevTools, DebugPanel, LogMonitor} = require('redux-devtools/lib/react');
    return (
      <DebugPanel top right bottom>
        <DevTools store={store} monitor={LogMonitor} />
      </DebugPanel>
    );
  }
  return null;
}
```
We are doing two things here. We are overriding createStore using the compose function, which allows us to apply multiple [store enhancers](https://redux.js.org/glossary#store-enhancer) like the devTools. We also expose a function renderDevTools that renders the DebugPanel.

#### App container

Next we need to modify our App.js to connect redux. To do this, we will use the Provider from react-redux. It's a component that takes your store as property and has a function as child. Do not worry about the weird looking function, its goal is to use the "context" function of React to make store available to all children. This will be simplified when React 0.14 is released as it feels quite 'hacky' right now.

```javascript
/* src/containers/App.js */

import React, { Component } from 'react';
import { combineReducers } from 'redux';
import { Provider } from 'react-redux';

import { createStore, renderDevTools } from '../utils/devTools';

import FriendListApp from './FriendListApp';
import * as reducers from '../reducers';

const reducer = combineReducers(reducers);
const store = createStore(reducer);
```
To create the store we use the createStore function that we've overridden in our devTools file as well as a map of all our reducers. The ES6 feature import * as reducers gives us an object looking like { key: fn(state, action), ... }. It's a nice way to construct the argument for combineReducers.

```jsx
export default class App extends Component {
  render() {
    return (
      <div>
        <Provider store={store}>
          {() => <FriendListApp /> }
        </Provider>

        {renderDevTools(store)}
      </div>
    );
  }
}
```
In our application, App.js is the top-level wrapper for Redux and FriendListApp the root component of our app. After creating a simple 'Hello world' in FriendListApp.js, we can finally start our app with redux and the devTools. You should get this (without the styles) :

![step 0](../../../../assets/images/getting-started-with-redux/step_0.png)

Although this is just an Hello world app, we already have Hot Reloading included, so you can modify the text and it should update automatically. As you can see, the developers tools on the right displays your empty store. So let's fill it up !

## 2. Building the app
Now that we are done with setting up things, we can focus on the core of our application.

### 2.1 Actions and actions creators
Actions are payloads of information that send data to your store. By convention, they have a type attribute that indicates what kind of action we are performing. Defining those types in a different module is a good practice and makes us think early on about what will our app do.

```javascript
/* src/constants/ActionTypes.js */

export const ADD_FRIEND = 'ADD_FRIEND';
export const STAR_FRIEND = 'STAR_FRIEND';
export const DELETE_FRIEND = 'DELETE_FRIEND';
```
As you can see, this is a pretty expressive way of defining the scope of our application, which will allow us to add a friend, mark him as favorite or remove him from our friend list.

Action creators are function that create actions. They do not dispatch to the store, making them portable and easier to test as they have no side-effects. We put them in the actions folder but keep in mind that they are a different notion.

```javascript
/* src/actions/FriendsActions.js */

import * as types from '../constants/ActionTypes';

export function addFriend(name) {
  return {
    type: types.ADD_FRIEND,
    name
  };
}

export function deleteFriend(id) {
  return {
    type: types.DELETE_FRIEND,
    id
  };
}

export function starFriend(id) {
  return {
    type: types.STAR_FRIEND,
    id
  };
}
```
As you can see, actions are pretty minimal. For adding an element we communicate all its properties (here we only deal with a name) and for the others we only reference the friend's id. In a more complex app, we would probably have to deal with async actions here, but this is for another episode...

### 2.2 Reducers
Reducers are the ones that are in charge of modifying the state of the application. They are pure functions with the following signature : (previousState, action) => newState. It's critical to understand that you should never (ok, almost never) mutate the previousState in your reducer. Instead you can create a new object based on the previousState properties. Otherwise bad things will happen (for starters you will break time travel). Also, this is not the place to handle side effects like routing or async calls.

We first need to define the looks of our application state by defining initialState :

```javascript
/* src/reducers/friends.js */

const initialState = {
  friends: [1, 2, 3],
  friendsById: {
    1: {
      id: 1,
      name: 'Theodore Roosevelt'
    },
    2: {
      id: 2,
      name: 'Abraham Lincoln'
    },
    3: {
      id: 3,
      name: 'George Washington'
    }
  }
};
```
As state can be whatever we want and our example is simple, we could just maintain an array of friends. But this approach doesn't scale very well so we will use an array of ids and a map of friends instead. You can read about this approach in normalizr.

Next we need to write the actual reducer. We use the ES6 feature of default arguments to handle the case of state being undefined. It's up to you how you write your reducer, in this case I'll use a switch.

```javascript
export default function friends(state = initialState, action) {
  switch (action.type) {

    case types.ADD_FRIEND:
      const newId = state.friends[state.friends.length-1] + 1;
      return {
        friends: state.friends.concat(newId),
        friendsById: {
          ...state.friendsById,
          [newId]: {
            id: newId,
            name: action.name
          }
        }
      }

    default:
      return state;
  }
}
```
If you are not familiar with ES6/7 features, you're probably as perplex as I was reading this. As we want to return a new state object, it's quite common to see people use Object.assign (docs) or the Spread operator (docs).

What's going on here is that we define a new id. In a real-world app we would probably get it from the server or at least make sure it's unique.
We then use concat to append this new id to our list of ids. Concat gives a new array and does not mutate the original one, so we are safe to use it here.

Computed properties is a nice ES6 feature that allow us to easily create a dynamic key in the friendsById object with [newId].

As you can see, despite the syntax that may be confusing at first, the logic is simple. You are given a state, you give back a new state. The important thing being that at no point in this process did we mutate the previous state.

So let's go ahead and implement our two other actions :

```javascript
import omit from 'lodash/object/omit';
import assign from 'lodash/object/assign';
import mapValues from 'lodash/object/mapValues';

/* ... */

case types.DELETE_FRIEND:
      return {
        ...state,
        friends: state.friends.filter(id => id !== action.id),
        friendsById: omit(state.friendsById, action.id)
      }

    case types.STAR_FRIEND:
      return {
        ...state,
        friendsById: mapValues(state.friendsById, (friend) => {
          return friend.id === action.id ?
            assign({}, friend, { starred: !friend.starred }) :
            friend
        })
      }
```
I added lodash as a dependency to manipulate objects more easily. As always in those two examples, the important thing is to not mutate the previous state, so we use functions that return a new object. For example instead of using "delete state.friendsById[action.id], we use the `_.omit` function that is not destructive.

You can also remark that the spread operator allows us to only manipulate the part of the state that we want to modify.

Redux doesn't care how you store your data so if a tool like Immutable.js makes sense in your project, you're free to use it.

At this point you can play with your store without the interface by calling dispatch manually in our App.js.

```javascript
/* src/containers/App.js */

import { addFriend, deleteFriend, starFriend } from '../actions/FriendsActions';

store.dispatch(addFriend('Barack Obama'));

store.dispatch(deleteFriend(1));

store.dispatch(starFriend(4));
```
You will see in your devTools the actions being displayed, and you can already play with the time travel feature !

![step 1](../../../../assets/images/getting-started-with-redux/step_1.png)

## 3. Building the UI
As this is not the focus of this tutorial, I'll skip over the creation of React components and only focus on their relationship with Redux.
We have three simple components :

* FriendList is the list of friends, it renders an <ul>
  * friends: array is an array of friends
* FriendListItem is a single friend element <li>
  * name: string is its display name
  * starred: boolean shows a star when marked as favorite
  * starFriend: function is the callback to invoke when the user clicks on the star item.
  * deleteFriend: function is the callback to invoke when the user clicks on the trash item.
* AddFriendInput is an input field to enter our new friend name
  * addFriend: function is the callback to invoke when hitting enter.

In Redux, it's a good practice to keep your components as "dumb" as possible, meaning the less they know about Redux the better. Here, FriendListApp will be the only "smart" component.

```javascript
import { bindActionCreators } from 'redux';
import { connect } from 'react-redux';
import * as FriendsActions from '../actions/FriendsActions';
import { FriendList, AddFriendInput } from '../components';

@connect(state => ({
  friendlist: state.friendlist
}))
export default class FriendListApp extends Component {
```
Oh god what is that!? OK don't worry this is an optional ES2016 feature called a decorator. They are a handy way of calling high order functions. The equivalent would be `connect(select)(FriendListApp);` where select is a function that returns what we did here.

```jsx
  static propTypes = {
    friendsById: PropTypes.object.isRequired,
    dispatch: PropTypes.func.isRequired
  }

  render () {
    const { friendlist: { friendsById }, dispatch } = this.props;
    const actions = bindActionCreators(FriendsActions, dispatch);

    return (
      <div className={styles.friendListApp}>
        <h1>The FriendList</h1>
        <AddFriendInput addFriend={actions.addFriend} />
        <FriendList friends={friendsById} actions={actions} />
      </div>
    );
  }
}
```
We use bindActionCreators to wrap our action creators with a dispatch call. The goal is to pass action creators to our other components without giving them the dispatch object (keeping them "dumb").

What happens next is classic React stuff. We bind the functions to onClick, onChange or onKeyDown properties to handle user interactions.
If you're interested in how to do this, you can refer to the full code.
At this point, you get the magic feeling of a working redux/react application. As displayed in the GIF, you are logging all the actions and can play with the time travel. Developing is so nice when you can do some actions, encounter a bug, rewind, fix it and replay the sequence with the bug corrected.

## Wrapping up

That's all for today, I hope this got you started and you've enjoyed playing with Redux. If you want get more involved, join the community on Reactiflux.

In future episode I want to discuss a lot more stuff regarding redux, like integrating async actions, dealing with sequences of actions, testing and playing with react-router and immutable.js
Let me know if you have any feedback.

{% include footer.md %}
