---
title:  "Building a simple npm module"
categories: [javascript]
tags: [javascript, npm, react, webpack, flux]
---
Are you looking for a kick-start to guide you through the process of building a reusable [React][react] component?. Are you also wondering how to integrate the [Flux Architecture Pattern][flux] and be able to publish your work as a public [npm][npm] module?. If your answer is yes, or you're just curious about what I'm talking about, this post is just for you.

<!-- TOC  -->

## Table of contents

- [Overview](#overview)
	- [What are we building ?](#what-are-we-building-)
	- [What should you have experience with ?](#what-should-you-have-experience-with-)
	- [Disclaimer](#disclaimer)
	- [In a hurry ?](#in-a-hurry-)
- [Preparing things up](#preparing-things-up)
	- [Initialize a Git repo](#initialize-a-git-repo)
	- [Initialize with npm](#initialize-with-npm)
	- [Add some structure](#add-some-structure)
	- [Configuring webpack](#configuring-webpack)
	- [Bundling](#bundling)
	- [Dependencies](#dependencies)
		- [React](#react)
		- [Alt.JS and Immutable.JS](#altjs-and-immutablejs)
- [Start coding](#start-coding)
	- [Baby steps](#baby-steps)
		- [Alt.JS instance](#altjs-instance)
		- [List store](#list-store)
		- [Main component](#main-component)
		- [List component](#list-component)
		- [Webpack entry file](#webpack-entry-file)
		- [Testing locally](#testing-locally)
	- [Developing features](#developing-features)
		- [Setting initial list items via props](#setting-initial-list-items-via-props)
		- [Adding list items](#adding-list-items)
		- [Removing list items](#removing-list-items)
		- [Editing list items](#editing-list-items)
		- [Searching items on the list](#searching-items-on-the-list)
- [Publishing to npm](#publishing-to-npm)
	- [Creating a user](#creating-a-user)
	- [Updating package.json](#updating-packagejson)
	- [.npmignore](#npmignore)
	- [Build and publish](#publish)
- [End of tutorial](#end-of-tutorial)
  - [What next ?](#what-next-)

<!-- / TOC -->

<!-- OVERVIEW -->

## Overview

### What are we building ?
Through out this tutorial we'll get our hands on coding to create a simple [React][react] component, that will consist on a list where you can add, edit, remove and search items. [Flux][flux] will be the pattern followed to manipulate the list state.

We also want to publish our component as a reusable module, and here is where [npm][npm] comes into play, and we'll be using it from the beginning to also manage our dependencies.

To keep our code as modularized as possible, we're going to take advantage of [webpack][webpack] and the available [Babel][babel] loaders, to make use of [ES6][es6] module syntax, and bundle everything into a single file.

So, here's a summary of the tools we'll be working with throughout the development process:

  * [npm][npm]
  * [React][react]
  * [Alt.JS][alt] ([Javascript][javascript] framework to work with [Flux][flux])
  * [webpack][webpack]
  * [Babel][babel]
  * [Git][git]
  * [Immutable.JS][immutable] ([Javascript][javascript] library to work with immutable data)

### What should you have experience with ?
[Javascript][javascript] and front-end web development in general.

### Disclaimer
This tutorial is not intended to be an in-depth guide on how to use any of the tools/patterns mentioned before. Quite the opposite, this is just a simple kick-start to guide you in the right direction so you can further develop your skills.

No details about version control with [Git][git] will be given throughout the tutorial.

### In a hurry ?
If you'd like to just go and take a look at the final source code, check it out [here][react-list-0.1.0].

<!-- / OVERVIEW -->

<!-- PREPARING THINGS UP -->

## Preparing things up
Make sure you have [Node.JS][node] and [npm][npm] installed. Use the latest stable releases.

### Initialize a Git repo
Create your project folder, we will be referencing it as `react-list/` throughout this tutorial. Initialize it as a [Git][git] repository and set its remote.

### Initialize with npm
Run the following command in your project root folder (`react-list/`):

> npm init

This will prompt you to enter the following information:

* name
  - Module name. Make sure that it doesn't exist already in the [npm][npm] registry, or just scope it with your username: *@username/module_name*.
* version:
  - Module [SemVer][semver] version number, it will default to *1.0.0*. We'll take care of this later.
* description:
  - Simply describes what your module is/does. Could be something like *React Component for a list where you can add, edit, remove and search items.*, in this case.
* entry point:
  - This is the file that will be included when someone requires your module. It will default to *index.js*. We'll take care of this later.
* test command:
  - The command that will be used for running tests. We won't be developing tests, so leave blank.
* git repository:
  - Your remote [Git][git] repo. If you've already set a remote, it will default to that one.
* keywords:
  - This lets people discover your package. Could be something like *list, items, react, react-component*, in this case.
* author:
  - Your name.
* license:
  - Just to let people know what license you're using. Let's go with *MIT* (remember to include the license in your source code).

After you answer to all of the past prompts, the resulting `react-list/package.json` will be outputted, asking for your confirmation. If it is OK, just answer *yes* and we're done with this step.

### Add some structure
Too keep things simple, let's just create the folder `react-list/src/` where all our source code will be present (in an organized manner). Also create the file `react-list/src/index.js`, that will serve as our entry point for [webpack][webpack].

### Configuring webpack
First of all, we need to add [webpack][webpack] as a development dependency for our module. This is simply done running the following command:

> npm install webpack webpack-dev-server --save-dev

Notice that we've also installed *webpack-dev-server* since we'll be using it to test our module locally. We also need [Babel][babel] and its pertinent loaders to be able to transpile [ES6][es6] and [JSX][jsx] code:

> npm install babel-core babel-loader babel-preset-es2015 babel-preset-react --save-dev

Now we can create our [webpack][webpack] configuration file:

`react-list/webpack.config.js`
{% highlight javascript %}
module.exports = {
  context: __dirname + '/src',

  entry: './index',

  output: {
    library: 'ReactList',
    libraryTarget: 'commonjs2',
    filename: 'react-list.js',
    path: __dirname + '/dist'
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
  },

  externals: {
    react: {
      root: 'React',
      commonjs: 'react',
      commonjs2: 'react',
      amd: 'react'
    }
  }
};
{% endhighlight %}

Let's give a simple explanation for each one of this configuration options:

* context:
  - The directory where [webpack][webpack] will look for the entry file.
* entry:
  - The entry file that [webpack][webpack] will use for bundling.
* output:
	- library: This needs to be specified for bundles that'll be used as modules/libraries. It's just a name for the library.
	- libraryTarget: This is the kind of output for the library, here we specify *commonjs2* because we're offering it as a module that needs to be imported.
  - path:
    + The path where the bundle will be saved.
  - filename:
    + The bundle file name.
* module:
  - loaders:
    + Here we define loaders that will be applied for determinated files. In this case we're defining *babel-loader* to process any [JS][javascript] or [JSX][jsx] files.
* resolve:
  - extensions:
    + The file extensions that [webpack][webpack] will use for bundling.
* externals:
	- Specify dependencies that are not resolved by [webpack][webpack].

### Bundling
Add the following script to `react-list/package.json`, so we can use it to build our bundle:

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

We should have our first bundle built on `react-list/dist/react-list.js` (of course with no functionality yet).

### Dependencies

#### React

As said before, our module will consist in a [React][react] component, this means that it will be used as a *plugin* from an enclosing [React][react] project. Given this, we won't add [React][react] as a direct dependency for our module, instead it will be a development dependency, but we must also specify it as a peer dependency inside our `react-list/package.json`.

So, first of all let's add [React][react] as a development dependency for our module:

> npm install react react-dom --save-dev

After that, we add the following to our `react-list/package.json`:

`react-list/package.json`
{% highlight json %}
{
  .
  .
  .
  "peerDependencies": {
    "react": "^0.14.3"
  },
  .
  .
  .
}
{% endhighlight %}

I encourage you to read a bit more about peer dependencies [here][npm-peer-dependencies].

#### Alt.JS and Immutable.JS

To be able to follow the [Flux Architecture Pattern][flux] in our module, we'll be using [Alt.JS][alt]. We also want to keep [flux][flux] stores' states immutable, and there's a suitable [Javascript][javascript] library for that, which is [Immutable.JS][immutable]. Let's add both of them as dependencies for our module:

> npm install alt immutable --save

<!-- / PREPARING THINGS UP -->

<!-- START CODING -->

## Start coding

We've finally arrived to the fun part of the tutorial: coding.

### Baby steps

#### Alt.JS instance

First of all, we need to initialize an [Alt.JS][alt] instance, that will hold our [flux][flux] actions and stores:

`react-list/src/Alt.js`
{% highlight javascript %}
import Alt from 'alt';

const alt = new Alt();

export default alt;
{% endhighlight %}

#### List store

Now we can start building the [flux][flux] store that will hold our list state:

`react-list/src/stores/ListStore.js`
{% highlight javascript %}
import Immutable from 'immutable';

import Alt from '../Alt';

class ListStore {
  constructor() {
    this.items = Immutable.fromJS(
      [
        { name: 'Item 1' },
        { name: 'Item 2' },
        { name: 'Item 3' },
      ]
    );
  }
}

export default Alt.createStore(ListStore, 'ListStore');
{% endhighlight %}

This creates a simple store that holds a list of items initialized with some dummy values.

#### Main component

With our store ready, we can create the main [React][react] component that will listen to changes in the store state:

`react-list/src/components/ListContainer.jsx`
{% highlight javascript %}
import AltContainer from 'alt/AltContainer';
import React from 'react';

import ListStore from '../stores/ListStore';

import List from './List';

class ListContainer extends React.Component {
  render() {
    return (
      <AltContainer store={ListStore}>
        <List/>
      </AltContainer>
    );
  }
}

export default ListContainer;
{% endhighlight %}

It's worth noting that here we're using an `AltContainer` component, that's offered by [Alt.JS][alt] as a helper to make it easy to listen to a given store state and pass it down as props. Read more about `AltContainer` [here][alt-container].

#### List component

In the previously shown code, we imported a `List` component, that will take care of rendering our list:

`react-list/src/components/List.jsx`
{% highlight javascript %}
import React from 'react';

class List extends React.Component {
  render() {
    return (
      <ul>
        {this.props.items.map((item, i) => {
          return (
            <li key={i}>{item.get('name')}</li>
          );
        })}
      </ul>
    );
  }
}

export default List;
{% endhighlight %}

#### Webpack entry file

With the previous defined files, we have simple functionality developed, but we haven't updated our [webpack][webpack] entry file to export our main component `react-list/src/components/ListContainer.jsx`. Let's do that right now:

`react-list/src/index.js`
{% highlight javascript %}
import ListContainer from './components/ListContainer';

export default ListContainer;
{% endhighlight %}

#### Testing locally

Now we're ready to see our premature module in action. To do this, let's create an alternate [webpack][webpack] configuration file for development, that will be an exact copy of the original one, with exceptions in the `entry` and `output` options:

`react-list/webpack.config.dev.js`
{% highlight javascript %}
module.exports = {
  .
  .
  .
  entry: './index.dev',

  output: {
    path: __dirname + '/dist',
    publicPath: '/assets/',
    filename: 'bundle.js'
  },
  .
  .
  .
};
{% endhighlight %}

This defines a different entry file to be used, and it also specifies a path (`output.publicPath`) that must be used to reference the bundle from a public directory on a local server.

Then, we create our entry file for development:

`react-list/src/index.dev.js`
{% highlight javascript %}
import React from 'react';
import ReactDOM from 'react-dom';

import ListContainer from './components/ListContainer';

ReactDOM.render(<ListContainer/>, document.getElementById('main'));
{% endhighlight %}

This will try to render our main component into an HTML tag with the **main** id, so let's create a public directory, with an HTML file that we will serve through *webpack-dev-server*:

`react-list/public/index.html`
{% highlight html %}
<html>
  <body>
    <div id='main'>
    </div>
    <script type="text/javascript" src='assets/bundle.js'></script>
  </body>
</html>
{% endhighlight %}

Everything's set to lift a development server, and see our module in action. Add the following [npm][npm] script:

`react-list/package.json`
{% highlight json %}
{
  "scripts": {
    .
    .
    .
    "serve": "webpack-dev-server --config webpack.config.dev.js --content-base public/"
  },
  .
  .
  .
}
{% endhighlight %}

Now run:

> npm run serve

Open a browser, and go to [http:/localhost:8080/][localhost-8080]. Our simple list should display.

### Developing features

#### Setting initial list items via props

We want to be able to specify initial items for our list. This can be done through props in our main component. But first, we need a [flux][flux] action that will permit us to update our store items:

`react-list/src/actions/ListActions.js`
{% highlight javascript %}
import Alt from '../Alt';

class ListActions {
  updateItems(items) {
    this.dispatch(items);
  }
}

export default Alt.createActions(ListActions);
{% endhighlight %}

Actions in [Alt.JS][alt] are just functions that receive an arbitrary number of parameters, and must finally call the [Flux Dispatcher][flux-dispatcher] with just one parameter. Here we're declaring an action that will receive a list of items and dispatch them.

Now we need to bind the previously specified action to our store:

`react-list/src/stores/ListStore.js`
{% highlight javascript %}
.
.
.
import ListActions from '../actions/ListActions';

class ListStore {
  constructor() {
    this.items = Immutable.List();

    this.bindListeners({
      handleUpdateItems: ListActions.UPDATE_ITEMS
    });
  }

  handleUpdateItems(items) {
    this.items = Immutable.fromJS(items);
  }
}
.
.
.
{% endhighlight %}

An action handler inside a store, must be a funcion that should receive the parameter dispatched. Here we're receiving the list of items dispatched, and initializing our store items from that list.

We still need to update our main component, to be able to receive a prop with the initial items, and update the store accordingly:

`react-list/src/components/ListContainer.jsx`
{% highlight javascript %}
.
.
.
import ListActions from '../actions/ListActions';
.
.
.
class ListContainer extends React.Component {
  componentDidMount() {
    ListActions.updateItems(this.props.initialItems);
  }
  .
  .
  .
}

ListContainer.propTypes = {
  initialItems: React.PropTypes.arrayOf(
    React.PropTypes.shape({
      name: React.PropTypes.string
    })
  )
};
.
.
.
{% endhighlight %}

Notice how we're forcing our items to be objects with a `name` property. This is to keep things simple, but this can obviously be modified to represent a more complex item.

Read more about [React][react] props validation [here][react-props-validation].

#### Adding list items

One of the features we want for our list, is to be able to add new items. This will be done through a simple text input (as said before, to keep things simple and just have items with a `name` property). Following the same flow as before, let's create our action first, that will receive an item and dispatch it:

`react-list/src/actions/ListActions.js`
{% highlight javascript %}
.
.
.
class ListActions {
  .
  .
  .
  addItem(item) {
    this.dispatch(item);
  }
}
.
.
.
{% endhighlight %}

Now we can bind this action to our store, that will receive the dispatched item and add it to the list of items:

`react-list/src/store/ListStore.js`
{% highlight javascript %}
.
.
.
class ListStore {
  constructor() {
    .
    .
    .
    this.bindListeners({
      .
      .
      .
      handleAddItem: ListActions.ADD_ITEM
    });
  }
  .
  .
  .
  handleAddItem(item) {
    this.items = this.items.push(Immutable.fromJS(item));
  }
}
.
.
.
{% endhighlight %}

With this ready, we can update our [React][react] components accordingly.

Let's create a `NewItem` component that will hold a simple form to add new items, and handle its submission:

`react-list/src/components/NewItem.jsx`:
{% highlight javascript %}
import React from 'react';

import ListActions from '../actions/ListActions';

class NewItem extends React.Component {
  onAdd(event) {
    event.preventDefault();

    const item = {
      name: this.name.value
    };

    ListActions.addItem(item);

    this.props.onAdd(item);

    this.name.value = '';
  }

  render() {
    return (
      <form onSubmit={this.onAdd.bind(this)}>
        <input
          type="text"
          ref={(ref) => this.name = ref}/>
        <button>{this.props.addText}</button>
      </form>
    );
  }
}

NewItem.propTypes = {
  onAdd: React.PropTypes.func.isRequired,
  addText: React.PropTypes.string
};

NewItem.defaultProps = {
  addText: 'Add'
};

export default NewItem;
{% endhighlight %}

It's worth noting how we are exposing a prop to specify a callback that will be executed after a new item is added to the list, with the recently added item passed as a parameter. This has the purpose to enable custom behavior (persist data into a back-end for example).

Now let's update our `List` and main components accordingly:

`react-list/src/components/List.jsx`
{% highlight javascript %}
.
.
.
import NewItem from './NewItem';
class List extends React.Component {
  render() {
    return (
      <div>
        <ul>
          {this.props.items.map((item, i) => {
            return (
              <li key={i}>{item.get('name')}</li>
            );
          })}
        </ul>
        <NewItem
          onAdd={this.props.onAdd}
          addText={this.props.addText}/>
      </div>
    );
  }
}

List.propTypes = {
  onAdd: React.PropTypes.func.isRequired,
  addText: React.PropTypes.string
};
.
.
.
{% endhighlight %}

`react-list/src/components/ListContainer.jsx`
{% highlight javascript %}
.
.
.
class ListContainer extends React.Component {
  .
  .
  .
  render() {
    return (
      <AltContainer store={ListStore}>
        <List
          onAdd={this.props.onAdd}
          addText={this.props.addText}/>
      </AltContainer>
    );
  }
}

ListContainer.propTypes = {
  .
  .
  .
  onAdd: React.PropTypes.func.isRequired,
  addText: React.PropTypes.string
};
{% endhighlight %}

#### Removing list items

We've already developed the functionality to add items. Now we want to be able to remove items from the list. Simple: create the action, bind to store, and update components accordingly.

**Action**

The action will receive a list index and dispatch it.

`react-list/src/actions/ListActions.js`
{% highlight javascript %}
.
.
.
class ListActions {
  .
  .
  .
  removeItem(index) {
    this.dispatch(index);
  }
}
.
.
.
{% endhighlight %}

**Store**

The store will receive the dispatched index and remove the corresponding item.

`react-list/src/stores/ListStore.js`
{% highlight javascript %}
.
.
.
class ListStore {
  constructor() {
    .
    .
    .
    this.bindListeners({
      .
      .
      .
      handleRemoveItem: ListActions.REMOVE_ITEM
    });
  }
  .
  .
  .
  handleRemoveItem(index) {
    this.items = this.items.delete(index);
  }
}
.
.
.
{% endhighlight %}

**Components**

We need an `Item` component, that will represent each one of the items in the list, and it'll offer functionality to remove the item:

`react-list/src/components/Item.jsx`
{% highlight javascript %}
import React from 'react';

import ListActions from '../actions/ListActions';

class Item extends React.Component {
  onRemove(event) {
    event.preventDefault();

    ListActions.removeItem(this.props.index);

    this.props.onRemove(this.props.index, this.props.item.toJS());
  }

  render() {
    return (
      <li>
        <span>
          {this.props.item.get('name')}
        </span>
        <a
          href="#"
          onClick={this.onRemove.bind(this)}>
          {this.props.removeText}
        </a>
      </li>
    );
  }
}

Item.propTypes = {
  item: React.PropTypes.object.isRequired,
  index: React.PropTypes.number.isRequired,
  onRemove: React.PropTypes.func.isRequired,
  removeText: React.PropTypes.string
};

Item.defaultProps = {
  removeText: 'Remove'
};

export default Item;
{% endhighlight %}

The props defined include the item itself, its index, and an exposed callback that will be executed on item removal, passing the index and its corresponding item as parameters.

As before, we need to update our `List` and main components:

`react-list/src/components/List.jsx`
{% highlight javascript %}
.
.
.
import Item from './Item';
.
.
.
class List extends React.Component {
  render() {
    return (
      <div>
        <ul>
          {this.props.items.map((item, i) => {
            return (
              <Item
                key={i}
                item={item}
                index={i}
                onRemove={this.props.onRemove}
                removeText={this.props.removeText}/>
            );
          })}
        </ul>
        <NewItem
          onAdd={this.props.onAdd}
          addText={this.props.addText}/>
      </div>
    );
  }
}

List.propTypes = {
  .
  .
  .
  onRemove: React.PropTypes.func.isRequired,
  removeText: React.PropTypes.string
};
.
.
.
{% endhighlight %}

`react-list/src/components/ListContainer.jsx`
{% highlight javascript %}
.
.
.
class ListContainer extends React.Component {
  .
  .
  .
  render() {
    return (
      <AltContainer store={ListStore}>
        <List
          onAdd={this.props.onAdd}
          addText={this.props.addText}
          onRemove={this.props.onRemove}
          removeText={this.props.removeText}/>
      </AltContainer>
    );
  }
}

ListContainer.propTypes = {
  .
  .
  .
  onRemove: React.PropTypes.func.isRequired,
  removeText: React.PropTypes.string
};
.
.
.
{% endhighlight %}

#### Editing list items

You're getting used to this right?, then let's get right to the code.

**Action**

The action will receive the index and the new item, then dispatch this data as an object (remember that the [Flux Dispatcher][flux-dispatcher] can only handle a single parameter).

`react-list/src/actions/ListActions.js`
{% highlight javascript %}
.
.
.
class ListActions {
  .
  .
  .
  editItem(index, item) {
    this.dispatch({ index, item });
  }
}
.
.
.
{% endhighlight %}

**Store**

The store will receive the dispatched object and modify the corresponding item in the list.

`react-list/src/stores/ListStore.js`
{% highlight javascript %}
.
.
.
class ListStore {
  constructor() {
    .
    .
    .
    this.bindListeners({
      .
      .
      .
      handleEditItem: ListActions.EDIT_ITEM
    });
  }
  .
  .
  .
  handleEditItem({ index, item }) {
    this.items = this.items.set(index, Immutable.fromJS(item));
  }
}
.
.
.
{% endhighlight %}

**Components**

We don't need to develop a new component, but we must modify the existing ones accordingly to offer edition functionality.

`react-list/src/components/Item.jsx`
{% highlight javascript %}
.
.
.
class Item extends React.Component {
  constructor() {
    super();
    this.state = {
      editing: false
    };
  }
  .
  .
  .
  onEdit(event) {
    event.preventDefault();

    const item = {
      name: this.name.value
    };

    ListActions.editItem(this.props.index, item);

    this.props.onEdit(this.props.index, this.props.item.toJS(), item);

    this.setState({
      editing: false
    });
  }

  showInput(event) {
    event.preventDefault();

    this.setState({
      editing: true
    });
  }

  hideInput(event) {
    event.preventDefault();

    this.setState({
      editing: false
    });
  }

  render() {
    return (
      <li>
        {this.state.editing ? (
          <form onSubmit={this.onEdit.bind(this)}>
            <input
              type="text"
              defaultValue={this.props.item.get('name')}
              onBlur={this.hideInput.bind(this)}
              autoFocus
              ref={(ref) => this.name = ref}/>
          </form>
        ) : (
          <span>
            {this.props.item.get('name')}
          </span>
        )}
        {this.state.editing ? null : (
          <span>
            <a
              href="#"
              onClick={this.showInput.bind(this)}>
              {this.props.editText}
            </a>
            <a
              href="#"
              onClick={this.onRemove.bind(this)}>
              {this.props.removeText}
            </a>
          </span>
        )}
      </li>
    );
  }
}

Item.propTypes = {
  .
  .
  .
  onEdit: React.PropTypes.func.isRequired,
  editText: React.PropTypes.string
};

Item.defaultProps = {
  .
  .
  .
  editText: 'Edit'
};
.
.
.
{% endhighlight %}

Notice again, how we are exposing a prop for a callback to be executed each time an item is edited. The callback will receive the index edited, the original item, and the new item.

Another important thing to notice in the previous code, is how we're using the component state to control logic to hide/show the edition input. You can read more about [React][react] components state [here][react-state].

`react-list/src/components/List.jsx`
{% highlight javascript %}
.
.
.
class List extends React.Component {
  render() {
    return (
      <div>
        <ul>
          {this.props.items.map((item, i) => {
            return (
              <Item
                key={i}
                item={item}
                index={i}
                onRemove={this.props.onRemove}
                removeText={this.props.removeText}
                onEdit={this.props.onEdit}
                editText={this.props.editText}/>
            );
          })}
        </ul>
        <NewItem
          onAdd={this.props.onAdd}
          addText={this.props.addText}/>
      </div>
    );
  }
}

List.propTypes = {
  .
  .
  .
  onEdit: React.PropTypes.func.isRequired,
  editText: React.PropTypes.string
};
.
.
.
{% endhighlight %}

`react-list/src/components/ListContainer.jsx`
{% highlight javascript %}
.
.
.
class ListContainer extends React.Component {
  .
  .
  .
  render() {
    return (
      <AltContainer store={ListStore}>
        <List
          onAdd={this.props.onAdd}
          addText={this.props.addText}
          onRemove={this.props.onRemove}
          removeText={this.props.removeText}
          onEdit={this.props.onEdit}
          editText={this.props.editText}/>
      </AltContainer>
    );
  }
}

ListContainer.propTypes = {
  .
  .
  .
  onEdit: React.PropTypes.func.isRequired,
  editText: React.PropTypes.string
};
.
.
.
{% endhighlight %}

#### Searching items on the list

We've already developed 3 out of 4 proposed features for our list of items. The final feature we're going to develop, is filtering the list by a search query (text). The matching items will be those that start with the given query. Let's get coding.

**Action**

The action will receive the query text and dispatch it.

`react-list/src/actions/ListActions.js`
{% highlight javascript %}
.
.
.
class ListActions {
  .
  .
  .
  searchItem(query) {
    this.dispatch(query);
  }
}
.
.
.
{% endhighlight %}

**Store**

The store will receive the query dispatched, and filter the list correspondingly.

`react-list/src/stores/ListStore.js`
{% highlight javascript %}
.
.
.
class ListStore {
  constructor() {
    .
    .
    .
    this.auxItems = Immutable.List();

    this.bindListeners({
      .
      .
      .
      handleSearchItem: ListActions.SEARCH_ITEM
    });
  }

  handleUpdateItems(items) {
    .
    .
    .
    this.auxItems = Immutable.fromJS(items);
  }

  handleAddItem(item) {
    .
    .
    .
    this.auxItems = this.auxItems.push(Immutable.fromJS(item));
  }

  handleRemoveItem(index) {
    .
    .
    .
    this.auxItems = this.auxItems.delete(index);
  }

  handleEditItem({ index, item }) {
    .
    .
    .
    this.auxItems = this.auxItems.delete(index);
  }

  handleSearchItem(query) {
    this.items = this.auxItems.filter(
      (item) => item.get('name').toLowerCase().startsWith(query.toLowerCase())
    );
  }
}
.
.
.
{% endhighlight %}

It's important to keep an original list, so we can search items without losing information. That's why we are adding an `auxItems` property to our store state.

**Components**

We must create a `SearchBox` component, that will serve as a simple form to enter the search query, and filter the list as the query changes:

`react-list/src/components/SearchBox.jsx`
{% highlight javascript %}
import React from 'react';

import ListActions from '../actions/ListActions';

class SearchBox extends React.Component {
  clearQuery() {
    this.query.value = '';

    ListActions.searchItem(this.query.value);
  }

  onSearch(event) {
    event.preventDefault();

    ListActions.searchItem(this.query.value);

    this.props.onSearch(this.query.value);
  }

  render() {
    return (
      <form onSubmit={(event) => event.preventDefault()}>
        <label>{`${this.props.searchText}: `}</label>
        <input
          type="search"
          onChange={this.onSearch.bind(this)}
          ref={(ref) => this.query = ref}/>
      </form>
    );
  }
}

SearchBox.propTypes = {
  onSearch: React.PropTypes.func.isRequired,
  searchText: React.PropTypes.string
};

SearchBox.defaultProps = {
  searchText: 'Search'
};

export default SearchBox;
{% endhighlight %}

As done with the other features, we're exposing a callback to be executed each time the query changes, passing the query as a parameter.

`react-list/src/components/List.jsx`
{% highlight javascript %}
.
.
.
import SearchBox from './SearchBox';

class List extends React.Component {
  onAdd(item) {
    this.searchBox.clearQuery();

    this.props.onAdd(item);
  }

  render() {
    return (
      <div>
        <SearchBox
          onSearch={this.props.onSearch}
          searchText={this.props.searchText}
          ref={(ref) => this.searchBox = ref}/>
        <ul>
          {this.props.items.isEmpty() ? (
            <span>{this.props.emptyText}</span>
          ) : this.props.items.map((item, i) => {
            return (
              <Item
                key={i}
                item={item}
                index={i}
                onRemove={this.props.onRemove}
                removeText={this.props.removeText}
                onEdit={this.props.onEdit}
                editText={this.props.editText}/>
            );
          })}
        </ul>
        <NewItem
          onAdd={this.onAdd.bind(this)}
          addText={this.props.addText}/>
      </div>
    );
  }
}

List.propTypes = {
  .
  .
  .
  onSearch: React.PropTypes.func.isRequired,
  searchText: React.PropTypes.string,
  emptyText: React.PropTypes.string
};

List.defaultProps = {
  emptyText: 'No items.'
};
.
.
.
{% endhighlight %}

`react-list/src/components/ListContainer.jsx`
{% highlight javascript %}
.
.
.
class ListContainer extends React.Component {
  .
  .
  .
  render() {
    return (
      <AltContainer store={ListStore}>
        <List
          onAdd={this.props.onAdd}
          addText={this.props.addText}
          onRemove={this.props.onRemove}
          removeText={this.props.removeText}
          onEdit={this.props.onEdit}
          editText={this.props.editText}
          onSearch={this.props.onSearch}
          searchText={this.props.searchText}
          emptyText={this.props.emptyText}/>
      </AltContainer>
    );
  }
}

ListContainer.propTypes = {
  .
  .
  .
  onSearch: React.PropTypes.func.isRequired,
  searchText: React.PropTypes.string,
  emptyText: React.PropTypes.string
};
.
.
.
{% endhighlight %}

That's it, we're done developing the proposed features for our list. Everything's pretty simple of course, if you're already wondering how to improve this reusable [React][react] component (I do have a lot of ideas in mind), don't hesitate and go ahead and improve it, you can always contribute to my module [here][react-list] (the latest version may be way advanced from what we've developed here, the tag corresponding to this tutorial is [0.1.0][react-list-0.1.0]).

We still need one more thing to care about according to this tutorial scope: [npm][npm] publishing.

<!-- START CODING -->

<!-- PUBLISHING TO NPM -->

## Publishing to npm

### Creating a user

If you don't have a user on the [npm][npm] registry, you should add one by running the following command:

> npm adduser

This will prompt you for a *username*, *password* and *email*.

If you created one on the [npm][npm] site, use:

> npm login

This will prompt for your credentials.

### Updating package.json

We should update our `version` and `main` options:

`react-list/package.json`
{% highlight json %}
{
  .
  .
  .
  "version": "0.1.0",
  .
  .
  .
  "main": "dist/react-list.js",
  .
  .
  .
{% endhighlight %}

I've decided to change the version number from *1.0.0* to *0.1.0* because our module is pretty simple and naive yet, not really qualifying as a major *first* version.

The `main` value is referencing the bundle generated with [webpack][webpack], which is what we really want to expose.

### .npmignore

We don't want [npm][npm] to keep track of source/development files on our codebase, so let's create the following file:

`react-list/.npmignore`

    node_modules

    public

    src

    .eslintrc

    .gitignore

    webpack.config.dev.js
    webpack.config.js

### Build and publish

Before publishing to [npm][npm] we need to make sure that we build our bundle:

> npm run build

Now we can publish:

> npm publish --access=public

Include the `--access=public` flag only if you've scoped the name of your module. This is because scoped modules are private by default, and you'll need an [npm][npm] paid account for that.

<!-- / PUBLISHING TO NPM -->

<!-- END OF TUTORIAL -->

## End of tutorial

That's it, we've just developed a simple [npm][npm] module, using [React][react] and [flux][flux] among other tools.

I really hope you found this useful to improve your skills, or at least have a general idea about the techs/concepts we used/discussed throughout the tutorial.

### What next ?

About the module we built, there are lots of features/improvements that can be done, try the following:

* Permit arbitrary objects as list items, and expose a prop to specify a renderer that could be a function that receives an item and returns the content to render.
* Add [CSS][css] classes to each relevant element inside components, so it is easy to customize the general style for the list.
* Pagination.
* Be creative.

I also encourage you to further read about he following topics/techs/tools:

* [Redux][redux] (an alternative to [Alt.JS][alt]).
* [Polymer][polymer] (may be an alternative to build web components, instead of [React][react]).
* Testing with [Mocha][mocha] and [Chai][chai].
* Continous integration with [Travis][travis].
* Automate releases with [Semantic Release][semantic-release], and [Commitizen][commitizen].
* Adding code coverage with [Istanbul][istanbul].

<!-- / END OF TUTORIAL -->

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
[react-list-0.1.0]: https://github.com/luisincrespo/react-list/tree/0.1.0
[node]: https://nodejs.org/
[semver]: http://semver.org/
[javascript]: https://developer.mozilla.org/en-US/docs/Web/JavaScript
[jsx]: https://jsx.github.io/
[npm-peer-dependencies]: https://nodejs.org/en/blog/npm/peer-dependencies/
[alt-container]: http://alt.js.org/docs/components/altContainer/
[flux-dispatcher]: https://facebook.github.io/flux/docs/dispatcher.html
[react-props-validation]: https://facebook.github.io/react/docs/reusable-components.html#prop-validation
[localhost-8080]: http://localhost:8080/
[react-state]: https://facebook.github.io/react/docs/interactivity-and-dynamic-uis.html#components-are-just-state-machines
[css]: https://developer.mozilla.org/en-US/docs/Web/CSS
[redux]: http://rackt.org/redux/
[mocha]: https://mochajs.org/
[chai]: http://chaijs.com/
[semantic-release]: http://semantic-release.org/
[travis]: https://travis-ci.org/
[commitizen]: https://commitizen.github.io/cz-cli/
[istanbul]: https://gotwarlost.github.io/istanbul/
[polymer]: https://www.polymer-project.org/1.0/
