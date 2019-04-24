---
layout: post
title: "Handling Authentication in React with Context and Hooks"
description: "Identity Management in React can be quite confusing, as there are multiple ways you can handle the users' authentication data in your application. This tutorial will show how you can handle Identity Management in React, by creating a global state for your authentication details with Context and update these details with Hooks."
date: "2019-04-08 08:30"
author:
  name: "Roy Derks"
  url: "gethackteam"
  mail: "hello@hackteam.io"
  avatar: "https://twitter.com/gethackteam/profile_image?size=original"
tags:
  - javascript
  - reactjs
---

**TL;DR:** Briefly describe what this article is about and what the reader will achieve/learn after reading it. Please,
also provide the link to a GitHub repository that contains code related to this article.

# Handling Authentication in React with Context and Hooks
Identity Management for Single-Page Applications (SPAs), and more specifically React, can be quite confusing. Topics to consider are where to store the users’ tokens or how to handle their data when it flows through your application. This tutorial will show how you can use Auth0 in a Single-Page React application to handle Identity Management, by creating a global state for your authentication details with Context and updating these with Hooks.

## Prerequisites
Before continue reading this tutorial I’m going to assume you have `node` and `npm` installed on your machine, and are familiar with JavaScript - and preferably React. Also, you’d need an Auth0 Developer account. If you don’t have one, please create one [here](https://auth0.com/signup) before continuing.

## State Management in React
As mentioned before, this tutorial will show you how to store the authentication details for your users in a global state. Having a (global) state is something that is important for SPAs, as it can be seen as a snapshot for your application at a given moment. How to handle this state is something that has evolved a lot in React the past years. An important distinction to make is that there are generally two types of states in React applications, a local state, and a global state. Where the local state is often limited to information about specific pages or components, the global state represents the state of the entire application. For this local state React already had a solution integrated to its core, but not for the global state. In order to have a global state that could be mutated from all your components, you had to install packages like Redux to handle this for you. However, with the release of React version 16.3, a renewed version of the Context API was introduced that makes having a global state possible.

## Introduction to Context
The Context API has been around for a long time but was always marked as an experimental feature. Although it was marked as experimental, popular packages like Redux and React Router made use of it. With Context, it becomes easier to access props in different components, without having to explicitly pass them down on each level. Using a Higher-Order Component (HOC) you can retrieve props from components that are placed lower in your component tree.

You can create a new Context by using the `React.createContext()` function, that creates an object that has a Provider and a Consumer component. A Context would consist of these two components, that should be nested like this:

```js
import React from ‘react’;

const Context = React.createContext();

render() {
    return (
        <Context.Provider>
            <Context.Consumer>
                { /* custom components */ }
            </Context.Consumer>
        </Context.Provider>
    );
}
```

### Passing Props
In order to pass props within the Context, you must use both the Provider and the Consumer component. This Provider component is the place where you’d pass a prop called `value` to, which you can subsequently consume within the Consumer component. Using the example of Context above, props from the Provider would be passed to the Consumer in this way:

```js
<Context.Provider value={/* object */}>
    <Context.Consumer>
        { props => /* custom components */ }
    </Context.Consumer>
</Context.Provider>
```

### Updating Context
As the Context Provider takes an object to set the value that is passed to the Consumer, this value can be mutated at the level of the Provider. For this you can use React’s local state, simply by passing an initial state value to this Provider and a function that mutates this state by using the `setState()` method. The implementation for this would like something like the following code example:

```js
import React from ‘react’;

const Context = React.createContext();

export default class App extends React.Component {
    state = {
        value: ‘foo’
    }

    updateState(value) {
        this.setState({ value });
    }

    render() {
        return (
            <Context.Provider value={{ value: this.state.value, updateState: this.updateState }}>
                <Context.Consumer>
                    { props => /* custom components */ }
                </Context.Consumer>
            </Context.Provider>
        );
    }
}
```

## Introduction to Hooks
Since the official introduction of Hooks in React v16.8.0, the previous code example can be drastically simplified. Multiple predefined Hooks as `useState` and `useReducer` were introduced; along with a possibility to create your own, custom Hooks. One of the Hooks with the biggest use case would be the `useState` Hook, that gives you a shortcut to local state-management. Let’s apply it to the code example above:

```js
import React from ‘react’

const Context = React.createContext();

const initialState = {
    value: ‘foo’
}

const [state, updateState] = React.useState(initialState);

const App = () => (
    <Context.Provider value={{ value: state.value, updateState }}>
        <Context.Consumer>
            { props => // … custom components }
        </Context.Consumer>
    </Context.Provider>
);

export default App;
```

### useReducer Hook
The setup above will work completely fine if you’re dealing with small(er) components and only need to update the state locally. For bigger applications or more advanced use cases, you could use the `useReducer` Hook instead. By using the `useReducer` Hook you can update an initial object based on whatever type of event would be fired. The difference with the `useState` Hook is that this initial object is unrelated to the built-in local state-management of React, so it can be applied globally in your application.

If you’ve ever used global state-management libraries like Redux, this might sound familiar to you. But if you’re not, this is how the `useReducer` Hooks is applied in a similar fashion as Redux:

```js
const initialState = {
    value: ‘foo’
}

reducer(state, action) {
    switch(action.type) {
        case ‘updateValue’:
            return {
                …state,
                value: action.payload
            }
        default:
            return state
    }
}

const [state, dispatch] = React.useReducer(reducer, initialState);
```

In the example above you can see that the `useReducer` Hook takes the constants `reducer` and `initialState` as a parameter. The output is the returned value by the reducer and a function to invoke the reducer function, that takes just the action as a parameter and inherits the current state. In this case, the reducer looks for a field called `type` in the action and updates the state with the value of `payload` from that action whenever it’s type equals `updateValue`. If not, the reducer will just return the current value of the state. This function to update the field `value` of initialState, can be called by doing the following:

```js
{
    () => dispatch({ type: "updateValue", payload: "new value" });
}
```

The action object that is used as a parameter for the dispatch function includes the fields `type` and `payload`, which the reducer will now use to update the field `value` of `initialState`.


## Updating Context with Hooks
With the help of this `useReducer` Hook you can also set the value that's passed to the Context Provider, and mutate this value if you’d also pass the dispatch function to the Provider. In order to demonstrate how to achieve this, let’s create a new project in the form of an application that can be used to manage a fictional event.

### Create a new React project
The first step is to create a new React project using Create React App, which provides you with a suitable configuration for most React applications. If you’re unfamiliar with the use of Create React App, please have a look at the [documentation](https://reactjs.org/docs/create-a-new-react-app.html#create-react-app) that explains what it is and how it can be used.

A new application can be created by executing the following command from your terminal, where `new-project` can be replaced with the name of your project:

```bash
npx create-react-app new-project
```

You can now move into the directory `new-project` and open the project in your code editor. In the next section, you'll make changes to the files that are placed in the `src` directory.

### Add a Context Provider and Consumer
Second, the boilerplate code that is provided in the file `src/App.js` can be removed and replaced with the following:

```js
// src/App.js
import React from "react";

const MeetupContext = React.createContext();

const initialState = {
    title: 'Auth0 Online Meetup',
    date: Date()
}

const App = () => (
    <MeetupContext.Provider value={ initialState }>
        <MeetupContext.Consumer>
                { props => (
                <div>
                    <h1>{ props.title }</h1>
                    <span>{ props.date }</span>
                </div>
            )}
        </MeetupContext.Consumer>
    </MeetupContext.Provider>
)

export default App
```

The code block above represents an implementation of the Context API in the form of `MeetupContext`, which has a Provider that takes the object initialState as value and a Consumer that displays this information.

Also, you can remove the files `src/App.test.js`, `src/App.css` and `src/Logo.svg` as testing and styling of your code will be skipped in this tutorial.

If you'd now open the browser on `http://localhost:3000/` you can see the first version of your application, which displays a title and the current date and time.

Let's extend this information with some attendees for this fictional Meetup. Therefore you need to add a new field to the initialState object, which you can call `attendees` and assign an array with random names to it.

```js
// src/App.js
import React from "react";

const MeetupContext = React.createContext();

const initialState = {
    title: 'Auth0 Online Meetup',
    date: Date(),
    attendees: ['Bob', 'Jessy', 'Christina', 'Adam']
}
...
```

And have the Consumer display this list of attendees:

```js
// src/App.js
...
const App = () => (
    <MeetupContext.Provider value={ initialState }>
        <MeetupContext.Consumer>
                { props => (
                <div>
                    <h1>{ props.title }</h1>
                    <span>{ props.date }</span>
          <div>
            <h2>{`Attendees (${ props.attendees.length })`}</h2>
            { props.attendees.map(attendant => <li>{ attendant }</li>) }
          </div>
                </div>
            )}
        </MeetupContext.Consumer>
    </MeetupContext.Provider>
)

export default App
```

The application is now able to display not only the Meetup information but also a list of attendees. Let's continue by adding the functionality to subscribe to this Meetup.

### Nested Context

Before you can actually subscribe to this Meetup, a new Context for the user should be added. You can nest multiple Context Providers and Consumers, making it possible to access props globally.

```js
// src/App.js
import React from "react";

const MeetupContext = React.createContext();
const UserContext = React.createContext();
...
```

As the Context for the user also needs an initial value, the current `initialState` constant with the value for the `MeetupContext` can be extended. You can do this by moving the fields about the Meetup to a new nested object called `meetup`. Also, create a new one for the user with just the field `name`.

```js
// src/App.js
import React from "react";

const MeetupContext = React.createContext();
const UserContext = React.createContext();

const initialState = {
    meetup: {
            title: 'Auth0 Online Meetup',
          date: Date(),
            attendees: ['Bob', 'Jessy', 'Christina', 'Adam']
      },
      user: {
            name: 'Roy'
      }
}
...
```

You can now add the newly created `UserContext` that takes the `user` field from `initialState` as its value within the `render()` function, and specify that `MeetupContext` should use the `meetup` field from `initialState` as its value. `MeetupContext` is placed within `UserContext`, as the Context for the user might be needed in other components (like a profile page) in the future as well.

```js
// src/App.js
...
const App = () => (
    <UserContext.Provider value={initialState.user}>
    <MeetupContext.Provider value={initialState.meetup}>
            <MeetupContext.Consumer>
                    { props => (
                        ...
```

The Consumer for the user can be placed just above `MeetupContext`, and should return the value from the Provider not as `props` but as the variable `user`. Otherwise, it would lead to a duplicate declaration of the constant `props`. For clarity, let's also rename the return value from the Meetup's Consumer.

```js
// src/App.js
...
const App = () => (
    <UserContext.Provider value={initialState.user}>
    <UserContext.Consumer>
      { user => (
        <MeetupContext.Provider value={ initialState.meetup }>
                <MeetupContext.Consumer>
                    { meetup => (
                        <div>
                            <h1>{ meetup.title }</h1>
                            <span>{ meetup.date }</span>
                  <div>
                    <h2>{`Attendees (${ meetup.attendees.length })`}</h2>
                    { meetup.attendees.map(attendant => <li>{ attendant }</li>) }
                  </div>
                        </div>
                    )}
                </MeetupContext.Consumer>
            </MeetupContext.Provider>
        )}
      </UserContext.Consumer>
    </UserContext.Provider>
);

export default App
```

With both the Context for the user and Meetup returned in the `render()` function, you can have this user subscribe to this Meetup in the next step.

### Implement useReducer Hooks

When this user subscribes to this Meetup, you want the user's name to be added to the list of attendees. Therefore the initial value for `MeetupContext.Provider` should be mutated and as mentioned before you can use the `useReducer` Hook to accomplish this.

You can start by creating a new component that returns the Provider for MeetupContext and takes both the user Context and any children as a prop. In this component, you can add the `useReducer` Hook, and extend the value for MeetupContext with the `dispatch` function that is returned by the Hook.

```js
// src/App.js
...
const MeetupContextProvider = ({ user, ...props }) => {
  const [state, dispatch] = React.useReducer(reducer, initialState.meetup);

  return (
    <MeetupContext.Provider value={{
        ...state,
        handleSubscribe: () => dispatch({ type: "subscribeUser", payload: user.name }),
    handleUnSubscribe: () => dispatch({ type: "unSubscribeUser", payload: user.name })
      }}>
      {props.children}
    </MeetupContext.Provider>
  );
}

const App = () => (
    ...
```

As you can see this `useReducer` Hook also takes the reducer as a parameter, which you can add directly above the `MeetupContextProvider` component. This reducer takes both the state and the received action as parameters, when it receives an action with the type `subscribeUser` it will add the payload field of that action to the attendees' array.

```js
// src/App.js
...
const reducer = (state, action) => {
    switch (action.type) {
            case 'subscribeUser':
                  return {
                        ...state,
                        attendees: [...state.attendees, action.payload],
                subscribe: true
                  }
            default:
                  return state
      }
}

const MeetupContextProvider = (props) => {
    ...
```

And you can do the same to unsubscribe the user for the Meetup:

```js
// src/App.js
const reducer = (state, action) => {
    switch (action.type) {
            case 'subscribeUser':
                  return {
                        ...state,
                        attendees: [...state.attendees, action.payload],
                subscribed: true
                  }
        case 'unSubscribeUser':
                  return {
                        ...state,
                        attendees: state.attendees.filter(attendee => attendee !== action.payload),
                subscribed: false
              }
            default:
                 return state
      }
}
```

Then of course as this action is now available within the Consumer for the Meetup, a button that invokes this function should be added to the application. Also, don't forget to send the user Context to MeetupContextProvider as this is needed to add the user to the list of attendees.

```js
// src/App.js
...
const App = () => (
    <UserContext.Provider value={initialState.user}>
    <UserContext.Consumer>
      { user => (
        <MeetupContextProvider user={ user }>
                <MeetupContext.Consumer>
                    { meetup => (
                        <div>
                            <h1>{ meetup.title }</h1>
                            <span>{ meetup.date }</span>
                  <div>
                    <h2>{`Attendees (${ meetup.attendees.length })`}</h2>
                    { meetup.attendees.map(attendant => <li>{ attendant }</li>) }
                    <p>
            {
                !meetup.subscribed
                    ? <button onClick={ meetup.handleSubscribe }>Subscribe</button>
                    : <button onClick={ meetup.handleUnSubscribe }>Unsubscribe</button>
                    </p>
                  }
                  </div>
                        </div>
                    )}
                </MeetupContext.Consumer>
            </MeetupContextProvider>
        )}
      </UserContext.Consumer>
    </UserContext.Provider>
)

export default App
```

When you now view the application in your browser, the list of attendees should update once you click the `subscribe` button. Every time you click it the Context will update, which leads to a new render of the Consumer. As you don't want unauthenticated users to subscribe to the meetup, let's add authentication to the application.

### Add Authentication

As you're going to use Auth0 for authentication, make sure that you've created an application in the Auth0 dashboard. If so, you can find your application keys [here](https://manage.auth0.com/#/applications/YOUR_CLIENT_ID/settings); for this tutorial, you'll need the following keys:
- Client ID
- Domain

Also, you need to specify a callback and a return URL for your application. As your application is running on `port 3000` when used locally, you should set the allowed callback URL to `http://localhost:3000/?callback`.

In order to safely add authentication with Auth0, you need to install two packages from `npm`. The first one is `dotenv` that is used to add local environment variables to your application, which you need to install as a development dependency:

```bash
yarn add dotenv --dev
```

The second one is `auth0-js`, which is the official client-side JavaScript SDK from Auth0; you can install this package by executing the following command in your terminal:

```bash
yarn add auth0-js
```

With these two packages installed, let's store your application keys in a local environment file called `.env` - which you can create in the root directory of your project. Inside this file you can add environment variables for your application keys, which you should prefix with `REACT_APP_` so they can be used by your Create React App without having to explicitly import the `dotenv` package:

```
REACT_APP_AUTH0_DOMAIN=[DOMAIN]
REACT_APP_AUTH0_CLIENT_ID=[CLIENT ID]
```

Furthermore, you can now create the file containing the logic to handle authentication using `auth0-js` and your application's key. You do this by creating a new file called `Auth.js` in the `src` directory, and add the following code block to this file:

```js
// src/Auth.js
import auth0 from 'auth0-js'

export default class Auth {
  constructor() {
    this.auth0 = new auth0.WebAuth({
      domain: process.env.REACT_APP_AUTH0_DOMAIN,
      audience: `https://${process.env.REACT_APP_AUTH0_DOMAIN}/userinfo`,
      clientID: process.env.REACT_APP_AUTH0_CLIENT_ID,
      redirectUri: 'http://localhost:3000/?callback',
      responseType: 'id_token',
      scope: 'openid profile'
    });

    this.handleAuthentication = this.handleAuthentication.bind(this);
    this.signIn = this.signIn.bind(this);
  }

  signIn() {
    this.auth0.authorize();
  }

  handleAuthentication() {
    return new Promise((resolve, reject) => {
      this.auth0.parseHash((err, authResult) => {
        if (err) return reject(err);
        if (!authResult || !authResult.idToken) {
          return reject(err);
        }
        this.idToken = authResult.idToken;
        this.profile = authResult.idTokenPayload;
        // set the time that the id token will expire at
        this.expiresAt = authResult.idTokenPayload.exp * 1000;
        resolve();
      });
    });
  }
}
```

This Auth class has all the methods to handle authentication and store the JWT token that is returned by Auth0 in the client. Something that is needed to actually check if a user is authenticated or not.

This new class should be imported in `src/App.js`, and used to create a new object called `auth`:

```js
// src/App.js
import React from "react";
import Auth from "./Auth";

const auth = new Auth();

const MeetupContext = React.createContext();
const UserContext = React.createContext();

...
```

As these authentication functions are related to the user, these need to be added to `UserContext`. Just as you already did for `MeetupContext`, the Provider for the user should be split into a different component.

```js
// src/App.js
…
const UserContextProvider = (props) => {
      const [state, dispatch] = React.useReducer(reducer, initialState.user);

    auth.handleAuthentication().then(() => {
        dispatch({ type: 'loginUser', payload: true });
    });

      return (
            <UserContext.Provider
                  value={{
                        ...state,
                        handleLogin: auth.signIn
                  }}
            >
                  {props.children}
            </UserContext.Provider>
      );
}

const MeetupContextProvider = ({ user, ...props }) => {
    …
```

In the code block above the Context Provider for the user is created, which also uses the `useReducer` Hook. Next, two functions from the Auth0 client are used to either initialize the authentication process or to verify it has returned a token. The authentication can be initialized by invoking the `handleLogin` function that is available on the user’s Context. If successful the `handleAuthentication` function will dispatch an action that sets the user as verified in the reducer, which you can do by adding another `case statement` to the reducer:

```js
// src/App.js
const reducer = (state, action) => {
    switch (action.type) {
            case 'loginUser':
                  return {
                        ...state,
                        isAuthenticated: action.payload
              }
        …
```

Now you can make the changes to the return function, so the user can actually log in and subscribe to the Meetup:

```js
// src/App.js
…
const App = () => (
    <UserContextProvider>
    <UserContext.Consumer>
      { user => (
        <MeetupContextProvider user={ user }>
                <MeetupContext.Consumer>
                    { meetup => (
                        <div>
                            <h1>{ meetup.title }</h1>
                            <span>{ meetup.date }</span>
                  <div>
                    <h2>{`Attendees (${ meetup.attendees.length })`}</h2>
                    { meetup.attendees.map(attendant => <li key={attendant}>{ attendant }</li>) }
                    <p>
                      {
                                                user.isAuthenticated
                                                    ? !meetup.subscribed
                                                            ? <button onClick={ meetup.handleSubscribe }>Subscribe</button>
                                  : <button onClick={ meetup.handleUnSubscribe }>Unsubscribe</button>
                                                        : <button onClick={ user.handleLogin }>Login</button>
                      }
                    </p>
                  </div>
                        </div>
                    )}
                </MeetupContext.Consumer>
            </MeetupContextProvider>
        )}
      </UserContext.Consumer>
    </UserContextProvider>
);

export default App
```

When you try and visit the project in the browser, make sure to restart the development server as it might be that the environment variables aren’t picked up by Webpack otherwise.

With these final changes, you’ve created a React application that uses the Context API and Hooks to update the global state of an application. The global state is in this case both the information about the user and the Meetup.
