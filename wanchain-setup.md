# Set up Wanchain node for development

These instructions cover how to set up a Wanchain node with RPC enabled, so
that the node can be accessed by the console and by web3.

Overall, the steps include getting gwan, running gwan with the correct options,
and testing that you can connect to the node via RPC. There are some options on
how to install gwan, but the subsequent steps should all be the same.

## Install gwan

#### Option 1 - Wallet

1. Download and install the Wanchain wallet [https://wanchain.org/products](https://wanchain.org/products)
2. Start the node in testnet mode
```bash
~/.config/WanWalletGui/binaries/Gwan/unpacked/gwan --testnet \
	--verbosity 4 --maxpeers 500 --maxpendpeers 100 --gasprice 180000000000 --txpool.nolocals \
	--rpc --rpcaddr 0.0.0.0 --rpcport 18545 --rpcapi "debug,eth,personal,net,admin,wan,txpool"
```


#### Option 2 - Pre-built gwan binary

1. Download the binary [https://wanchain.org/products](https://wanchain.org/products)
```bash
wget https://wanchain.org/download/gwan-linux-amd64-1.0.7-3c1c638c.tar.gz
tar xzvf gwan-linux-amd64-1.0.7-3c1c638c.tar.gz
cd gwan-linux-amd64-1.0.7-3c1c638c
```
2. Start the node in testnet mode
```bash
./gwan --testnet \
	--verbosity 4 --maxpeers 500 --maxpendpeers 100 --gasprice 180000000000 --txpool.nolocals \
	--rpc --rpcaddr 0.0.0.0 --rpcport 18545 --rpcapi "debug,eth,personal,net,admin,wan,txpool"
```

#### Option 3 - Build from source

1. Install and configure Golang [https://golang.org/doc/install](https://golang.org/doc/install)
2. Get and build go-wanchain
```bash
go get github.com/wanchain/go-wanchain
cd $GOPATH/src/github.com/wanchain/go-wanchain
make gwan
```
3. Start the node in testnet mode
```bash
./build/bin/gwan --testnet \
	--verbosity 4 --maxpeers 500 --maxpendpeers 100 --gasprice 180000000000 --txpool.nolocals \
	--rpc --rpcaddr 0.0.0.0 --rpcport 18545 --rpcapi "debug,eth,personal,net,admin,wan,txpool"
```

## Test connection

Check to see that you can connect to the Wanchain node from the command line.
Replace the path to gwan with the path that you used above to start the node.

```bash
./gwan attach http://localhost:18545
```
