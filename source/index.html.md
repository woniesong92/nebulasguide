---
title: Nebulas Guide

language_tabs: # must be one of https://git.io/vQNgJ
  - javascript

toc_footers:
  - <a href='http://soulchain.org'>Soulchain.org</a>

search: true
---

# Introduction

This is Nebulas Guide. Use this guide to get ahead on blockchain programming on Nebulas. This guide includes both a step-by-step tutorial to build a dapp from scratch and deploy it to production and documentation for the most frequently used properties on Nebulas. To follow this guide, you only need to know JavaScript and [React](https://reactjs.org/).

# Quickstart

In Quickstart, we will build a dapp from scratch and deploy it to production. In this section, we will build a simple React app that interacts with a smart contract on Nebulas.

## Create React app

> Create a react app:

```shell
$ npm install -g create-react-app

$ create-react-app my-nebulas
```

> Verify that your react app is created successfully:

```shell
$ cd my-nebulas

$ yarn start
```

You should see this screen when your react app is running.

![React](react-start.png)

For our first app, we will build an app that lets users share doc pictures on Nebulas. For now, let's just grab a cute dog picture and show it in our app.

> Modify your React component like this:

```javascript
import React, { Component } from 'react';

class App extends Component {
  render() {
    // Create mock dog objects. We will replace these with
    // real data later.
    const dogs = [
      {
        name: "Dog1",
        url: "https://woniesong92.github.io/nebulasguide/images/dog1.jpg"
      },
      {
        name: "Dog2",
        url: "https://woniesong92.github.io/nebulasguide/images/dog2.jpg"
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

> Create a new contract file:

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
        url: "https://woniesong92.github.io/nebulasguide/images/dog1.jpg"
      },
      {
        name: "Dog2",
        url: "https://woniesong92.github.io/nebulasguide/images/dog2.jpg"
      },
    ]

    for (var i=0; i<dogs.length; i++) {
      var dog = dogs[i]

      this.add(dog.name, dog.url)
    }
  },

  // addDog() will add a new dog to your contract's storage
  addDog: function(name, url) {
    var dog = new Dog(JSON.stringify({
      name: name,
      url: url,
    }))

    this.dogs.set(this.numDogs, dog)
    this.numDogs += 1
  },

  getDogs: function() {
    var dogs = []

    for (var i=0; i<this.numDogs; i++) {
      var dog = this.dogs.get(i)

      dogs.push(dog)
    }

    return dogs
  },
}

module.exports = FirstContract
```

Let's write our first smart contract. A smart contract is basically just a javascript file that will have access to Nebulas' API when deployed.
Unlike Ethereum, you do not have to learn a new langauge (e.g. Solidity).

In this simple smart contract, we first define a simple `Dog` constructor that takes in a stringified json and creates a `Dog` object.

Then, we define the main contract with only a few functions: `init`, `addDog`, and `getDogs`. When you define an `init` function, this will be executed when you first deploy your smtart contract to Nebulas. For example, if you want to *populate* your contract with a few cute dogs before users upload new dogs, this is the right place.

If you already know JavaSript, every line should be familar except for the lines that have `LocalContractStorage` in them. `LocalContractStorage` is a reserved function, which lets you have access to Nebulas' storage. Think of `LocalContractStorage.defineProperty(this, "numDogs")` as saying *"I am going to create a persistent variable `numDogs` in Nebulas"*. The difference between `defineProperty` and `defineMapProperty` is simply that the former is a primitive-type store while the latter is a key-value store.

## Deploy Smart Contract

> Download Web Wallet

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

## Read from Smart Contract

> Install Nebulas package in your React app

```shell
$ yarn add nebulas
```

> Use the dog objects from contract in your React app

```javascript
import React, { Component } from 'react'
import nebulas from 'nebulas'

class App extends Component {
  constructor(props) {
    super(props)

    this.state = {
      dogs: [],
    }
  }

  componentWillMount() {
    this.fetchDogs()
  }

  fetchDogs() {
    const CONTRACT_ADDRESS = "YOUR_CONTRACT_ADDRESS"
    const AUTHOR_ADDRESS = "YOUR_WALLET_ADDRESS"
    const NEBULAS_URL = "https://testnet.nebulas.io"
    const Account = nebulas.Account
    const neb = new nebulas.Neb()
    const request = new nebulas.HttpRequest(NEBULAS_URL)
    const value = "0"
    const nonce = "0"
    const gas_price = "1000000"
    const gas_limit = "2000000"
    const callFunction = "getDogs"
    const callArgs = ""  
    const contract = {
      "function": callFunction,
      "args": callArgs
    }

    neb.setRequest(request)
    neb.api.call(
      AUTHOR_ADDRESS,
      CONTRACT_ADDRESS,
      value, nonce,
      gas_price, gas_limit,
      contract
    ).then((resp) => {
      this.setState({
        dogs: JSON.parse(resp.result),
      })
    }).catch((err) => {
      console.log("error:" + err.message)
    })
  }

  render() {
    const {dogs} = this.state

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
```

Now that we have deployed our contract to Nebulas, we are ready to replace our mock data in our React app with actual the data from our contract in Nebulas.

To interact with the smart contract, you need to install the [nebulas package](https://www.npmjs.com/package/nebulas) in your React app. Then, we will call the `getDogs()` function in our smart contract via the API in `componentWillMount()` and update our state. For now, do not worry too much about the parameters we pass into `neb.api.call` as you will get used to them naturally as you interact more with your contract in future.

If you did everything correctly, you should see exactly the same screen as you saw when you go to your React app. However this time, the dog objects are coming straight from the smart contract instead of the static mock data. After all, interacting with Nebulas is just as simple as interacting with a 3rd-party API service.

## Write to Smart Contract

> Install [nebpay](https://www.npmjs.com/package/nebpay) package

```shell
$ yarn add nebpay
```

> Add a button to add a dog and the corresponding handler

```javascript
import React, { Component } from 'react'
import nebulas from 'nebulas'
import NebPay from 'nebpay'

class App extends Component {
  ...
  onAddDog() {
    const newDog = {
      name: "myDog",
      url: "https://woniesong92.github.io/nebulasguide/images/dog3.jpg"
    }
    const nebPay = new NebPay()
    const price = 0
    const callFunction = "addDog"
    const callArgs = JSON.stringify([newDog])
    const opts = {}
    
    nebPay.call(
      CONTRACT_ADDRESS,
      price,
      callFunction,
      callArgs,
      opts,
    )
  }

  render() {
    const {dogs} = this.state

    return (
      <div className="App">
        <div>
          {dogs.map((dog) => (
            <div>
              <label>{dog.name}</label>
              <img src={dog.url} />
            </div>
          ))}
        </div>
        <div>
          <button onClick={this.onAddDog}>Add Dog</button>
        </div>
      </div>
    );
  }
}
```

Remember how we wanted to let users add dogs to our contract? Since our contract already has `addDog()` function, we just need to add a way for our React app to call this function. In other words, we need to add a button and its corresponding handler. For now, let's skip adding an input field and just have users add *a dog* with the same name and image.

When you look at the function `onAddDog()` on the right side, you will notice that it is extremly similar to the function `fetchDogs()` we wrote above. The biggest difference is that instead of using the nebulas api directly, we are now using the `nebpay` package. This package lets users interact with your smart contract directly via either the [Chrome extension](https://github.com/ChengOrangeJu/WebExtensionWallet) or the mobile nebulas wallet. In other words, `nebpay` lets end users sign and send transactions to your smart contract.

Why is it needed? It's because altering the state in Nebulas blockchain requires a small amount of NAS, just like how Ethereum requires GAS for altering its state. While calling the `getDogs()` function does not alter the state, `addDog()` does. Consequently, users who want to add a dog to your smart contract must sign and send the transaction to your contract by paying a small fee. `nebpay` allows you to implement such interaction as easily as just using the `nebulas` package.

As a result of `nebPay.call()`, you get a transaction hash and you can optionally pass in a listener to know when the transaction has been confirmed. However, for now, we will just leave the code as simple as it is on the right side. Try clicking the button `Add Dog` and refresh the page in 30 seconds to 1 minute (i.e. the time it takes for the transaction to be confirmed on Testnet). Verify that you see an added dog in your app.

## Add Validation

> Add a new property `author` where we will store deployer's address

```javascript
var FirstContract = function () {
  LocalContractStorage.defineProperty(this, "numDogs")
  LocalContractStorage.defineMapProperty(this, "dogs")
  LocalContractStorage.defineProperty(this, "author")
}
...
```

> In `init()`, set author to be the deployer (i.e. you)
> and check if the sender is the author in `addDog()`

```javascript
FirstContract.prototype = {
  init: function() {
    ...
    this.author = Blockchain.transaction.from
  },
  addDog: function(name, url) {
    if (Blockchain.transaction.from !== this.author) {
      throw new Error("You are not the author")
    }

    var dog = new Dog(JSON.stringify({
      name: name,
      url: url,
    }))

    this.dogs.set(this.numDogs, dog)
    this.numDogs += 1
  },
}
```

But do you really want to let everyone add Dog to your app? Well, adding a dog to your app now is harmless, but what if you want to build a [dapp](http://naswarriors.soulchain.org) that should only let authors add new items?

To do this, we need to add validation. More specifically, in the function that adds a new dog, we need to check whether the *user* (i.e. the person who just sent a transaction to our contract) is the author. In other words, we want to validate that `user.address === author.address`. How do we get the author's address and the user's address?

The author is the person who deploys the app. In the `init()` function, we have access to the deployer's address in the property `Blockchain.transaction.from`. `Blockchain.transaction` is also another special object that is available in a Nebulas smart contract that lets you access certain properties of a transaction. The counterpart of `Blockchain.transaction.from` is `Blockchain.transaction.to`. For now, we just need to add a new property `author` after `dogs` and `numDogs` so that we can store the author's address.

In the function `addDog()` we can access the user's address the same way. Compare it with the author's address that we stored in the `init()` function and throw an error if they don't match. When you throw an error, the function will return with the error immediately. On the front end, if the contract throws any kind of an error as a result of user's transaction, you will get the error's message in response.

## Deploy to Production

This is the final section of the Quickstart! Now that we have built our first dapp, we have to deploy to production to be able to share it with other users. Deploying to production means hosting your frontend in a public address and deploying your smart contract to Mainnet.

### Host your frontend

If you want to use Nebulas as the primary backend, hosting your frontend is easy and free. Just build your project with yarn and upload your build folder to your favorite hosting service. This [guide](https://www.codementor.io/yurio/all-you-need-is-react-firebase-4v7g9p4kf) goes through how you can host your React app with Firebase. You can also just host it on [GitHub Pages](https://pages.github.com/). Whichever platform you decide to use, remember to change your Nebulas URL from `https://testnet.nebulas.io` to `https://mainnet.nebulas.io`.

If you want to use your own backend in addition to Nebulas, here are a few options: [Heroku](https://heroku.com), [Digital Ocean](https://www.digitalocean.com/), [AWS EC2](https://aws.amazon.com/ec2/).

### Deploy your contract to Mainnet

Deploying your contract to Mainnet is exactly the same as deploying it to Testnet. Simply choose Mainnet instead of Testnet for your deployment environment. The main difference is that to deploy to Mainnet, you need real NAS (i.e. you cannot use the NAS you got from the testnet faucet). To get real NAS, you can purchase it from the [exchanges](https://coinmarketcap.com/currencies/nebulas-token/#markets) that have NAS and transfer it to your Mainnet wallet.

![Web-wallet2](web-wallet2.jpg)

Congratulations! Now you have officially built and deployed a Dapp to the world.

## Next steps

Check out the following places to learn about Nebulas further.

- [API document](#api)
- [Nebulas Resources](https://nebulas.io/resources.html)
- [Nebulas Subreddit](https://www.reddit.com/r/nebulas)
- [Nebulas Medium](https://medium.com/@nebulasio)

# API

## Blockchain

> Blockchain API Usage

```javascript
var SampleContract = function () {
    LocalContractStorage.defineProperties(this, {
        name: null,
        count: null
    });
    LocalContractStorage.defineMapProperty(this, "allocation");
};

SampleContract.prototype = {
    init: function (name, count, allocation) {
      this.name = name;
      this.count = count;
      allocation.forEach(function (item) {
          this.allocation.put(item.name, item.count);
      }, this);
      console.log('init: Blockchain.block.coinbase = ' + Blockchain.block.coinbase);
      console.log('init: Blockchain.block.hash = ' + Blockchain.block.hash);
      console.log('init: Blockchain.block.height = ' + Blockchain.block.height);
      console.log('init: Blockchain.transaction.from = ' + Blockchain.transaction.from);
      console.log('init: Blockchain.transaction.to = ' + Blockchain.transaction.to);
      console.log('init: Blockchain.transaction.value = ' + Blockchain.transaction.value);
      console.log('init: Blockchain.transaction.nonce = ' + Blockchain.transaction.nonce);
      console.log('init: Blockchain.transaction.hash = ' + Blockchain.transaction.hash);
    },
    transfer: function (address, value) {
      var result = Blockchain.transfer(address, value);
      console.log("transfer result:", result);
      Event.Trigger("transfer", {
        Transfer: {
          from: Blockchain.transaction.to,
          to: address,
          value: value
        }
      });
    },
    verifyAddress: function (address) {
      var result = Blockchain.verifyAddress(address);
      console.log("verifyAddress result:", result);
    }
};

module.exports = SampleContract;
```

*Blockchain top-level API*

* `Blockchain.block`
  * current block 
* `Blockchain.transaction`
  * current transaction, transaction's value/gasPrice/gasLimit as BigNumber
* `Blockchain.transfer(address, value)`
  * transfer NAS from contract to address
* `Blockchain.verifyAddress(address)`
  * verify address

*Blockchain.properties*

* `block`: current block for contract execution
  * `timestamp`: block timestamp
  * `seed`: random seed
  * `height`: block height
* `transaction`: current transaction for contract execution
  * `hash`: transaction hash
  * `from`: transaction from address
  * `to`: transaction to address
  * `value`: transaction value, a BigNumber object for contract use
  * `nonce`: transaction nonce
  * `timestamp`: transaction timestamp
  * `gasPrice`: transaction gasPrice, a BigNumber object for contract use
  * `gasLimit`: transaction gasLimit, a BigNumber object for contract use
* `transfer(address, value)`: transfer NAS from contract to address
  * params:
    * `address`: nebulas address to receive NAS
    * `value`: transfer value, a BigNumber object. The unit is wei, only integer value is valid.
  * return (boolean value):
    * `true`: transfer success
    * `false`: transfer failed   
* `verifyAddress(address)`: verify address
  * params:
    * `address`: address need to check
  * return (number value):
    * `87`: user wallet address
    * `88`: smart-contract address
    * `0`: address is invalid 

## LocalContractStorage

> Usage

```javascript
var FirstContract = function () {
  // Use `defineProperty` to store primitive-type values
  LocalContractStorage.defineProperty(this, "author")

  // Use `defineMapProperty` to store key-value-type values
  LocalContractStorage.defineMapProperty(this, "dogFamily")
}

FirstContract.prototype = {
  ...
  init: function() {
    this.author = Blockchain.transaction.from
    this.dogFamily.set("dad", "Marlie")
    this.dogFamily.set("mom", "Lucy")
  },
  getDad: function() {
    return this.dogFamily.get("dad")
  },
  setFamilyMember: function(memberType, memberName) {
    this.dogFamily.set(memberType, memberName)
  },
  ...
}
```

> Module implementation

```typescript
interface Descriptor {
  // serialize value to string;
  stringify?(value: any): string;

  // deserialize value from string;
  parse?(value: string): any;
}

interface DescriptorMap {
  [fieldName: string]: Descriptor;
}

interface ContractStorage {
  // get and return value by key from Native Storage.
  rawGet(key: string): string;
  // set key and value pair to Native Storage,
  // return 0 for success, otherwise failure.
  rawSet(key: string, value: string): number;

  // define a object property named `fieldname` to `obj` with descriptor.
  // default descriptor is JSON.parse/JSON.stringify descriptor.
  // return this.
  defineProperty(obj: any, fieldName: string, descriptor?: Descriptor): any;

  // define object properties to `obj` from `props`.
  // default descriptor is JSON.parse/JSON.stringify descriptor.
  // return this.
  defineProperties(obj: any, props: DescriptorMap): any;

  // define a StorageMap property named `fieldname` to `obj` with descriptor.
  // default descriptor is JSON.parse/JSON.stringify descriptor.
  // return this.
  defineMapProperty(obj: any, fieldName: string, descriptor?: Descriptor): any;

  // define StorageMap properties to `obj` from `props`.
  // default descriptor is JSON.parse/JSON.stringify descriptor.
  // return this.
  defineMapProperties(obj: any, props: DescriptorMap): any;

  // delete key from Native Storage.
  // return 0 for success, otherwise failure.
  del(key: string): number;

  // get value by key from Native Storage,
  // deserialize value by calling `descriptor.parse` and return.
  get(key: string): any;

  // set key and value pair to Native Storage,
  // the value will be serialized to string by calling `descriptor.stringify`.
  // return 0 for success, otherwise failure.
  set(key: string, value: any): number;
}

interface StorageMap {
  // delete key from Native Storage, return 0 for success, otherwise failure.
  del(key: string): number;

  // get value by key from Native Storage,
  // deserialize value by calling `descriptor.parse` and return.
  get(key: string): any;

  // set key and value pair to Native Storage,
  // the value will be serialized to string by calling `descriptor.stringify`.
  // return 0 for success, otherwise failure.
  set(key: string, value: any): number;
}
```

The ```LocalContractStorage``` module provides a state trie based storage capability. It accepts string only key value pairs. And all data are stored to a private state trie associated with current contract address, only the contract can access them.

## BigNumber

```javascript
var zero = new BigNumber(0);
var one = new BigNumber(1);
var two = one.plus(1);

console.log("is greater than", two.gt(one))
console.log("is greater than or equal to", two.gte(one))
console.log("is less than", one.lt(two))
console.log("is less than or equal to", one.lte(one))
console.log("is equal to", one.eq(one))
```

The `BigNumber` module use the [bignumber.js](https://github.com/MikeMcl/bignumber.js)(v4.1.0), a JavaScript library for arbitrary-precision decimal and non-decimal arithmetic. The contract can use `BigNumber` directly to handle the value of the transaction and other values transfer.

## Event

```javascript
  // Event.Trigger("eventName", customObject)

  Event.Trigger("transfer", {
    Transfer: {
      from: Blockchain.transaction.to,
      to: address,
      value: value
    }
  });
```

The `Event` module records execution events in contract. The recorded events are stored in the event trie on the chain, which can be fetched by [`GetEventsByHash`](https://github.com/nebulasio/wiki/blob/master/rpc.md#geteventsbyhash)API with the execution transaction hash. All contract event topics have a `chain.contract.` prefix before the topic they set in contract.

## Math.random

> Math.random()

```javascript
var BankVaultContract = function () {};

BankVaultContract.prototype = {
  init: function () {},
  game: function(subscript){
    var arr =[1,2,3,4,5,6,7,8,9,10,11,12,13];

    for(var i = 0; i < arr.length; i++) {
      var rand = parseInt(Math.random()*arr.length);
      var t = arr[rand];
      arr[rand] = arr[i];
      arr[i] = t;
    }

    return arr[parseInt(subscript)];
  },
};
module.exports = BankVaultContract;
```

> Math.random.seed(myseed)

```javascript
var BankVaultContract = function () {};

BankVaultContract.prototype = {
  init: function () {},
  game: function(subscript, myseed){
    var arr =[1,2,3,4,5,6,7,8,9,10,11,12,13];
  
    for(var i = 0; i < arr.length; i++){
      if (i == 8) {
        // reset random seed with `myseed`
        Math.random.seed(myseed);
      }

      var rand = parseInt(Math.random() * arr.length);
      var t = arr[rand];
      arr[rand] = arr[i];
      arr[i] = t;
    }

    return arr[parseInt(subscript)];
  },
};

module.exports = BankVaultContract;
```

* `Math.random()` returns a floating-point, pseudo-random number in the range from 0 inclusive up to but not including 1. The typical usage is:
* `Math.random.seed(myseed)` if needed, you can use this method to reset random seed. The argument `myseed` must be a **string**.

## Date 
```javascript
var BankVaultContract = function () {};

BankVaultContract.prototype = {
  init: function () {},
  test: function(){
    var d = new Date();
    return d.toString();
  }
};

module.exports = BankVaultContract;
```

NOTE:

* Unsupported methods：`toDateString()`, `toTimeString()`, `getTimezoneOffset()`, `toLocaleXXX()`.
* `new Date()`/`Date.now()` returns the timestamp of current block in milliseconds.
* `getXXX` returns the result of `getUTCXXX`.

## Accept

```javascript
var DepositeContent = function(JSONString) {
  if (JSONString) {
    var o = JSON.parse(JSONString);
    this.balance = new BigNumber(o.balance);
    this.address = o.address;
  } else {
    this.balance = new BigNumber(0);
    this.address = "";
  }
};

DepositeContent.prototype = {
  toString: function () {
    return JSON.stringify(this);
  }
};

var BankVaultContract = function () {
  LocalContractStorage.defineMapProperty(this, "bankVault", {
    parse: function (JSONString) {
      return new DepositeContent(JSONString);
    },
    stringify: function (o) {
      return o.toString();
    }
  });
};

BankVaultContract.prototype = {
  init: function () {},
  save: function () {
    var from = Blockchain.transaction.from;
    var value = Blockchain.transaction.value;
    value = new BigNumber(value);
    var orig_deposit = this.bankVault.get(from);

    if (orig_deposit) {
        value = value.plus(orig_deposit.balance);
    }

    var deposit = new DepositeContent();
    deposit.balance = new BigNumber(value);
    deposit.address = from;
    this.bankVault.put(from, deposit);
  },

  accept: function() {
    this.save();
    Event.Trigger("transfer", {
      Transfer: {
        from: Blockchain.transaction.from,
        to: Blockchain.transaction.to,
        value: Blockchain.transaction.value,
      }
    });
  }
};

module.exports = BankVaultContract;
```

Send a binary transfer to a contract account. As the `to` is a smart contact address, which has declared a function `accept()` and it excutes correctly, the transfer will succeed. If the Tx is a non-binary Tx,it will be treated as a normal function.

## External Contract Call

<aside class="warning">
  External Contract Call is not available yet in production.
</aside>

We provide a simple method in a smart contract to call another contract, the following example shows that how proxyKvStore provide service by using the contract kvStore.

```javascript
// proxyKvStore.js:

var proxyKvStore = function() {};

var contractInterface = {
  save: function(key value) {},
  get: function(key) {},
}

proxyKvStore.prototype = {
  init: function() {},
  save: function(address, key, value) {
    var kvStore  = new Blockchain.Contract(address, contractInterface);
    kvStore.value(20000000000000000).save(key, value);
  },
  saveByCall: function(address, key, value) {
    var kvStore  = new Blockchain.Contract(address, contractInterface);
    var args = new Array();
    args[0] = key;
    args[1] = value;
    kvStore.value(20000000000000000).call('save', JSON.stringify(args));
  },
  get: function(address, key) {
    var kvStore = new Blockchain.Contract(address, contractInterface);
    return kvStore.get(key)
  },
}

module.exports = proxyKvStore;
```

```javascript
// kvStore.js:

var item = function(JSONString) {
  if (JSONString) {
    var obj = JSON.parse(JSONString);
    this.key = obj.key;
    this.value = obj.value;
    this.author = Blockchain.transaction.from;
  } else {
    this.key = "";
    this.author = "";
    this.value = "";
  }
};

item.prototype = {
  toString: function () {
    return JSON.stringify(this);
  }
};

var kvStore = function () {
  LocalContractStorage.defineMapProperty(this, "repo", {
    parse: function (text) {
      return new item(text);
    },
    stringify: function (o) {
      return o.toString();
    }
  });
};

kvStore.prototype = {
  init: function () {
    // todo
  },
  save: function (key, value) {
    key = key.trim();
    value = value.trim();
    if (key === "" || value === ""){
      throw new Error("empty key / value");
    }
    if (value.length > 128 || key.length > 128){
      throw new Error("key / value exceed limit length")
    }

    var from = Blockchain.transaction.from;
    var item = this.repo.get(key);

    if (item){
      throw new Error("value has been taken");
    }

    item = new item();
    item.author = from;
    item.key = key;
    item.value = value;
    this.repo.put(key, item);
  },

    get: function (key) {
        key = key.trim();
        if ( key === "" ) {
            throw new Error("empty key")
        }
        return this.repo.get(key);
    }, 

    throwErr: function() {
        throw("err for test");
    }
};

module.exports = kvStore;
```

> In the example, to use KvStore, we first should define an interface like this:

```javascript
var contractInterface = {
    save: function(key, value) {
    },
    get: function(key) {
    },
}
```

> Then, create a contract object with the interface and the address of the contract you want to call:

```javascript
var kvStore  = new Blockchain.Contract(address, contractInterface);
```

> After that, we can call function in KvStore.js thought this object:
```javascript
kvStore.value(2000000000000000000).save(key, value);
// or kvStore.save(key, value);
```

> as well as this way, keep the style of calling a function by sendRawTransaction RPC:

```javascript
kvStore.value(2000000000000000000).call('save', JSON.stringify(args));
```

> the 'value' function decides how much nas will be transfered to callee contract. It is not necessary， and the default value is 0.
> It should be noted that in the execution environment of the callee contract, Blockchain.from returns the address of the caller contract, and Blockchain.value is determined by the parameter of the 'value' function executed by the caller contract.
