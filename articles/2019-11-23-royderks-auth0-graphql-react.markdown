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
cd auth-graphql-app && yarn start
```

The command `yarn start` will run the start script of Create React App to compile and run your code on a local development server, which you can see by opening [http://localhost:3000](http://localhost:3000) in your browser. On this page, you'll see the first version of your application. The application at this point will look like this:

![create react app](articles/initial-create-react-app.jpg)

Before you connect with the GraphQL server and create new components to display the responses from the server, you need to know how the Create React App project is structured. Let's start with the file `src/index.js`, which is the entry point of your React application, in this file a component called `App` is rendered by `ReactDOM`. This means that all the components that you'd like to render in the browser must be reachable from that component that's defined in the the file `src/App.js`. To make you start with a "fresh" applicationn, let's delete some of the code in this file, so you'd end up with the following content for `src/App.js`:


```js
import React from 'react';

function App() {
  return "Hello GraphQL!";
}

export default App;
```

After changing the file above, the application will render the text *"Hello GraphQL"* in the browser. Next to this change you can also delete the following files from the project as you won't be needing these:

- 


​
## Using GraphQL with Apollo
### Setup Apollo Client with React
*This section will show how to set up Apollo Client to connect with the GraphQL server `
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