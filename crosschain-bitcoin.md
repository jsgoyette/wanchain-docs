# Wanchain Bitcoin Integration How-To

This document describes the required setup and steps to run the bitcoin integration examples from the WanX npm package.

## Set up Wanchain node

- [Instructions to set up Wanchain node](./wanchain-setup.md)

## Set up Bitcoin node

- [Instructions to set up Bitcoin node](./bitcoin-setup.md)

## Add testnet funds

### Wanchain

1. Create new Wanchain account in wallet
```
$ ./gwan --testnet account new
```
2. Get testnet WAN from faucet
  - [https://faucet1.wanchain.org/](https://faucet1.wanchain.org/)

### Bitcoin

1. Get new address from GUI wallet or command line
```bash
$ bitcoin-cli -testnet getnewaddress
```
2. Get testnet bitcoin from faucet
  - [https://lnroute.com/testnet-faucets/](https://lnroute.com/testnet-faucets/)

```bash
$ git clone https://github.com/wanchain/wanx
$ cd wanx
```
