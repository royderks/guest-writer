---
layout: post
title: Developing Secure Web Applications with React and GraphQL
metatitle: Developing Secure Web Applications with React and GraphQL
description: "Develop a secure web application that handles authentication and authorization using React and GraphQL."
metadescription: "Develop a secure web application that handles authentication and authorization using React and GraphQL."
date: 2019-11-23 20:12
category: Developers, Tutorial, React, GraphQL
post_length:
auth0_aside: true
community_topic_id:
author:
  name: "Roy Derks"
  url: "gethackteam"
  avatar: "https://twitter.com/gethackteam/profile_image?size=original"
design:
  illustration: https://cdn.auth0.com/blog/illustrations/ambassador-in-the-moon.png
tags:
  - react
  - graphql
  - apollo
related:
  - https://auth0.com/blog/build-and-secure-a-graphql-server-with-node-js/
---

**TL;DR:** The introduction of GraphQL in 2015 changed how applications can request information from an API, and how that information flows through these applications. No longer do you have to filter large chunks of data to display the correct information to your users, but instead by using GraphQL you can specifically request whatever information you want to show them. React is a great library to create web applications with ease, and handle side effects like authentication using the lifecycles it provides.

This post will show how you can query and mutate data from a GraphQL server and display this information in a React application, and secure this application using Auth0.

## Prerequisites

Before you continue reading this tutorial, you'll need to make sure that you've `node` and `npm` installed on your machine. If you don't have these installed yet, you can find the installation instructions [here](https://nodejs.org/en/download/).

This post will be using a GraphQL server that is set up to handle authentication and authorization with Auth0, the code for this server can be found [here](https://github.com/auth0-blog/auth0-graphql-server). To run this server you need to follow the steps in the _Getting started_ section README of that project, including adding your Auth0 information. If you don't have an Auth0 account yet, you can <a href="https://auth0.com/signup" data-amp-replace="CLIENT_ID" data-amp-addparams="anonId=CLIENT_ID(cid-scope-cookie-fallback-name)">register one for free here</a>.

Also, I'm assuming you already have prior knowledge about JavaScript and React and know how you can create components and update state. If you haven't

# Developing Secure Web Applications with React and GraphQL

Web applications have rapidly increased in complexity as more and more technologies and tools for the web are available nowadays. One of these technologies is React, a JavaScript library for creating Single-Page Applications (SPA) or mobile applications. Another technology that has arisen the past few years is GraphQL, which simplifies how you can query and mutate data over HTTP. When used together these technologies let you create future-proof web applications that can be secured using Auth0. In this post, you'll learn how to develop a secure web application with React and GraphQL using Apollo. Not only will this application handle authentication, logged in users can have different authorization levels that define the actions they can take in the application.
​

## What is GraphQL?

In short: GraphQL is a query language for APIs that lets you query and mutate data over HTTP. It was publicly released by Facebook in 2015 to help them provide their (mobile) applications with just the data they need, instead of sending "raw" data over REST that needs to be filtered in the frontend. If you feel like you need to learn more about GraphQL, you can read a longer description in the [first part](https://github.com/auth0-blog/auth0-graphql-server) of this series about GraphQL.
​

## Creating a SPA with React

In this first section, you'll set up a basic React application with [Create React App](https://github.com/facebook/create-react-app). As mentioned before React is a library that helps you create modern web applications, and provides you with building blocks to handle more complex concepts like state management.

### Getting started with React

To render React in your browsers, you need to compile it first. For this, you'd need to set up a module builder like Webpack and something to compile your JavaScript code like Babel. However, when you create a new React application with Create React App you no longer need to configure these manually. Installing Create React App globally on your machine and initiate a new project can be done by running the following command from your terminal:

```
npx create-react-app auth-graphql-app
```

This command will start the installation process of Create React App and create a new directory called `auth-graphql-app` that holds the boilerplate code for your React application. You can move into this directory and start the application by running:

```
cd auth-graphql-app && npm start
```

The command `npm start` will run the start script of Create React App to compile and run your code on a local development server, which you can see by opening [http://localhost:3000](http://localhost:3000) in your browser. On this page, you'll see the first version of your application. The application at this point will look like this:

![create react app](articles/initial-create-react-app.jpg)

The application you'll built in this post will be an application that displays a list of events and makes it possible to manage the attendants of those events. Therefore this application will consists of two pages, the page to list the events and a page to edit the events. Before you connect with the GraphQL server and create new components to display the responses from the server, you need to know how the Create React App project is structured. Let's start with the file `src/index.js`, which is the entry point of your React application, in this file a component called `App` is rendered by `ReactDOM`.

This means that all the components that you'd like to render in the browser must be reachable from that component that's defined in the the file `src/App.js`, including routing. Routing in React (or other Single-Page Applications) works differently from routing in regular websites, and requires the usage of a library called `react-router-dom`. This package can be installed from npm by running the command below from your terminal:

```
npm install react-router-dom
```

With `react-router-dom` you can import a `Router` component that will be used by your application to dynamically change between pages, without having to refresh the page. This `Router` component must be added at the top level in `src/index.js`, where it must wrap the `App` component:

```js
// src/index.js

import React from "react";
import ReactDOM from "react-dom";
import { BrowserRouter as Router } from "react-router-dom";
import App from "./App";
import * as serviceWorker from "./serviceWorker";

ReactDOM.render(
  <Router>
    <App />
  </Router>,
  document.getElementById("root")
);

...

```

The routes for your application can be defined in the `src/App.js` file, where you must delete all the code that's currently in this file and replace it with the following:

```js
import React from "react";
import { Switch, Route } from "react-router-dom";

function App() {
  return (
    <Switch>
      <Route path="/event/:id">Event</Route>
      <Route path="*">All Events</Route>
    </Switch>
  );
}

export default App;
```

After changing the file above you've created two routes for your application, where the `Switch` component first tries to find the route for a single event and otherwise directes to the _All Events_ route. The route for a single event can be found by visiting in example `http://localhost:3000/event/123` Later on in this tutorial you'll add the data to these routes.

Next to creating the routes you can also delete the following files from the project as you will no longer be needing these:

```
src
|-- App.css
|-- App.test.js
|-- src/index.css
|-- logo.svg
|-- src/setupTests.js
```

The project is now ready to be connected to the GraphQL server for which [Apollo](https://www.apollographql.com/docs/react/) will be used in the next section.
​

## Using GraphQL with Apollo

The data for the application you create in this post is returned by the GraphQL server that was described in the Prerequisites section of this post. After you've followed the instructions in the _Getting started_ section of that project's README, including adding your Auth0 information, the GraphQL server will become available at `http://localhost:4000/graphql` and an interactive playground at `http://localhost:4000/graphqplayground`. Using this GraphQL Playground interface you can inspect the schema of this server or send documents containing queries and/or mutations to it. An example of a query that can be handled by this GraphQL server is:

```
query {
  events {
    title
    date
    attendants {
      name
    }
  }
}
```

The response of this query will be the full list events including its `title`, `date` and the `name` of every attendant of the event. This response will be in JSON and looks like the following:

```json
{
  "data": {
    "events": [
      {
        "title": "GraphQL Introduction Night",
        "date": "2019-11-06T17:34:25+00:00",
        "attendants": [
          {
            "name": "Peter"
          },
          {
            "name": "Kassandra"
          }
        ]
      },
      {
        "title": "GraphQL Introduction Night #2",
        "date": "2019-11-06T17:34:25+00:00",
        "attendants": [
          {
            "name": "Kim"
          }
        ]
      }
    ]
  }
}
```

As mentioned before you can send documents to the GraphQL server over plain HTTP, but also by using a package like [Apollo](https://www.apollographql.com/docs/react/). With Apollo, you can connect with the GraphQL server, handle sending documents to the server, and enable caching for data retrieved from the GraphQL server. How to set up Apollo Client to work with React will be shown in the next part of this section.

### Setup Apollo Client with React

Before you can send documents to the GraphQL server you need to set up the connection with the server, which can be done by adding the package `apollo-boost`. This package configures a GraphQL client for your application with Apollo's recommended settings for caching, state management and error handling. Next to `apollo-boost`, you also need to install the packages `@apollo/react-hooks` and `graphql`. With `@apollo/react-hooks` you can handle queries and mutations from your React components, while `graphql` is needed to use GraphQL's query language inside your application. To add these packages to your application run the following command from your terminal:

```
npm install apollo-boost @apollo/react-hooks graphql
```

In the file `src/index.js` you can subsequently import the method `ApolloClient` from `apollo-boost` that will create the GraphQL client for you. This method needs the endpoint to the GraphQL server that is running on `http://localhost:4000/` to get you started, and by default the client expects the GraphQL server to be running on the `/graphql` endpoint. Creating the client is done by making the changes below to `src/index.js`:

```js
// src/index.js
import React from "react";
import ReactDOM from "react-dom";
import { BrowserRouter as Router } from "react-router-dom";
import ApolloClient from "apollo-boost";
import App from "./App";
import * as serviceWorker from "./serviceWorker";

const client = new ApolloClient({
  uri: "https://48p1r2roz4.sse.codesandbox.io"
});

ReactDOM.render(
  <Router>
    <App />
  </Router>, 
  document.getElementById("root")
);

...

```

But creating a client isn't sufficient to connect your React application to the GraphQL server, as you also need to create a Provider that wraps your application and makes it possible to access the client from everywhere in your project. This Provider uses the Context API from React, about which I [previously wrote a post](https://auth0.com/blog/handling-authentication-in-react-with-context-and-hooks/) on the Auth0 blog in case you aren't familiar with it.

You can create a Provider by importing `ApolloProvider` from `@apollo/react-hooks` in the file `src/index.js` and pass the client to it. The `ApolloProvider` must be placed inside the `Router` to wrap the `App` component that is rendered by `ReactDOM` by making the following changes:

```js
// src/index.js
import React from "react";
import ReactDOM from "react-dom";
import { BrowserRouter as Router } from "react-router-dom";
import ApolloClient from "apollo-boost";
import ApolloProvider from "@apollo/react-apollo";
import App from "./App";
import * as serviceWorker from "./serviceWorker";

const client = new ApolloClient({
  uri: "http://localhost:4000"
});

ReactDOM.render(
  <Router>
    <ApolloProvider client={client}>
      <App />
    </ApolloProvider>
  </Router>,
  document.getElementById("root")
);
```

From any component that's nested within `ApolloProvider` you can now send documents with queries or mutations using `@apollo/react-hooks`, which will be demonstrated in the next part of this section.

### Query the GraphQL Server with Apollo

By wrapping the `App` component with `ApolloProvider` you can now use the methods `useQuery` and `useMutation` from `@apollo/react-hooks` to request and mutate data from the GraphQL server. These methods use the [Hooks pattern](https://www.apollographql.com/docs/react/api/react-hooks/) of React and take a GraphQL document as a parameter.

First, let's add a list of all the events to the application by sending a document with a query to the GraphQL server using `useQuery`. This should be done from a new React component which you can create in the directory `src` and call it `Events.js` with the following command:

```bash
touch src/Events.js
```

The query for getting the events has been mentioned in this post before, and must be added to this file together with the `useQuery` Hook:

```js
// src/Events.js
import React from "react";
import { gql } from "apollo-boost";
import { useQuery } from "@apollo/react-hooks";

const GET_EVENTS = gql`
  query getEvents {
    events {
      id
      title
      date
    }
  }
`;

function Events() {
  const { loading, data, error } = useQuery(GET_EVENTS);

  if (loading) return "Loading...";
  if (error) return "Something went wrong...";

  return (
    <div>
      <h1>Events</h1>
      <ul>
        {data.events.map(({ id, title, date }) => (
          <li key={id}>
            {title} ({date})
          </li>
        ))}
      </ul>
    </div>
  );
}

export default Events;
```

The query `getEvents` is passed to the `useQuery` Hook that returns the variable `loading` when it gets called, after resolving the Hook will return either a `data` object with the events or an `error` object. If all goes well, the `data` object contains an array with the event which you can iterate over to return a list of all the events.

To see the list of events in your browser you also need to import the `Events` component in the file `src/App.js` and have it returned by the `*` routes in your `App` component. You do this by changing:

```js
// src/App.js
import React from "react";
import { Switch, Route } from "react-router-dom";
import Events from "./Events";

function App() {
  return (
    <Switch>
      <Route path="/event/:id">Event</Route>
      <Route path="*">
        <Events />
      </Route>
    </Switch>
  );
}

export default App;
```

If you open the browser again, you can see that the list of events is being displayed together with the `title` and `date` of each event. This page could use some styling, which can be done in several ways with React. One of the easiest ways to add styling to a React component is by using inline style attributes, that require properties to be written in camel case. Let's see how this works by adding inline styling to the components in the file `src/Events.js`:

```js
// src/Events.js

...

function Events() {
  const { loading, data, error } = useQuery(GET_EVENTS);

  if (loading) return "Loading...";
  if (error) return "Something went wrong...";

  return (
    <div style={{ fontFamily: "Helvetica" }}>
      <header
        style={{
          display: "inline-block",
          width: "100%",
          backgroundColor: "lightBlue",
          padding: "10 20px",
          textAlign: "center",
          borderRadius: "5px"
        }}
      >
        <h1 style={{ color: "white" }}>Events</h1>
      </header>
      <ul style={{ listStyle: "none", width: "100%", padding: "0" }}>
        {data.events.map(({ id, title, date }) => (
          <li
            key={id}
            style={{
              backgroundColor: "lightGrey",
              marginBottom: "10px",
              padding: "10px",
              borderRadius: "5px"
            }}
          >
            <h2>{title}</h2>
            <span style={{ fontStyle: "italic" }}>{date}</span>
          </li>
        ))}
      </ul>
    </div>
  );
}

export default Events;
```

But having just a list of all the events is not sufficient, as you also want to display the attendants of that event. However, this information is private and requires you to send a valid JWT along with your query. This JWT can be retrieved by sending a request to the Auth0 authentication service, which you'll do in the next section of this post.

## Securing a React app

To secure your React application Auth0 is used, just as it was used to secure the GraphQL server that you're using for this post. By sending a request to the Auth0 authentication service with your credentials you'll retrieve a JWT that can be validated by the GraphQL server.

### Handle Authentication

Sending a request to Auth0 can be done by using the package [`@auth0/auth0-spa-js`](https://auth0.com/docs/quickstart/spa/react/01-login) together with your Auth0 `Domain` and `Client ID`. To get these values you need to create a new Single-Page Application on the [Application Settings](https://manage.auth0.com/#/applications/) page in the Auth0 dashboard. Make sure to use the same Auth0 account as you did to set up the GraphQL server. To proceed you need to install both `@auth0/auth0-spa-js` from npm:

```
npm install @auth0/auth0-spa-js
```

After installing this package you must add a new file called `src/react-auth0-spa.js` in your project, by running:

```bash
touch src/react-auth0-spa.js
```

And place the following code inside:

```js
// src/react-auth0-spa.js
import React, { useState, useEffect, useContext } from "react";
import createAuth0Client from "@auth0/auth0-spa-js";

const DEFAULT_REDIRECT_CALLBACK = () =>
  window.history.replaceState({}, document.title, window.location.pathname);

export const Auth0Context = React.createContext();
export const useAuth0 = () => useContext(Auth0Context);
export const Auth0Provider = ({
  children,
  onRedirectCallback = DEFAULT_REDIRECT_CALLBACK,
  ...initOptions
}) => {
  const [isAuthenticated, setIsAuthenticated] = useState();
  const [user, setUser] = useState();
  const [auth0Client, setAuth0] = useState();
  const [loading, setLoading] = useState(true);
  const [popupOpen, setPopupOpen] = useState(false);

  useEffect(() => {
    const initAuth0 = async () => {
      const auth0FromHook = await createAuth0Client(initOptions);
      setAuth0(auth0FromHook);

      if (window.location.search.includes("code=")) {
        const { appState } = await auth0FromHook.handleRedirectCallback();
        onRedirectCallback(appState);
      }

      const isAuthenticated = await auth0FromHook.isAuthenticated();

      setIsAuthenticated(isAuthenticated);

      if (isAuthenticated) {
        const user = await auth0FromHook.getUser();
        setUser(user);
      }

      setLoading(false);
    };
    initAuth0();
    // eslint-disable-next-line
  }, []);

  const loginWithPopup = async (params = {}) => {
    setPopupOpen(true);
    try {
      await auth0Client.loginWithPopup(params);
    } catch (error) {
      console.error(error);
    } finally {
      setPopupOpen(false);
    }
    const user = await auth0Client.getUser();
    setUser(user);
    setIsAuthenticated(true);
  };

  const handleRedirectCallback = async () => {
    setLoading(true);
    await auth0Client.handleRedirectCallback();
    const user = await auth0Client.getUser();
    setLoading(false);
    setIsAuthenticated(true);
    setUser(user);
  };
  return (
    <Auth0Context.Provider
      value={{
        isAuthenticated,
        user,
        loading,
        popupOpen,
        loginWithPopup,
        handleRedirectCallback,
        getIdTokenClaims: (...p) => auth0Client.getIdTokenClaims(...p),
        loginWithRedirect: (...p) => auth0Client.loginWithRedirect(...p),
        getTokenSilently: (...p) => auth0Client.getTokenSilently(...p),
        getTokenWithPopup: (...p) => auth0Client.getTokenWithPopup(...p),
        logout: (...p) => auth0Client.logout(...p)
      }}
    >
      {children}
    </Auth0Context.Provider>
  );
};
```

This file sets up the connection with Auth0 and returns a Provider called `Auth0Provider` and a Hook. This Provider is similar to `ApolloProvider` and needs your Auth0 credentials, making it possible to use the `useAuth0` Hook to connect with Auth0 from any components that are nested inside.

Before adding the `Auth0Provider` to your project you need to store the `Domain` and `Client ID` somewhere safe, like a local environment file. When you created an application using Create React App, you can create a `.env` in the root folder of your project and use the constants in this file from the `process.env` variable. Important to know is that these constants need to be prefixed with `REACT_APP_`. From the root directory of the project you need to run this command to create the `.env` file:

```bash
touch .env
```

In this file you can place the following code:

```
REACT_APP_AUTH0_DOMAIN=YOUR_AUTH0_DOMAIN
REACT_APP_AUTH0_CLIENT_ID=YOUR_CLIENT_ID
```

The values of `YOUR_AUTH0_DOMAIN` and `YOUR_CLIENT_ID` must be replaced by the values from your Auth0 React "Quick Start" page as follows:

- The value of `AUTH0_DOMAIN` is the value of the `issuer` object property from the code snippet, without the protocol, `https://`, the quotation marks, and the trailing slash. It follows this format `YOUR-AUTH0-TENANT.auth0.com`.
- The value of `CLIENT_ID` is a unique public identifier for your application. Although it's a public identifier, it’s recommended to not make it easily guessable by third parties.

Next to these credentials you need to set the _Callback URL_, _Logout URL_ and _Allowed Web Origins_ for your application on the [Application Settings](https://manage.auth0.com/#/applications/) page in the Auth0 dashboard. These values must be equal to the address where your React application is running, which is `http://localhost:3000`.

You can now restart the development server of Create React App by running `npm install` again. Your Auth0 credentials should now be available to use in the application to set up `Auth0Provider` in the file `src/index.js`, and be placed inside the `ApolloProvider` component like this:

```js
// src/index.js
import React from "react";
import ReactDOM from "react-dom";
import ApolloClient from "apollo-boost";
import { ApolloProvider } from "@apollo/react-hooks";
import { BrowserRouter as Router } from "react-router-dom";
import App from "./App";
import { Auth0Provider } from "./react-auth0-spa";
import * as serviceWorker from "./serviceWorker";

const client = new ApolloClient({
  uri: "http://localhost:4000/graphql"
});

ReactDOM.render(
  <Router>
    <ApolloProvider client={client}>
      <Auth0Provider
        domain={process.env.REACT_APP_AUTH0_DOMAIN}
        client_id={process.env.REACT_APP_AUTH0_CLIENT_ID}
        redirect_uri={window.location.origin}
      >
        <App />
      </Auth0Provider>
    </ApolloProvider>
  </Router>,
  document.getElementById("root")
);

...

```





### Handle Authorization

_This section will show how to handle different authorization levels and send this to the GraphQL server_
​

## Conclusion
