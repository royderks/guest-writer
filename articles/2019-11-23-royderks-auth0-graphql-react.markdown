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

This post will be using a GraphQL server that is set up to handle authentication and authorization with Auth0, the code for this server can be found [here](https://github.com/auth0-blog/auth0-graphql-server). To run this server you need to follow the steps in the *Getting started* section README of that project, including adding your Auth0 information. If you don't have an Auth0 account yet, you can <a href="https://auth0.com/signup" data-amp-replace="CLIENT_ID" data-amp-addparams="anonId=CLIENT_ID(cid-scope-cookie-fallback-name)">register one for free here</a>.

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

Before you connect with the GraphQL server and create new components to display the responses from the server, you need to know how the Create React App project is structured. Let's start with the file `src/index.js`, which is the entry point of your React application, in this file a component called `App` is rendered by `ReactDOM`. This means that all the components that you'd like to render in the browser must be reachable from that component that's defined in the the file `src/App.js`. To make you start with a "fresh" applicationn, let's delete some of the code in this file, so you'd end up with the following content for `src/App.js`:


```js
import React from 'react';

function App() {
  return "Hello GraphQL!";
}

export default App;
```

After changing the file above, the text *"Hello GraphQL"* will be visible in the browser meaning you have deleted all the code rendering the initial application. Next to this change you can also delete the following files from the project as you will no longer be needing these:

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
The project you'll create in this post will be an application that displays a list of events and makes it possible to manage the attendants of those events. The data for this application is returned by the GraphQL server that was described in the Prerequisites section of this post. After you've followed the instructions in the *Getting started* section of that project's README, including adding your Auth0 information, the GraphQL server will become available at `http://localhost:4000/graphql` and an interactive playground at `http://localhost:4000/graphqplayground`. Using this GraphQL Playground interface you can inspect the schema of this server or send documents containing queries and/or mutations to it. An example of a query that can be handled by this GraphQL server is:

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

As mentioned before you can send documents to the GraphQL server over plain HTTP, but also by using a package like [Apollo](https://www.apollographql.com/docs/react/). With Apollo, you can connect with the GraphQL server, handle queries and mutations, and enable caching for data retrieved from the GraphQL server. How to set up Apollo Client to work with React will be shown in the next part of this section.

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
import ApolloClient from "apollo-boost";
import App from "./App";
import * as serviceWorker from "./serviceWorker";

const client = new ApolloClient({
  uri: "https://48p1r2roz4.sse.codesandbox.io"
});

ReactDOM.render(<App />, document.getElementById("root"));

...

```

But creating a client isn't sufficient to connect your React application to the GraphQL server, as you also need to create a Provider that wraps your application and makes it possible to access the client from everywhere in your project. This Provider uses the Context API from React, about which I [previously wrote a post](https://auth0.com/blog/handling-authentication-in-react-with-context-and-hooks/) on the Auth0 blog in case you aren't familiar with it.

You can create a Provider by importing `ApolloProvider` from `@apollo/react-hooks` in the file `src/index.js` and pass the client to it. The `ApolloProvider` must wrap the `App` component that is rendered by `ReactDOM` by making the following changes:

```js
// src/index.js
import React from "react";
import ReactDOM from "react-dom";
import ApolloClient from "apollo-boost";
import ApolloProvider from "@apollo/react-apollo";
import App from "./App";
import * as serviceWorker from "./serviceWorker";

const client = new ApolloClient({
  uri: "http://localhost:4000"
});

ReactDOM.render(
  <ApolloProvider client={client}>
    <App />
  </ApolloProvider>,
  document.getElementById("root")
);
```

From any component that's nested within `ApolloProvider` you can now send documents with queries or mutations using the methods `useQuery` and `useMutation` from `@apollo/react-hooks`, which will be demonstrated in the next part of this section.

### Query the GraphQL Server with Apollo
*This section will show how to query the GraphQL server using Hooks from `react-apollo`
​
​
## Securing a React app
### Handle Authentication
*This section will show how to add authentication to a SPA using Auth0*
### Handle Authorization
*This section will show how to handle different authorization levels and send this to the GraphQL server*
​
## Conclusion