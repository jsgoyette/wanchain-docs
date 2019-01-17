# Wanchain Bitcoin Integration How-To

This document describes the required setup and steps to run the bitcoin integration examples from the WanX npm package.

## Set up Wanchain node

- [Instructions to set up Wanchain node](./wanchain-setup.md)

## Set up Bitcoin node

- [Instructions to set up Bitcoin node](./bitcoin-setup.md)

## Create accounts and add funds

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
2. Get tBTC from faucet
  - [https://lnroute.com/testnet-faucets/](https://lnroute.com/testnet-faucets/)

## Set up WanX examples

1. Clone the WanX repository
```bash
$ git clone https://github.com/wanchain/wanx
$ cd wanx
```
2. Install dependencies needed for example
```
$ npm install
$ npm install keythereum node-bitcoin-rpc wanchainjs-tx
```
3. Copy and edit Bitcoin inbound example
```
$ cd examples
$ cp btc2wbtc-complete-manual.js btc2wbtc-test.js
$ vi btc2wbtc-test.js
```
4. Update example script with correct values
- line 18: confirmed wanchain node ip/port are correct
- line 23: update with the bitcoin RPC rpcuser/rpcpassword
- line 36: update with revoker bitcoin address (can be any legacy address, even with 0 balance)
- line 37: update with your Wanchain address
- line 38: update with the amount you want to send (in satoshis)
- line 52: update the path to the keystore file (user => youruser)
- line 54: update the keystore password
5. Run the script
```
> node btc2wbtc-test.js
```
