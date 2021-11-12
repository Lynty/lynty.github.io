---
layout: post
author: lynn
tags: ethereum blockchain geth node mining
---

* T0C
{:toc}

---
<!--
<a href="" target="_blank"></a>

&nbsp;

<figure><center><img src="/assets/images/" style="width:100%">
<figcaption></figcaption></center></figure><br>
-->

## Why Run an Ethereum Node?

- learning purposes
- <a href="https://ethereum.org/en/developers/docs/nodes-and-clients/#why-should-i-run-an-ethereum-node" target="_blank">personal and network benefits</a>

## Installing Geth

- Geth is an open-source client written in Go
  - there are plenty of clients to choose from but I've been trying to learn Go
- start a node in a docker container
  - connecting to the Görli test network to start
  - mount volume as client's data directory to ensure downloaded data is preserved between restarts and/or container lifecycles
    - `mkdir /Users/ldong/Projects/Learning_Labs/blockchain/geth/goerli`

```bash
docker run --name geth -v /Users/ldong/Projects/Learning_Labs/blockchain/geth/goerli:/root/.ethereum/geth/goerli \
  -p 8545:8545 -p 30303:30303 -dit \
  ethereum/client-go:latest --goerli
```

- here are the ports that are automatically exposed:
  - `8545` TCP, used by the HTTP based JSON RPC API
  - `8546` TCP, used by the WebSocket based JSON RPC API
  - `8547` TCP, used by the GraphQL API
  - `30303` TCP and UDP, used by the P2P protocol running the network

## What's Happening?

- `docker logs geth -f`

```
INFO [11-10|19:39:50.265] Starting Geth on Görli testnet...
INFO [11-10|19:39:50.274] Maximum peer count                       ETH=50 LES=0 total=50
INFO [11-10|19:39:50.274] Smartcard socket not found, disabling    err="stat /run/pcscd/pcscd.comm: no such file or directory"
WARN [11-10|19:39:50.276] Sanitizing cache to Go's GC limits       provided=1024 updated=662
INFO [11-10|19:39:50.276] Set global gas cap                       cap=50,000,000
INFO [11-10|19:39:50.277] Allocated trie memory caches             clean=99.00MiB dirty=165.00MiB
INFO [11-10|19:39:50.277] Allocated cache and file handles         database=/root/.ethereum/goerli/geth/chaindata cache=329.00MiB handles=524,288
INFO [11-10|19:39:50.303] Opened ancient database                  database=/root/.ethereum/goerli/geth/chaindata/ancient readonly=false
INFO [11-10|19:39:50.303] Writing custom genesis block
INFO [11-10|19:39:50.311] Persisted trie from memory database      nodes=361 size=51.17KiB time=1.4678ms gcnodes=0 gcsize=0.00B gctime=0s livenodes=1 livesize=0.00B
INFO [11-10|19:39:50.312] Initialised chain configuration          config="{ChainID: 5 Homestead: 0 DAO: <nil> DAOSupport: true EIP150: 0 EIP155: 0 EIP158: 0 Byzantium: 0 Constantinople: 0 Petersburg: 0 Istanbul: 1561651, Muir Glacier: <nil>, Berlin: 4460644, London: 5062605, Arrow Glacier: <nil>, Engine: clique}"
INFO [11-10|19:39:50.314] Initialising Ethereum protocol           network=5 dbversion=<nil>
```

- connecting to geth can be done via RPC or IPC
  - RPC = remote node connection over HTTP
  - IPC = direct node connection using JavaScript console (can do more with IPC)
    - it stands for Interprocess Communication and is a mechanism for establishing a connection between processes which allows data flow between them
- let's hop into the container with `docker exec -it geth sh`
- connect via IPC
  - `geth attach /root/.ethereum/goerli/geth.ipc`

```
Welcome to the Geth JavaScript console!

instance: Geth/v1.10.13-unstable-0efed7f5-20211109/linux-amd64/go1.17.3
at block: 0 (Wed Jan 30 2019 13:26:31 GMT+0000 (UTC))
 datadir: /root/.ethereum/goerli
 modules: admin:1.0 clique:1.0 debug:1.0 eth:1.0 miner:1.0 net:1.0 personal:1.0 rpc:1.0 txpool:1.0 web3:1.0
```

- gather peer information from within the JavaScript console such as number of peers and detailed info of peers, check block number

```javascript
> net.peerCount
1
> admin.peers
[{
    caps: ["eth/65", "eth/66", "les/2", "les/3", "les/4", "snap/1"],
    enode: "enode://63037fb2d7072e30fdcafeb386e6fc9edab80b5b4b933a1a799d28343cde857de2b3ca1c33266f50740ce1cecdd99aefe49c469cc1186a349eb8cd4f19b4dc33@65.19.136.133:30303",
    enr: "enr:-KO4QB77nof221Oksl_L-S-yiWVIjTJx1Tx4o-vkA8YqiJgpBTQi4DwYpUJQoZzxzJQTt9f06_ZSwF5ugPmOo-odV2QFg2V0aMfGhLjGKZ2AgmlkgnY0gmlwhEETiIWDbGVzwQGJc2VjcDI1NmsxoQNjA3-y1wcuMP3K_rOG5vye2rgLW0uTOhp5nSg0PN6FfYRzbmFwwIN0Y3CCdl-DdWRwgnZf",
    id: "551ff030a46f72e1bb26d3bbc2f0d3cc1b6e01a6b588e651095a25150b360fd8",
    name: "Geth/v1.10.8-stable-26675454/linux-amd64/go1.16.4",
    network: {
      inbound: false,
      localAddress: "172.17.0.2:58772",
      remoteAddress: "65.19.136.133:30303",
      static: false,
      trusted: false
    },
    protocols: {
      eth: {
        difficulty: 8536335,
        head: "0xcfe8c31251f3dc585e78fdf4c1381dcfb31b0c32727ccb67f0a63db3e25f90a6",
        version: 66
      },
      snap: {
        version: 1
      }
    }
}]
> eth.blockNumber
0
```


- thoughts
  - I should probably reference the <a href="https://web3js.readthedocs.io/en/v1.2.11/index.html" target="_blank">web3.js docs</a>
  - why am I only connected to one peer? is that normal?
  - why is the blockNumber 0?
    - found the <a href="https://github.com/ethereum/go-ethereum/issues/16411#issuecomment-377176640" target="_blank">answer </a>from the lead geth developer himself
    - (the reason has to do with fast syncing)

## Creating Accounts

- `geth --goerli account new`

```
Your new key was generated

Public address of the key:   0xC3e10Ac93531E1C8c541AC117F410d00a855996e
Path of the secret key file: /root/.ethereum/goerli/keystore/UTC--2021-11-10T23-33-36.940215200Z--c3e10ac93531e1c8c541ac117f410d00a855996e
```

- in console: `eth.accounts`

```
["0xc3e10ac93531e1c8c541ac117f410d00a855996e"]
```

## Conclusion

I played around and struggled with this for a decent amount of time. It is SUPER easy to run a node but it's not so easy understanding what's going on in the backend. Some of the things I attempted to do:

- connect to metamask extension via RPC
- migrate my [last deployed smart contract]({% post_url 2021-11-08-building-a-blockchain-app %}) to Görli test network
- import and export accounts
- run dev mode network
- set up a private network with genesis json file
  - I learned that ganache is a lot more user friendly than geth

I haven't even begun to try and do some mining. That's for another day. For now, I'll continue poking around in the Görli test network. I have to make a tweet or post something on Facebook to get Ether from the <a href="https://faucet.goerli.mudit.blog/" target="_blank">Goerli Authenticated Faucet</a> so I think right now is a good break point. I have my node up and running inside a Docker container and I now understand how to run `geth` and javascript console commands!
