---
layout: post
author: lynn
tags: dapp blockchain poc
---


## Part 1: Project Setup, Initial Deployment, and Test
### 1. install dependencies
> macOS

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

- open Ganache and create new workspace
- screenshot


- migrate smart contract onto Ganache personal blockchain
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

## Part 2: Sell Products
