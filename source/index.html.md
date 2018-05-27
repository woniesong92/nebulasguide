---
title: Nebulas Guide

language_tabs: # must be one of https://git.io/vQNgJ
  - javascript

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>
  - <a href='https://github.com/lord/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true
---

# Introduction

Welcome to the Nebulas Guide! You can use this document to kickstart your project in Nebulas. This guide is authored by [Howon](https://howonsong.com/).

The only prerequisites for this guide are the following:

* JavaScript
* React

# Quickstart

In Quickstart, we will deploy a simple React app to Nebulas.

## Create React app

> 1.Create a react app:

```shell
$ npm install -g create-react-app

$ create-react-app my-nebulas
```

> 2.Verify that your react app is created successfully:

```shell
$ cd my-nebulas

$ yarn start
```

You should see this screen when your react app is running.

![React](react-start.png)

For our first app, we will build an app that lets users share doc pictures on Nebulas. For now, let's just grab a cute dog picture and show it in our app.

> 3.Modify your React component like this:

```javascript
import React, { Component } from 'react';

class App extends Component {
  render() {
    // Create mock dog objects. We will replace these with
    // real data later.
    const dogs = [
      {
        name: "Dog1",
        url: "/dog1.jpg"
      },
      {
        name: "Dog2",
        url: "/dog2.jpg"
      },
    ]

    return (
      <div className="App">
        {dogs.map((dog) => (
          <div>
            <label>{dog.name}</label>
            <img src={dog.url} />
          </div>  
        ))}
      </div>
    );
  }
}

export default App;
```

Here is an adorable picture:

![Dog1](dog1.jpg)

Once you've modified your React app, you should be seeing two dogs with labels and pictures.
We are *mocking* data here with a couple of dog objects. Let's replace them with actual data from Nebulas!

## Write Smart Contract

> 4.Create a new contract file:

```shell
$ mkdir contract
$ vim contract/firstcontract.js
```

```javascript
// contract/firstcontract.js
var Dog = function(jsonString) {
  if (jsonString) {
    var obj = JSON.parse(jsonString)

    this.name = obj.name
    this.url = obj.url
  } else {
    this.name = ""
    this.url = ""
  }
}

Dog.prototype = {
  toString: function () {
    return JSON.stringify(this)
  }
}

var FirstContract = function () {
  // Define storage properties (i.e. we will have two "storage variables" in our contract)
  LocalContractStorage.defineProperty(this, "numDogs")
  LocalContractStorage.defineMapProperty(this, "dogs")
}

FirstContract.prototype = {
  // init() will run when you first deploy
  init: function() {
    this.numDogs = 0
    var dogs = [
      {
        name: "Dog1",
        url: "dog1.jpg"
      },
      {
        name: "Dog2",
        url: "dog2.jpg"
      },
    ]

    for (var i=0; i<dogs.length; i++) {
      var dog = dogs[i]

      this.add(dog.name, dog.url)
    }
  },

  // add() will add a new dog to your contract's storage
  add: function(name, url) {
    var dog = new Dog(JSON.stringify({
      name: name,
      url: url,
    }))

    this.dogs.set(this.numDogs, dog)
    this.numDogs += 1
  },
}

module.exports = FirstContract
```

Let's write our first smart contract. A smart contract is basically just a javascript file that will have access to Nebulas' API when deployed.
Unlike Ethereum, you do not have to learn a new langauge (e.g. Solidity).

In this simple smart contract, we first define a simple `Dog` constructor that takes in a stringified json and creates a `Dog` object.

Then, we define the main contract with only two functions: `init` and `add`. When you define an `init` function, this will be executed when you first deploy your smtart contract to Nebulas. For example, if you want to *populate* your contract with a few cute dogs before users upload new dogs, this is the right place.

If you already know JavaSript, every line should be familar except for the lines that have `LocalContractStorage` in them. `LocalContractStorage` is a reserved function, which lets you have access to Nebulas' storage. Think of `LocalContractStorage.defineProperty(this, "numDogs")` as saying *"I am going to create a persistent variable `numDogs` in Nebulas"*. The difference between `defineProperty` and `defineMapProperty` is simply that the former is a primitive-type store while the latter is a key-value store.

## Deploy Smart Contract

> 5.Download Web Wallet

```shell
$ git clone git@github.com:nebulasio/web-wallet.git
$ cd web-wallet

// Open index.html in your browser and create a wallet on Testnet
```

We are ready to deploy our simple contract to Nebulas. To deploy a contract to Nebulas, you need to have some NAS, the currency used on Nebulas' network. To get NAS and deploy a contract to Nebulas, you need the [Web Wallet](https://github.com/nebulasio/web-wallet). Download it and open `index.html` in your browser to see the interface.

![Web Wallet](web-wallet1.jpg)

We are going to deploy our contract to Testnet. Think of Testnet as a staging server and Mainnet as production server. We will deploy to Testnet because it will let us test our contract without spending real NAS. You can ask for free NAS on Testnet by visiting the [Faucet](https://testnet.nebulas.io/claim). For wallet address, you can use the one you just created. You will have your free NAS within minutes.

Now that we have some NAS on Testnet, we can finally deploy our contract to Testnet. Go to the Contract tab, and simply copy our contract's content and paste it into the code section.

![Deploy](deploy.jpg)

After you deploy (i.e. submit) your contract, you will get the transaction hash and the deployed contract's address. Copy the address, as we will have to use it in our React app to interact with the contract.

![Contract Address](contract-address.jpg)

Congratulations! You have your first contract deployed to Nebulas.

## Interact with Smart Contract



# Kittens

## Get All Kittens

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let kittens = api.kittens.get();
```

> The above command returns JSON structured like this:

```json
[
  {
    "id": 1,
    "name": "Fluffums",
    "breed": "calico",
    "fluffiness": 6,
    "cuteness": 7
  },
  {
    "id": 2,
    "name": "Max",
    "breed": "unknown",
    "fluffiness": 5,
    "cuteness": 10
  }
]
```

This endpoint retrieves all kittens.

### HTTP Request

`GET http://example.com/api/kittens`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
include_cats | false | If set to true, the result will also include cats.
available | true | If set to false, the result will include kittens that have already been adopted.

<aside class="success">
Remember — a happy kitten is an authenticated kitten!
</aside>

## Get a Specific Kitten

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.get(2)
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.get(2)
```

```shell
curl "http://example.com/api/kittens/2"
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let max = api.kittens.get(2);
```

> The above command returns JSON structured like this:

```json
{
  "id": 2,
  "name": "Max",
  "breed": "unknown",
  "fluffiness": 5,
  "cuteness": 10
}
```

This endpoint retrieves a specific kitten.

<aside class="warning">Inside HTML code blocks like this one, you can't use Markdown, so use <code>&lt;code&gt;</code> blocks to denote code.</aside>

### HTTP Request

`GET http://example.com/kittens/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the kitten to retrieve

## Delete a Specific Kitten

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.delete(2)
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.delete(2)
```

```shell
curl "http://example.com/api/kittens/2"
  -X DELETE
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let max = api.kittens.delete(2);
```

> The above command returns JSON structured like this:

```json
{
  "id": 2,
  "deleted" : ":("
}
```

This endpoint deletes a specific kitten.

### HTTP Request

`DELETE http://example.com/kittens/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the kitten to delete

