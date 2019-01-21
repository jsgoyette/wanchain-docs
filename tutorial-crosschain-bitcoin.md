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
make a Bitcoin inbound transaction, the steps are instead:

1. Create a new P2SH address (meeting certain criteria) and send bitcoin to it.
   Once the bitcoin transaction is sent to the network, then make a contract
   call on Wanchain that notifies the Storeman group of the lock.
2. Same as above.
3. Same as above.
4. Wait for a followup contract call on Wanchain from the Storeman group.

Outbound transactions (converting the token on Wanchain back to the
corresponding native coin) are basically the reverse of inbound transactions,
but with one important difference: unlike inbound transactions, outbound
transactions require that a locking fee is sent when the lock is made. Thus,
the lock for outbound transactions requires that you send the token that you
would like exchange back to the original asset, as well as a small fee in
priced in WAN.  The outbound steps are thus as follows.

#### Outbound cross-chain transaction steps
1. Make a transaction on Wanchain that locks the tokens and that includes the
   lock fee in WAN.
2. Wait for a followup contract call on the original chain from the Storeman
   group that confirms the lock.
3. Make a smart contract call on the original chain to redeem the coin.
4. Wait for a followup contract call on Wanchain from the Storeman group that
   finalizes the transfer.

Again since Bitcoin does not have smart contracts, the 2nd and 3rd steps are
slightly different for Bitcoin. Also, the 4th step is not required for the
Bitcoin integration. For Bitcoin, the steps are thus:

1. Same as above.
2. Wait for a followup contract call on Wanchain from the Storeman group that confirms the lock.
3. Make a bitcoin transaction that redeems the bitcoin locked by the Storeman group.

The steps to make a cross-chain transaction may be a bit confusing given that
the steps are slightly different for each chain. Just remember that the exact
inbound and outbound steps are laid out for each integration in the
documentation of the [WanX](https://github.com/wandevs/wanx) repository.

## Getting set up

### Set up nodes

To be able to do our example Bitcoin cross-chain transactions, you will need
to have access to Wanchain and Bitcoin nodes. You can refer to the following
intructions to get the nodes set up.

- [Instructions to set up Wanchain node](./wanchain-setup.md)
- [Instructions to set up Bitcoin node](./bitcoin-setup.md)

### Create accounts

Now with Wanchain and Bitcoin nodes set up, you'll need to create new addresses
and make sure they are funded. For Wanchain, you can create a new address
keystore with `gwan`.

```bash
$ ./gwan --testnet account new
```

When creating a new account, it should prompt you for a passphrase for
encrypting the keystore file. Once the passphrase is entered it will then
create a new keystore file in your Wanchain directory (usually
`~/.wanchain/testnet/keystore`). For making cross-chain transactions, all that
will be needed is the path to this keystore and the passphrase used to encrypt
it.

Now you'll need to add funds to our Wanchain account. The easiest way to get
some testnet WAN is to use a faucet, such as this one:
[Wanchain Testnet Faucet](https://faucet1.wanchain.org).

Next you'll need to set up a new Bitcoin address as well. You can use the
`bitcoin-cli` command to create a new Bitcoin address.

```
$ bitcoin-cli -testnet getnewaddress
```

Finally, with the new Bitcoin address ready, we just need to add some funds.
Again, the easiest way to get some testnet bitcoin is to use a faucet, such as
the ones listed here:
[https://lnroute.com/testnet-faucets/](https://lnroute.com/testnet-faucets/)

### Set up for development

At this point you should have a testnet Wanchain node and a testnet Bitcoin
node running, and you should have a funded Wanchain address and a funded
Bitcoin address. With those in place, we are now ready to set up our script to
do an cross-chain transactions. To get started, let's create a new directory
and initialize npm in it.

```
$ mkdir crosschain-test
$ cd crosschain-test
$ npm init
```

Then let's add the dependencies we will be using in our example scripts.

```
$ npm install --save wanx web3 wanchainjs-tx keythereum node-bitcoin-rpc moment bignumber.js
```

Before setting up our script, let's add a file (`./btc-utils.js`) with a
utility function for sending bitcoin to a given address. We'll use this
`sendBtc` function to add funds to the lock address that we create.

**btc-utils.js**
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

## Making an inbound bitcoin transaction

Now let's start on our script that will make the inbound Bitcoin cross-chain
transaction, which we will call `btc-inbound.js`. To remind you, this script
will convert bitcoin (BTC) to the bitcoin token on Wanchain (wBTC), and it will
do so by following the four steps listed above.

To start, let's add the necessary imports to the top of the script.

**btc-inbound.js**
```javascript
const WanX = require('wanx');
const Web3 = require('web3');
const keythereum = require('keythereum');
const WanTx = require('wanchainjs-tx');
const moment = require('moment');
const bitcoinRpc = require('node-bitcoin-rpc');
const BigNumber = require('bignumber.js');

const btcUtils = require('./btc-utils');
```

Next, let's set up an RPC connection with the bitcoin node.

```javascript
const btcNode = [ 'localhost', 18332, 'btcuser', 'btcpassword' ];

bitcoinRpc.init(...btcNode);
bitcoinRpc.setTimeout(2000);
```

Then set up WanX, as well as Web3 for communicating with the Wanchain node.

```javascript
const config = {
  wanchain: { url: 'http://localhost:18545' },
};

const wanx = new WanX('testnet', config);

const web3wan = new Web3(new Web3.providers.HttpProvider(config.wanchain.url));
```

For the final piece of the setup, let's use keythereum and unlock our Wanchain
account private key. Make sure to put in the correct Wanchain address, the
correct path to the keystore file, as well as the correct keystore passphrase.

```javascript
const wanAddress = '<myWanchainAddress>';

const wanDatadir = '/home/<myuser>/.wanchain/testnet/';
const wanKeyObject = keythereum.importFromFile(wanAddress, wanDatadir);
const wanPrivateKey = keythereum.recover('mypassword', wanKeyObject);
```

Now we are ready to initialize a new wanx chain and start our cross-chain
transaction. First, let's set up a new chain with wanx, using the `newChain`
method. The 1st argument specifies the chain we want to connect with, in this
case "btc", and the 2nd argument indicates that this will be an inbound
transaction (`true` for inbound, and `false` for outbound).

```javascript
// bitcoin, inbound
const cctx = wanx.newChain('btc', true);
```

Our final setup action will be to define the transaction options. Each chain
type requires certain parameters to be passed within the options, which can be
deduced from the [WanX Documentation](https://wanchain/wanx).

In the options we need to define the Bitcoin address that will be the revoker
address, in the case that we need to revoke the transaction. The revoker
address must be a legacy address, and it does not need to be funded. Thus, for
the revoker we will create a new legacy bitcoin address for the sole purpose of
revoking.

```bash
$ bitcoin-cli -testnet getnewaddress '' legacy
```

Now add the revoker address to our `btc-inbound.js` script.

```javascript
const revokerAddress = 'mvTfNujpcQwHaefMxfJRix4vhfNBxSFbBe';
```

At last, let's define the transaction options.

```javascript
const opts = {
  // Revoker bitcoin address
  from: revokerAddress,

  // Recipient wanchain address
  to: wanAddress,

  // Value in satshis
  value: '210000',

  // Storeman group addresses
  storeman: {
    wan: '0x9ebf2acd509e0d5f9653e755f26d9a3ddce3977c',
    btc: '0x83e5ca256c9ffd0ae019f98e4371e67ef5026d2d',
  },

  // Generate a new redeemKey
  redeemKey: wanx.newRedeemKey('sha256'),
};
```

The options includes the revokerAddress and wanAddress, the amount to be sent
(210000 satoshis), the addresses of the Storeman group, and a new redeemKey.
The redeemKey includes two parts, a random string (which is the transaction
identifier) and the hash of the random string (which is the key needed to
redeem the token). For the case of Bitcoin, we need the redeemKey hash to be a
SHA256 hash.

With our transaction options defined, we are finally at the point where we can
actually start the transaction. The following snippet kicks off the transaction
and runs through all of the required four steps through a chain of promises.

```javascript
Promise.resolve([]).then(() => {

  // Step 1a: generate a new P2SH lock address and send bitcoin to it

  // Log our options
  console.log('Starting btc inbound lock', opts);

  // Create new P2SH lock address
  const contract = cctx.buildHashTimeLockContract(opts);

  // Log the P2SH address details
  console.log('Created new BTC contract', contract);

  // Add the lockTime to opts
  opts.lockTime = contract.lockTime;

  // Convert BTC amount from satoshis to bitcoin
  const sendAmount = (new BigNumber(opts.value)).div(100000000).toString();

  console.log('Send amount', sendAmount);

  // Send BTC to P2SH lock address
  return btcUtils.sendBtc(bitcoinRpc, contract.address, sendAmount);

}).then(txid => {

  // Step 1b: save the txid of the funding transaction

  // Add txid to opts
  opts.txid = txid;

  console.log('BTC tx sent', txid);

  // Get the tx count to determine next nonce
  return web3wan.eth.getTransactionCount(opts.to);

}).then(txCount => {

  // Step 1c: send the lock notice on Wanchain

  // Construct the raw lock tx
  const lockTx = cctx.buildLockTx(opts);

  // Add nonce to tx
  lockTx.nonce = web3wan.utils.toHex(txCount);

  // Sign and serialize the tx
  const transaction = new WanTx(lockTx);
  transaction.sign(wanPrivateKey);
  const serializedTx = transaction.serialize().toString('hex');

  // Send the lock transaction on Wanchain
  return web3wan.eth.sendSignedTransaction('0x' + serializedTx);

}).then(receipt => {

  // Step 2: wait for the Storeman group to confirm the lock

  console.log('Lock submitted and now pending on storeman');
  console.log(receipt);

  // Scan for the lock confirmation from the storeman
  return cctx.listenLock(opts, receipt.blockNumber);

}).then(log => {

  console.log('Lock confirmed by storeman');
  console.log(log);

  // Get the tx count to determine next nonce
  return web3wan.eth.getTransactionCount(opts.to);

}).then(txCount => {

  // Step 3: make Wanchain contract call to redeem the wBTC token

  // Get the raw redeem tx
  const redeemTx = cctx.buildRedeemTx(opts);

  // Add nonce to tx
  redeemTx.nonce = web3wan.utils.toHex(txCount);

  // Sign and serialize the tx
  const transaction = new WanTx(redeemTx);
  transaction.sign(wanPrivateKey);
  const serializedTx = transaction.serialize().toString('hex');

  // Send the lock transaction on Wanchain
  return web3wan.eth.sendSignedTransaction('0x' + serializedTx);

}).then(receipt => {

  // Step 4: wait for the Storeman group to confirm the redeem

  console.log('Redeem submitted and now pending on storeman');
  console.log(receipt);

  // Scan for the lock confirmation from the storeman
  return cctx.listenRedeem(opts, receipt.blockNumber);

}).then(log => {

  console.log('Redeem confirmed by storeman');
  console.log(log);
  console.log('COMPLETE!!!');

}).catch(err => {

  console.log('Error:', err);

});
```

As you can see, the script starts by generating a new P2SH address
(`buildHashTimeLockContract`) and sending bitcoin to it (`sendBtc`). The
`lockTime` and `txid` are saved to `opts`, as they are needed parameters in the
next step where we send a lock notice contract call on Wanchain
(`buildLockTx`). Once the lock notice is sent, it waits for a response from the
storeman (`listenLock`). After the Storeman group confirms the lock, it then
sends a redeem call on Wanchain (`buildRedeemTx`), and then finally waits for
the Storeman group to confirm the redeem (`listenRedeem`).
