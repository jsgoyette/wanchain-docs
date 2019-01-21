# Tutorial - Making a Bitcoin cross-chain transaction on Wanchain

## Intro
One of the main features of Wanchain is the built-out integrations with other
block chains. Today one can already make mainnet cross-chain transaction on
Wanchain with Bitcoin, Ethereum, and select ERC-20 tokens.

So let's say you wanted to build out an app that utilized Wanchain's cross-chain
feature. Naturally, you wouldn't want to build your app on top of the Wanchain
wallets, since they are intended to be used as a GUI and not as an
architectural backend. Instead, you would want to interact programmatically
with Wanchain, as well as the other chains, using lower-level APIs.

Currently, the easiest way for developers to make cross-chain transactions in
their code is with the [WanX](https://github.com/wandevs/wanx) npm package. The
purpose of this tutorial is to go over making cross-chain transactions with
WanX, by demonstrating an inbound and an outbound transaction with Bitcoin.

## Cross-chain overview

Before getting started, we need to review some of the basics of how cross-chain
transactions work on Wanchain. For a full overview of the cross-chain
implementation, check out [An Overview of the Wanchain Cross-Chain Implementation Model](https://medium.com/wanchain-foundation/an-overview-of-the-wanchain-2-0-cross-chain-implementation-model-c455cfd25664)
and the offical [Cross-Chain Implementation Reference](./). For our example case of
Bitcoin, there is also documentation on the WanX repo for making
[Inbound](https://github.com/wanchain/wanx/blob/dev/docs/btc-inbound.md)
and [Outbound](https://github.com/wanchain/wanx/blob/dev/docs/btc-outbound.md)
transactions.

The short version of the story is that there are (usually) four steps needed
to make a cross-chain transaction. For inbound transactions (converting native
coin to the corresponding token on Wanchain), the steps are as
follows.

#### Inbound cross-chain transaction steps
1. Make a transaction on the original chain that locks funds with a particular
   smart contract.
2. Wait for a followup contract call on Wanchain from the Storeman group that
   confirms the lock.
3. Make a smart contract call on Wanchain to redeem the token.
4. Wait for a followup contract call on the original chain from the Storeman
   group that finalizes the transfer.

Since Bitcoin doesn't have smart contracts, the first and last steps are
handled a bit differently for Bitcoin cross-chain transactions. Accordingly, to
make a Bitcoin inbound transaction, the first step is instead:

1. Create a new P2SH address (meeting certain criteria) and send bitcoin to it.
   Once the bitcoin transaction is sent to the network, then make a contract
   call on Wanchain that notifies the Storeman group of the lock.

And for Bitcoin, the last step is instead to wait for a contract call on
Wanchain, and not on Bitcoin.

Outbound transactions are basically the reverse of inbound transactions, but
with one important difference: unlike inbound transactions, outbound
transactions require that a locking fee is sent when the lock is made. Thus,
the lock for outbound transactions requires that you send the token that you would like
exchange back to the original asset, as well as a small fee in priced in WAN.
The outbound steps are thus as follows.

#### Outbound cross-chain transaction steps
1. Make a transaction on Wanchain that locks the tokens and that includes the
   lock fee in WAN.
2. Wait for a followup contract call on the original chain from the Storeman
   group that confirms the lock.
3. Make a smart contract call on the original chain to redeem the coin.
4. Wait for a followup contract call on Wanchain from the Storeman group that
   finalizes the transfer.

Again since Bitcoin does not have smart contracts, the 2nd and 3rd steps are
slightly different for Bitcoin. Also for the Bitcoin integration, the 4th step
is not required. For bitcoin, the steps are thus:

1. Same as above.
2. Wait for a followup contract call on Wanchain from the Storeman group that confirms the lock.
3. Make a bitcoin transaction that redeems the bitcoin locked by the Storeman group.

The steps to make a cross-chain transaction may be a bit confusing given that
the steps are slightly different for each chain. Just remember that the exact
inbound and outbound steps are laid out for each integration in the
documentation of the [WanX](https://github.com/wandevs/wanx) repository.

## Getting set up

### Set up nodes

To be able to do our example Bitcoin cross-chain transactions, we will need
to have access to Wanchain and Bitcoin nodes.

To get the nodes set up, you can refer to the following intructions.

- [Instructions to set up Wanchain node](./wanchain-setup.md)
- [Instructions to set up Bitcoin node](./bitcoin-setup.md)

### Create accounts

Now that we have Wanchain and Bitcoin nodes set up, we need to create new
addresses and make sure they are funded. For Wanchain, we can create a new
address keystore with `gwan`.

```bash
$ ./gwan --testnet account new
```

When creating a new account, it should prompt you for a passphrase for
encrypting the keystore file. Once the passphrase is entered it will then
create a new keystore file in your Wanchain directory (usually
`~/.wanchain/testnet/keystore`). For making a cross-chain transaction, all that
will be needed is the path to this keystore and the passphrase used to encrypt
it.

Now with the Wanchain address created, we need to add funds to it. The easiest
way to get some testnet WAN is to use a faucet, such as this one:
[Wanchain Testnet Faucet](https://faucet1.wanchain.org).

Next you'll need to set up a new Bitcoin address. You can use the `bitcoin-cli`
command to create a new Bitcoin address.
```
$ bitcoin-cli -testnet getnewaddress
```

Finally, with the new Bitcoin address ready, we just need to add some funds.
Again, the easiest way to get some testnet bitcoin is to use a faucet, such as
this one:
[https://lnroute.com/testnet-faucets/](https://lnroute.com/testnet-faucets/)

## Making an inbound bitcoin transaction

At this point you should have a testnet Wanchain node and a testnet Bitcoin
node running, and you should have a funded Wanchain address and a funded
Bitcoin address. With those in place, we are now ready to start building a
script to do an inbound bitcoin transaction. For those of you who want to cut
to the chase, you can check out the examples in the WanX repository, which the
following example is based from.

To get started, let's create a new directory and initialize npm in it.

```
$ mkdir crosschain-test
$ cd crosschain-test
$ npm init
```

Then let's add the dependencies we will be using in our example script.

```
npm install --save wanx web3 wanchainjs-tx keythereum node-bitcoin-rpc moment bignumber.js
```

Before setting up our script, let's add a file (`./btc-utils.js`) with a
utility function for sending bitcoin to a given address. We'll use this
`sendBtc` function to add funds to the lock address that we create.

** btc-utils.js **
```javascript
module.exports = {
  sendBtc,
};

function callRpc(bitcoinRpc, method, args) {
  return new Promise((resolve, reject) => {
    bitcoinRpc.call(method, args, (err, res) => {
      if (err) {
        return reject(err);
      } else if (res.error) {
        return reject(res.error);
      }

      resolve(res.result);
    });
  });
}

function sendBtc(bitcoinRpc, toAddress, toAmount, changeAddress) {
  return Promise.resolve([]).then(() => {

    return callRpc(bitcoinRpc, 'createrawtransaction', [[], { [toAddress]: toAmount }]);

  }).then(rawTx => {

    const fundArgs = { changePosition: 1 };

    if (changeAddress) {
      fundArgs.changeAddress = changeAddress;
    }

    return callRpc(bitcoinRpc, 'fundrawtransaction', [rawTx, fundArgs]);

  }).then(fundedTx => {

    return callRpc(bitcoinRpc, 'signrawtransactionwithwallet', [fundedTx.hex]);

  }).then(signedTx => {

    return callRpc(bitcoinRpc, 'sendrawtransaction', [signedTx.hex]);

  });
}

```

Now let's start on our script that will make the inbound Bitcoin cross-chain
transaction, which we'll call `btc-inbound.js`. To remind you, this script will
convert bitcoin (BTC) to the bitcoin token on Wanchain (wBTC), and it will do
so by following the four steps listed above.
