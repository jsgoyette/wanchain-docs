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
2. Install dependencies needed for wanx
```
$ npm install
```
3. Install dependencies needed for the examples
```
$ npm install keythereum node-bitcoin-rpc wanchainjs-tx
```

## Run bitcoin inbound example

1. Copy and edit Bitcoin inbound example
```
$ cd examples
$ cp btc2wbtc-complete-manual.js btc2wbtc-test.js
$ vi btc2wbtc-test.js
```
2. Update example script with correct values
- line 18: confirm wanchain node ip and port are correct
- line 23: update with the bitcoin RPC rpcuser/rpcpassword
- line 36: update with revoker bitcoin address (legacy address only, but 0 balance is okay)
```
$ bitcoin-cli -testnet getnewaddress '' legacy
```
- line 37: update with your Wanchain address
- line 38: update with the amount you want to send (in satoshis)
- line 52: update the path to the keystore file (user => youruser)
- line 54: update the keystore password
3. Run the script
```
> node btc2wbtc-test.js
```

## Run bitcoin outbound example

1. Copy and edit Bitcoin outbound example
```
$ cp wbtc2btc-complete-manual.js wbtc2btc-test.js
$ vi wbtc2btc-test.js
```
2. Update example script with correct values
- line 18: confirm wanchain node ip and port are correct
- line 23: update with the bitcoin RPC rpcuser/rpcpassword
- line 36: update with your Wanchain address
- line 37: update with redeemer bitcoin address (legacy address only, but 0 balance is okay)
```
$ bitcoin-cli -testnet getnewaddress '' legacy
```
- line 38: update with the destination bitcoin address where you want the bitcoin to be sent
- line 39: update with the amount you want to send (in satoshis)
- line 49: update with the WIF of the redeemer bitcoin address
```
> bitcoin-cli -testnet dumpprivkey <address>
```
- line 53: update with the amount of miner fee to be spent in the redeem
- line 56: update the path to the keystore file (user => youruser)
- line 58: update the keystore password
3. Run the script
```
> node wbtc2btc-test.js
```
