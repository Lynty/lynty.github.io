---
layout: post
author: lynn
tags: dapp blockchain poc
---

> i'm on a macbook pro  

### 1. install dependencies
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

```solidity
pragma solidity ^0.5.0;

contract Marketplace {
    string public name;

    constructor() public {
        name = "Dapp University Marketplace";
}
```
