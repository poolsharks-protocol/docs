# User Interaction

## What It Does

This is the layer which users will interact with directly to perform any action of funds in regards to DCEX. It contains a set of Smart Contracts that, at their most basic level, provide a way for a user to emit a given event. In more advanced cases, it can be a hybrid contract, where an external user can interact with the liquidity inside DCEX directly, without having to perform a deposit or withdraw, while still emitting an event such that any changes can take effect in the database.

```solidity
function transfer(
    address transferToken,
    uint256 transferAmount,
    address receiver
) external override {
    address sender = msg.sender;
    emit TransferRequest(sender, transferToken, transferAmount, receiver);
}
```

## Abstraction

Since funds are being stored in a common contract between users, and funds can be spent by modules, we don't require certian EIPs to be used in the traditonal way (in all scenarios). 

Let's take for example an ERC20 transfer to your friend:

Normally a user with a wallet would call to the ERC20 `transfer(address dest, uint256 amount)` method and use up about `30,000-60,000` gas units.

In the case a DCEX user wants to transfer ERC20 token internally, they would call to the DCEX ERC20 Core contract's `transfer(address token, uint256 amount, address receiver)` method, consuming approximately `24,000-26,000` gas units. Since funds are local, there is no need to execute any code on-chain other than the event is firing off. The [Ingestion Layer](http://localhost:8000/intro/layers/ingestion/) will handle rassigning balances as needed.

Taking this to the protocol level, we can imagine a DCEX user interacting with a Module like GroupSwap, which allows users to pool gas together for on-chain swaps. Instead of a user calling to the ERC20 contract's `approve()` method and the DEX contract's `swap` method, they can call to the relevant Module contract's `groupSwap()` function.

Each request for the GroupSwap module consumes approximately `25,000-27,000` gas units. The amount of gas units consumed by a request to a Module can and will vary based on the amount of data emitted. 

The DCEX user doesn't need to perform an `approve()`, because the `groupSwap()` function calls to the Core contract's `reserveToken()` function. In this design, reservations can be seen as approvals, where a reservation is defined by a user calling to a module which then calls to the Core ERC20 contract to reserve or unreserve the token.

## User Functionality

Any actions a user will perform is through a contract call to the Core or any authorized Module. For info on active Modules and what each function does in detal, see the developer docs.

## Production Requirements

**AUDITS REQUIRED**

This layer should be production ready; there are not any major differences between these contracts and any other protocol's contracts'.