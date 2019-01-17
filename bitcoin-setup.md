# Set up Bitcoin node for Wanchain development

These instructions cover how to set up a Bitcoin node with RPC enabled, so
that the node can be accessed through scripting.

## Install bitcoind

#### Option 1 - Install Bitcoin Core
1. Download and install Bitcoin Core [https://bitcoin.org](https://bitcoin.org)

#### Option 2 - Install bitcoind from ppa
```bash
sudo apt-add-repository ppa:bitcoin/bitcoin
sudo apt-get update
sudo apt-get install bitcoind
```
## Configure
Use at least the following config settings (`~/.bitcoin/bitcoin.conf`)
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

#### Start bitcoind
```bash
bitcoind -testnet -daemon
```

#### Check connection
Test that you can connect to the Bitcoin node from the command line
```bash
bitcoin-cli -testnet getbestblockhash
```
