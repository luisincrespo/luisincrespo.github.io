---
title:  "Building a Simple NPM Module: React List Component"
categories: [javascript]
tags: [javascript, npm, react, webpack, flux]
---
Are you looking for a kick-start to guide you through the process of building a reusable [React][react] component?. Are you also wondering how to integrate the [Flux Architecture Pattern][flux] with this component and be able to publish your work as a public [npm][npm] module?. If your answer is yes, or you're just curious about what I'm talking about, this post is just for you.

## Overview

### What are we building ?
Through out this tutorial we'll get our hands on coding to create a simple [React][react] reusable component, that will consist on a list where you can add, edit, remove and search items. [Flux][flux] will be the pattern followed to manipulate the list's state.

We also want to publish our component as a reusable module, and here is where [npm][npm] comes into play, and we'll be using it from the beginning to also manage our dependencies.

To keep our code as modularized as possible, we're going to take advantage of [webpack][webpack] and the available [Babel][babel] loaders, to make use of [ES6's][es6] module syntax, and bundle everything into a single file.

So, here's a summary of the tools we'll be working with throughout the development process:
  * [npm][npm]
  * [React][react]
  * [Alt.JS][alt] (Javascript framework to work with [Flux][flux])
  * [webpack][webpack]
  * [Babel][babel]
  * [Git][git]
  * [Immutable.JS][immutable] (Javascript library to work with immutable data)

### What should you have experience with ?
[Javascript][javascript] and front-end development in general, of course.

It's important for you to be comfortable with the use of [Git][git], since no details about its usage will be present during this post.

### Disclaimer
This tutorial is not intended to be an in-depth guide on how to use any of the tools/patterns mentioned before. Quite the opposite, this is just a simple kick-start to guide you in the right direction so you can further develop your skills.

### In a hurry ?
If you'd like to just go and take a look at the final source code, check it out [here][react-list].

## Preparing things up
Make sure you have [Node.JS][node] and [npm][npm] installed. Use the latest stable releases.

### Initialize a [Git][git] repo
Create your project folder, we will be referencing it as `react-list/` throughout this tutorial. Initialize it as a [Git][git] repository and set its remote.

### Initialize with [npm][npm]
Run the following command in your project's root folder:

> npm init

This will prompt you to enter the following information:

* name
  - Module's name.
* version:
  - Module's [SemVer][semver] version number, it will default to *1.0.0*. We'll take care of this later.
* description:
  - Simply describes what your module is/does. Could be something like *React component for a list of items.* in this case.
* entry point:
  - This is the file that will be included when someone requires your module it will default to *index.js*. We'll take care of this later.
* test command:
  - The command that will be used for running tests. We won't be developing tests, so leave blank.
* git repository:
  - Your remote [Git][git] repo. If you've already set a remote, it will default to that one.
* keywords:
  - This lets people discover your package. Could be something like *list, items, react, react-component* in this case.
* author:
  - Your name.
* license:
  - Just to let people know what license you're using. Let's go with *MIT* (remember to include the license in your source code).

After you answer to all of the past prompts, the resulting `react-list/package.json` will be outputted, asking you if it is OK. If it is, just answer *yes* and we're done with this step.

### Add some structure
Too keep things simple, let's just create the folder `react-list/src` where all our source code will be present (in an organized manner). Also create the file `react-list/src/index.js`, that will serve as our entry point for [webpack][webpack].

### Configuring [webpack][webpack]
First of all, we need to add [webpack][webpack] as a development dependency for our module. This is simply done running the following command:

> npm install --save-dev webpack webpack-dev-server

Notice that we've also installed *webpack-dev-server* since we'll be using it to test our module locally. We also need [Babel][babel] and its pertinent loaders to be able to transpile [ES6][es6] and [JSX][jsx] code:

> npm install --save-dev babel-core babel-loader babel-preset-es2015 babel-preset-react

Now we can create our [webpack][webpack] configuration file:

`react-list/webpack.config.js`
{% highlight javascript %}
module.exports = {
  context: __dirname + '/src',

  entry: './index',

  output: {
    path: __dirname + '/dist',
    filename: "bundle.js"
  },

  module: {
    loaders: [
      {
        test: /\.jsx?$/,
        exclude: /node_modules/,
        loader: 'babel-loader',
        query: {
          presets: ['react', 'es2015']
        }
      }
    ]
  },

  resolve: {
    extensions: ['', '.js', '.jsx']
  }
}
{% endhighlight %}

Let's give a simple explanation for each one of this configuration options:

* context:
  - The directory where [webpack][webpack] will look for the entry file.
* entry:
  - The entry file that [webpack][webpack] will use for bundling.
* output:
  - path:
    + The path where the bundle will be saved.
  - filename:
    + The bundle's file name.
* module:
  - loaders:
    + Here we define loaders that will be applied for determinated files. In this case we're defining *babel-loader* to process any [JS][javascript] or [JSX][jsx] file.
* resolve:
  - extensions:
    + The file extensions that [webpack][webpack] will use for bundling.

### Bundling
Add the following script to `react-list/package.json`, so we can then use it to build our bundle:

`react-list/package.json`
{% highlight json %}
{
  .
  .
  .
  "scripts": {
    .
    .
    .
    "build": "webpack --config webpack.config.js --progress --colors"
  },
  .
  .
  .
}
{% endhighlight %}

Now we can run the following command:

> npm run build

We should have our first bundle built on `react-list/dist/bundle.js` (of course with no functionality yet).

[flux]: https://facebook.github.io/flux/
[react]: https://facebook.github.io/react/
[npm]: https://www.npmjs.com/
[webpack]: https://webpack.github.io/
[babel]: https://babeljs.io/
[es6]: http://es6-features.org/
[alt]: http://alt.js.org/
[git]: https://git-scm.com/
[immutable]: https://facebook.github.io/immutable-js/
[react-list]: https://github.com/luisincrespo/react-list
[node]: https://nodejs.org/
[semver]: http://semver.org/
[javascript]: https://developer.mozilla.org/en-US/docs/Web/JavaScript
[jsx]: https://jsx.github.io/
