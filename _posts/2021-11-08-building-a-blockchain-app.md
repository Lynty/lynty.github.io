---
layout: post
author: lynn
tags: dapp blockchain poc
---

* T0C
{:toc}

---

## Part 1: Project Setup, Initial Deployment, and Test
Following <a href="https://www.dappuniversity.com/articles/how-to-build-a-blockchain-app" target="_blank">this</a> tutorial by dapp university to build a Craigslist-like app on a personal ethereum blockchain!

> macOS

- install dependencies
  - node 
    - `v17.0.1`
    - server-side javascript platform
  - ganache 
    - `Ganache CLI v6.12.2 (ganache-core: 2.13.2)`
    - UI `v2.5.4`
    - personal blockchain
  - truffle framework 
    - `v5.0.5`
    - dev tools for creating smart contracts

```bash
brew install node

npm install -g ganache-cli
wget https://github.com/trufflesuite/ganache-ui/releases/download/v2.5.4/Ganache-2.5.4-mac.dmg
open Ganache-2.5.4-mac.dmg

npm install -g truffle@5.0.5

git clone https://github.com/dappuniversity/starter_kit marketplace
cd marketplace
npm install
```
- create a smart contract file in `src/contracts` called `Marketplace.sol`
  - declare version of Solidity to use
  - create `name` variable for the state that will be stored on the blockchain and set its value in `constructor` function

```solidity
pragma solidity ^0.5.0;

contract Marketplace {
    string public name;

    constructor() public {
        name = "Dapp University Marketplace";
    }
}
```

```bash
truffle compile
```

<br>
- open ganache and create a new workspace

<figure>
<center><img src="/assets/images/ganache_workspace.png" style="width:100%">
<figcaption>name the workspace "marketplace"</figcaption></center>
</figure>
<br>

<figure>
<center><img src="/assets/images/ganache.png" style="width:80%"></center><br>
<figcaption>ensure rpc server points to http://127.0.0.1:7545</figcaption>
</figure>
<br>

- migrate smart contract onto ganache personal blockchain
  - create migration file in `migrations/` called `2_deploy_contracts.js`

```solidity
const Marketplace = artifacts.require("Marketplace");

module.exports = function(deployer) {
  deployer.deploy(Marketplace);
};
```

```bash
truffle migrate
```

- `truffle console` is a javascript runtime env that allows us to interact with blockchain and smart contracts
  - web3: get accounts, block number
    - ex: `web3.eth.getAccounts()`
  - smart contract info: get address, name
    - ex: `marketplace = await Marketplace.deployed()`

- create in `test/` a file called `Marketplace.test.js`
  - check contract has been deployed, has an address, and has a name
  - Truffle framework comes with mocha and chai
    - mocha: test framework
    - chai: assertion library

```javascript
const Marketplace = artifacts.require('./Marketplace.sol')

contract('Marketplace', (accounts) => {
  let marketplace

  before(async () => {
    marketplace = await Marketplace.deployed()
  })

  describe('deployment', async () => {
    it('deploys successfully', async () => {
      const address = await marketplace.address
      assert.notEqual(address, 0x0)
      assert.notEqual(address, '')
      assert.notEqual(address, null)
      assert.notEqual(address, undefined)
    })

    it('has a name', async () => {
      const name = await marketplace.name()
      assert.equal(name, 'Dapp University Marketplace')
    })

  })
})
```

```bash
truffle test
```

---

&nbsp;

## Part 2: Sell & Buy Products

- Sell: allow user to list an item for sale on the marketplace
  - model a structure for our products with `struct`
  - need a place to store product in blockchain so we will use `mapping()`
  - keep track of how many products exist with `productCount` counter cache variable
  - function to create a new product
  - add the struct to the mapping and store it on the blockchain
  - trigger an event that lets someone know a product was created
- Buy: allow user to purchase a product
  - fetch product from mapping and create a copy in memory
  - store current owner to a variable and will transfer ownership
  - add requirements (id, enough ether for transaction, buyer != seller, and product not already purchased)
  - facilitate transaction
  - trigger an event to declare product has been purchased
- `src/contracts/Marketplace.sol` smart contract should look like the following:

```solidity
pragma solidity ^0.5.0;

contract Marketplace {
    string public name;
    uint public productCount = 0;
    mapping(uint => Product) public products;

    struct Product {
        uint id;
        string name;
        uint price;
        address payable owner;
        bool purchased;
    }

    event ProductCreated(
        uint id,
        string name,
        uint price,
        address payable owner,
        bool purchased
    );

    event ProductPurchased(
        uint id,
        string name,
        uint price,
        address payable owner,
        bool purchased
    );

    constructor() public {
        name = "Dapp University Marketplace";
    }

    function createProduct(string memory _name, uint _price) public {
        // Require a valid name
        require(bytes(_name).length > 0);
        // Require a valid price
        require(_price > 0);
        // Increment product count
        productCount ++;
        // Create the product
        products[productCount] = Product(productCount, _name, _price, msg.sender, false);
        // Trigger an event
        emit ProductCreated(productCount, _name, _price, msg.sender, false);
    }

    function purchaseProduct(uint _id) public payable {
        // Fetch the product
        Product memory _product = products[_id];
        // Fetch the owner
        address payable _seller = _product.owner;
        // Make sure the product has a valid id
        require(_product.id > 0 && _product.id <= productCount);
        // Require that there is enough Ether in the transaction
        require(msg.value >= _product.price);
        // Require that the product has not been purchased already
        require(!_product.purchased);
        // Require that the buyer is not the seller
        require(_seller != msg.sender);
        // Transfer ownership to the buyer
        _product.owner = msg.sender;
        // Mark as purchased
        _product.purchased = true;
        // Update the product
        products[_id] = _product;
        // Pay the seller by sending them Ether
        address(_seller).transfer(msg.value);
        // Trigger an event
        emit ProductPurchased(productCount, _product.name, _product.price, msg.sender, true);
    }
}
```

- create tests to make sure the `createProduct` function works properly
  - add extra tools to test suite (`chai-as-promised`)
  - add 3 new accounts to test scenario (`deployer`, `seller`, `buyer`)
  - test example for creating products
    - ensure counter goes up
    - ensure correct values in smart contract event logs
    - check failure cases (ex: if price value in log is negative)
- `test/Marketplace.test.js` smart contract should look like the following:

```javascript
const Marketplace = artifacts.require('./Marketplace.sol')

require('chai')
  .use(require('chai-as-promised'))
  .should()

contract('Marketplace', ([deployer, seller, buyer]) => {
  let marketplace

  before(async () => {
    marketplace = await Marketplace.deployed()
  })

  describe('deployment', async () => {
    it('deploys successfully onto blockchain', async () => {
      const address = await marketplace.address
      assert.notEqual(address, 0x0)
      assert.notEqual(address, '')
      assert.notEqual(address, null)
      assert.notEqual(address, undefined)
    })

    it('has a name', async () => {
      const name = await marketplace.name()
      assert.equal(name, 'Dapp University Marketplace')
    })
  })

  describe('products', async () => {
    let result, productCount

    before(async () => {
      result = await marketplace.createProduct('iPhone X', web3.utils.toWei('1', 'Ether'), { from: seller })
      productCount = await marketplace.productCount()
    })

    it('creates products', async () => {
      // SUCCESS
      assert.equal(productCount, 1)
      const event = result.logs[0].args
      assert.equal(event.id.toNumber(), productCount.toNumber(), 'id is correct')
      assert.equal(event.name, 'iPhone X', 'name is correct')
      assert.equal(event.price, '1000000000000000000', 'price is correct')
      assert.equal(event.owner, seller, 'owner is correct')
      assert.equal(event.purchased, false, 'purchased is correct')

      // FAILURE: Product must have a name
      await await marketplace.createProduct('', web3.utils.toWei('1', 'Ether'), { from: seller }).should.be.rejected;
      // FAILURE: Product must have a price
      await await marketplace.createProduct('iPhone X', 0, { from: seller }).should.be.rejected;
    })
    
    it('lists products', async () => {
      const product = await marketplace.products(productCount)
      assert.equal(product.id.toNumber(), productCount.toNumber(), 'id is correct')
      assert.equal(product.name, 'iPhone X', 'name is correct')
      assert.equal(product.price, '1000000000000000000', 'price is correct')
      assert.equal(product.owner, seller, 'owner is correct')
      assert.equal(product.purchased, false, 'purchased is correct')
    })

    it('sells products', async () => {
      // Track the seller balance before purchase
      let oldSellerBalance
      oldSellerBalance = await web3.eth.getBalance(seller)
      oldSellerBalance = new web3.utils.BN(oldSellerBalance)
    
      // SUCCESS: Buyer makes purchase
      result = await marketplace.purchaseProduct(productCount, { from: buyer, value: web3.utils.toWei('1', 'Ether')})
    
      // Check logs
      const event = result.logs[0].args
      assert.equal(event.id.toNumber(), productCount.toNumber(), 'id is correct')
      assert.equal(event.name, 'iPhone X', 'name is correct')
      assert.equal(event.price, '1000000000000000000', 'price is correct')
      assert.equal(event.owner, buyer, 'owner is correct')
      assert.equal(event.purchased, true, 'purchased is correct')
    
      // Check that seller received funds
      let newSellerBalance
      newSellerBalance = await web3.eth.getBalance(seller)
      newSellerBalance = new web3.utils.BN(newSellerBalance)
    
      let price
      price = web3.utils.toWei('1', 'Ether')
      price = new web3.utils.BN(price)
    
      const exepectedBalance = oldSellerBalance.add(price)
    
      assert.equal(newSellerBalance.toString(), exepectedBalance.toString())
    
      // FAILURE: Tries to buy a product that does not exist, i.e., product must have valid id
      await marketplace.purchaseProduct(99, { from: buyer, value: web3.utils.toWei('1', 'Ether')}).should.be.rejected;      // FAILURE: Buyer tries to buy without enough ether
      // FAILURE: Buyer tries to buy without enough ether
      await marketplace.purchaseProduct(productCount, { from: buyer, value: web3.utils.toWei('0.5', 'Ether') }).should.be.rejected;
      // FAILURE: Deployer tries to buy the product, i.e., product can't be purchased twice
      await marketplace.purchaseProduct(productCount, { from: deployer, value: web3.utils.toWei('1', 'Ether') }).should.be.rejected;
      // FAILURE: Buyer tries to buy again, i.e., buyer can't be the seller
      await marketplace.purchaseProduct(productCount, { from: buyer, value: web3.utils.toWei('1', 'Ether') }).should.be.rejected;
    })
  })
})
```

```bash
truffle compile
truffle test
```

---

&nbsp;

## Part 3: Frontend Marketplace Website Setup

- start app and run starter kit in browser which comes with the following:
  - react.js for building out interface
  - bootstrap for UI elements without css
  - web3.js for connecting app to blockchain

```bash
npm run start
```

- connect web browser to blockchain with metamask extension
  - connect metamask to ganache personal blockchain instance
  - import some accounts from ganache to metamask
  - in ganache, click the key for the 2nd account and paste the private key in metamask during account importing
  - in metamask, rename the account to "Seller"
  - in ganache, click the key for the 3rd account and paste the private key in metamask during account importing
  - in metamask, rename the account to "Buyer"
  - change network from Ethereum Mainnet to Localhost (may need to edit the network to have the correct port number)

- Build out UI for marketplace dApp in `src/components/App.js`
- `src/components/App.js` should look like the following:

```javascript
import React, { Component } from 'react';
import Web3 from 'web3'
import logo from '../logo.png';
import './App.css';
import Marketplace from '../abis/Marketplace.json'
import Navbar from './Navbar'
import Main from './Main'

class App extends Component {

  async componentWillMount() {
    await this.loadWeb3()
    await this.loadBlockchainData()
  }

  async loadWeb3() {
    if (window.ethereum) {
      window.web3 = new Web3(window.ethereum)
      await window.ethereum.enable()
    }
    else if (window.web3) {
      window.web3 = new Web3(window.web3.currentProvider)
    }
    else {
      window.alert('Non-Ethereum browser detected. You should consider trying MetaMask!')
    }
  }

  async loadBlockchainData() {
    const web3 = window.web3
    // Load account
    const accounts = await web3.eth.getAccounts()
    this.setState({ account: accounts[0] })
    const networkId = await web3.eth.net.getId()
    const networkData = Marketplace.networks[networkId]
    if(networkData) {
      const marketplace = web3.eth.Contract(Marketplace.abi, networkData.address)
      this.setState({ marketplace })
      const productCount = await marketplace.methods.productCount().call()
      this.setState({ productCount })
      // Load products
      for (var i = 1; i <= productCount; i++) {
        const product = await marketplace.methods.products(i).call()
        this.setState({
          products: [...this.state.products, product]
        })
      }
      this.setState({ loading: false})
    } else {
      window.alert('Marketplace contract not deployed to detected network.')
    }
  }

  constructor(props) {
    super(props)
    this.state = {
      account: '',
      productCount: 0,
      products: [],
      loading: true
    }

    this.createProduct = this.createProduct.bind(this)
    this.purchaseProduct = this.purchaseProduct.bind(this)
  }

  createProduct(name, price) {
    this.setState({ loading: true })
    this.state.marketplace.methods.createProduct(name, price).send({ from: this.state.account })
    .once('receipt', (receipt) => {
      this.setState({ loading: false })
    })
  }

  purchaseProduct(id, price) {
    this.setState({ loading: true })
    this.state.marketplace.methods.purchaseProduct(id).send({ from: this.state.account, value: price })
    .once('receipt', (receipt) => {
      this.setState({ loading: false })
    })
  }

  render() {
    return (
      <div>
        <Navbar account={this.state.account} />
        <div className="container-fluid mt-5">
          <div className="row">
            <main role="main" className="col-lg-12 d-flex">
              { this.state.loading
                ? <div id="loader" className="text-center"><p className="text-center">Loading...</p></div>
                : <Main
                  products={this.state.products}
                  createProduct={this.createProduct}
                  purchaseProduct={this.purchaseProduct} />
              }
            </main>
          </div>
        </div>
      </div>
    );
  }
}

export default App;
```

- List account in Navbar by creating `src/components/Navbar.js`
- `src/components/Navbar.js` should look like the following:

```javascript
import React, { Component } from 'react';

class Navbar extends Component {

  render() {
    return (
      <nav className="navbar navbar-dark fixed-top bg-dark flex-md-nowrap p-0 shadow">
        <a
          className="navbar-brand col-sm-3 col-md-2 mr-0"
          href="http://www.dappuniversity.com/bootcamp"
          target="_blank"
          rel="noopener noreferrer"
        >
          Dapp University's Blockchain Marketplace
        </a>
        <ul className="navbar-nav px-3">
          <li className="nav-item text-nowrap d-none d-sm-none d-sm-block">
            <small className="text-white"><span id="account">{this.props.account}</span></small>
          </li>
        </ul>
      </nav>
    );
  }
}
export default Navbar;
```

- Create scaffolding for the marketplace UI by creating a main file
- `src/components/Main.js` should look like the following:

```javascript
import React, { Component } from 'react';

class Main extends Component {

  render() {
    return (
      <div id="content">
        <h1>Add Product</h1>
        <form onSubmit={(event) => {
          event.preventDefault()
          const name = this.productName.value
          const price = window.web3.utils.toWei(this.productPrice.value.toString(), 'Ether')
          this.props.createProduct(name, price)
        }}>
          <div className="form-group mr-sm-2">
            <input
              id="productName"
              type="text"
              ref={(input) => { this.productName = input }}
              className="form-control"
              placeholder="Product Name"
              required />
          </div>
          <div className="form-group mr-sm-2">
            <input
              id="productPrice"
              type="text"
              ref={(input) => { this.productPrice = input }}
              className="form-control"
              placeholder="Product Price"
              required />
          </div>
          <button type="submit" className="btn btn-primary">Add Product</button>
        </form>
        <p> </p>
        <h2>Buy Product</h2>
        <table className="table">
          <thead>
            <tr>
              <th scope="col">#</th>
              <th scope="col">Name</th>
              <th scope="col">Price</th>
              <th scope="col">Owner</th>
              <th scope="col"></th>
            </tr>
          </thead>
          <tbody id="productList">
           { this.props.products.map((product, key) => {
             return(
               <tr key={key}>
                 <th scope="row">{product.id.toString()}</th>
                 <td>{product.name}</td>
                 <td>{window.web3.utils.fromWei(product.price.toString(), 'Ether')} Eth</td>
                 <td>{product.owner}</td>
                 <td>
                   { !product.purchased
                     ? <button
                         name={product.id}
                         value={product.price}
                         onClick={(event) => {
                           this.props.purchaseProduct(event.target.name, event.target.value)
                         }}
                       >
                         Buy
                       </button>
                     : null
                   }
                   </td>
               </tr>
             )
           })} 
          </tbody>
        </table>
      </div>
    );
  }
}

export default Main;
```

---

&nbsp;

## Part 4: Test App Transactions

- created 4 products with seller metamask account (1 banana, 2 breads, 1 crumb)
- changed to buyer account and purchased 1 banana and 1 bread

<figure>
<center><img src="/assets/images/product_list.png" style="width:100%">
<figcaption>list of products (2 belong to seller and 2 belong to buyer)</figcaption></center>
</figure>
<br>

<figure>
<center><img src="/assets/images/metamask_accounts.png" style="width:40%">
<figcaption>seller and buyer accounts in metamask with ETH balances after buyer purchased 2 items from seller</figcaption></center>
</figure>
<br>

<figure>
<center><img src="/assets/images/ganache_seller_buyer.png">
<figcaption>seller and buyer accounts in ganache with ETH balances after buyer purchased 2 items from seller</figcaption></center>
</figure>
<br>

<figure>
<center><img src="/assets/images/insufficient_funds.png" style="width:40%">
<figcaption>metamask notification after buyer attempts to purchase a product that costs more than the funds available</figcaption></center>
</figure>

---

&nbsp;

## Part 5: Final Thoughts

The greatest takeaway of walking through this lab was getting hands-on experience with tools like Ganache, Truffle, and the Metamask chrome extension. I can't say I'm any better of a developer by going through this tutorial. It was a lot of copy and pasting but this is only the beginning! The last time I touched object oriented programming was in college where I struggled in both C++ and Java and I didn't think I would ever intentionally come back to it. But I must say it was fun to work with Solidity and Javascript! Through this exercise, I learned more about smart contracts, the async/await pattern, and have a much better understanding of how decentralized apps are built and deployed 👍. Now that I've followed a step-by-step tutorial of creating a decentralized application on a personal blockchain, I plan on following another tutorial that will walk me through running my own Ethereum node or configuring a <a href="https://docs.chain.link/docs/running-a-chainlink-node/" target="_blank">Chainlink</a> node.
