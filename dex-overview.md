# Overview of Demo DEX

#### Links
- [Demo Video](https://www.youtube.com/watch?v=codcqb66G6Q)
- [Exchange Contract](https://github.com/wandevs/demo-dex-contracts/blob/master/contracts/Exchange.sol)

The demo exchange has two main features:
1. Token exchange on Wanchain using a smart contract to make the token transfers
2. Cross-chain mechanism where users can "deposit" Ethereum into their Wanchain
   account (the Ethereum integration was completely functional, but the Bitcoin
   integration was only capable of doing the first half of an outbound
   cross-chain transaction).

### Token Exchange

The exchange has a very simple token exchange process that involves a single
smart contract. The contract is callable only by the "owner" of the contract,
which we set to be the exchange backend. The contract basically is called only
once an order is ready to be filled, which initiates the token transfer. Thus
the exchange process looks like:
1. The maker makes an "approve" call to the token contract for the token they
   want to spend (for example, the WBTC token contract), granting
   permission for the exchange to transfer token on the user's behalf.
2. Once the approve call succeeds, an open order is added to the exchange.

There were two versions of the site, and the next steps in the order process
were different between the two versions. I'll talk about the first version,
where users trade directly with other users. In the second version, there was a
hidden taker who would take any submitted order (the video uses this second
version). With that said, the following steps in the order are:

3. The taker makes an "approve" contract to the token contract for the token
   they want to spend.
4. Once the approve call succeeds, the exchange backend calls `fulfillOrder`
   method on the exchange contract, which does the token swap, excluding a
   small percentage of token that is kept by the exchange as trading fees.
5. If successful, the order status is updated on the exchange, otherwise the
   order remains open.

NOTE: this design is only for demo purposes!

The exchange expects that the user has the WanMask extension installed on the
browser and does not work if the extension is not installed and open. For all
transactions, the exchange generates the transaction data, and then prompts
WanMask to sign and send the transaction. With this setup, the user is able to
hold their token in their own wallet and does not need to ever send their token
to any exchange account.

### Cross-chain

The cross-chain feature of the exchange allows users to "deposit" or
"withdrawal" Ethereum from their Wanchain account. By clicking on the
"ETH-WETH" icon, it prompts the user with the amount of Ethereum they want to
send to their Wanchain account. In later versions of the exchange there was
also another icon/button for starting the withdrawal.

NOTE: The demo video does not show WanMask for the cross-chain part. The demo
was hacked together and instead used a secret backend Ethereum account. It
ideally should have used MetaMask instead (I think at the time there was an
issue where you couldn't use WanMask and MetaMask at the same time, but that
issue is now resolved).

The process was not really finished, but for an inbound transaction it would
have worked like this:
1. User selects how much they want to send cross-chain.
2. MetaMask prompts the user to send a lock transaction.
3. After some time, once the token is ready to be redeemed on Wanchain,
   WanMask prompts the user to send the redeem transaction.
4. Token balances on the exchange update as WanMask updates with new balances.
