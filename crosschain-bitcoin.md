# Wanchain Bitcoin Integration How-To

This document describes the required setup and steps to run the bitcoin integration examples from the WanX npm package.

## Set up Wanchain node

There are a couple of options for setting up a Wanchain node: either install and open up the wallet, or install and run the Golang implementation.

### Option 1 - Wallet

1. Download and install the Wanchain wallet [https://wanchain.org](https://wanchain.org)
2. Open the wallet and let it synchronize
3. Switch to testnet
  -

### Option 2 - Golang

1. Download the binaries [https://github.com/wanchain/go-wanchain/releases](https://github.com/wanchain/go-wanchain/releases) or build from source.
2. Start the node in testnet mode
```bash
./gwan --testnet \
	--verbosity 4 --maxpeers 500 --maxpendpeers 100 --gasprice 180000000000 --txpool.nolocals \
	--rpc --rpcaddr 0.0.0.0 --rpcport 18545 --rpcapi "debug,eth,personal,net,admin,wan,txpool"
```
3. Test that you can connect to the Wanchain node from the command line
```bash
./gwan attach http://localhost:18545
```

## Set up Bitcoin node

1. Download and install Bitcoin Core [https://bitcoin.org](https://bitcoin.org), or install `bitcoind`
2. Use at least the following config settings (`~/.bitcoin/bitcoin.conf`)
```
server=1
rpcusername=myuser
rpcpassword=mypassword
txindex=1
```
If you want to be able to connect to rpc from another machine, also set the `rpcallowip` option
```
rpcallowip=192.168.0.0/24
```
3. Start `bitcoin-qt` or start the node daemon
```bash
bitcoind -testnet -daemon
```
4. Test that you can connect to the Bitcoin node from the command line
```bash
bitcoin-cli -testnet getbestblockhash
```
## Add testnet funds

### Wanchain

1. Create new Wanchain account in wallet
2. Get WAN from faucet
  - [fixme](fixme)

### Bitcoin

1. Get new address from GUI wallet or command line
```bash
bitcoin-cli -testnet getnewaddress
```

```bash
git clone https://github.com/wanchain/wanx
cd wanx
```
