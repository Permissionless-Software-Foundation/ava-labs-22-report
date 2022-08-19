# AVAX DEX Summary
In 2021, [Ava Labs](https://www.avalabs.org/) awarded a grant to the [Permissionless Software Foundation](https://psfoundation.cash/) to develop a decentralized exchange (DEX) for the AVAX X-Chain. This report captures the progress that was made as a result of that grant.

## Scope
The application for the grant included [this presentation to Shapeshift DAO](https://youtu.be/XNvGjH57wdc), and this [proof of concept code](https://gist.github.com/christroutner/ac8810146ee3664c4ee8d6cb8bd66afe). The goal was to port the [SWaP protocol](https://github.com/vinarmani/swap-protocol) created by Vin Armani ([video presentation](https://youtu.be/jypfYJkdJ1k)) to the X-Chain. This protocol allows on-chain trading of tokens, peer-to-peer, with no intermediary. The protocol is *trustless*, meaning neither party can gain an advantage on the other. It is also *atomic*, meaning the trade is either complete or not. There is no in-between state where the trade can get stuck.

The scope of the grant was to create a graphical user interface (GUI) for trading tokens on the X-Chain, leveraging the SWaP protocol. At the start it was simply a proof-of-concept. The GUI now exists and functions, but additional funding will be required to improve the current user experience.

## Challenges
- Financial
  - Unprepared for the structure of the grant, half up front half at the end.
  - Drop in crypto prices
- Dependencies
  - AvalanchJS no longer compiles with Browserify
  - Full node change from cb58 to hex broke a lot of code

## Introduction to SWaP Protocol
SWaP is an acronym that stands for *Signal, Watch, and Pay*. These are the three (3) primary phases of the protocol. A workflow diagram illustrates the phases below. It describes how two users, Alice and Bob, complete a token trade with one another. There are specific words used to describe the process, which are capitalized. For definitions and technical description, read the [developer documentation](https://github.com/Permissionless-Software-Foundation/bch-dex/tree/master/dev-docs#definitions). The description below tries to describe the process in a non-technical manner.

![SWaP Protocol Workflow](./diagrams/swap-workflow.png)

### Signal
In the first phase, Alice has a token that she wants to sell on the open market. Using the GUI, she selects the token, the quantity to sell, and at what price. This generates an *Order*, which is tracked by the `avax-dex` app. The app also writes the trade data to the [pay-to-write database (P2WDB)](https://p2wdb.com) as an *Offer*. Alice is known as a *Maker*, because she just made an Offer.

### Watch
Bob wants to buy tokens, so he uses the GUI to browse tokens for sale on the open market. He sees Alice's listing and clicks the Buy button. His instance of `avax-dex` generates a *Counter Offer*. This is a partially-signed transaction based on data in the Offer. The Counte Offer is uploaded to the P2WDB.

Bob is known as a *Taker*, because he just took the Offer by generating a Counter Offer.

### Pay
Alice's local instance of the P2WDB triggers a webhook to her instance of `avax-dex`, when Bob's Counter Offer is written to the P2WDB. Alice's instance of `avax-dex` reads in the Counter Offer and validates it, ensuring that the transaction meets the terms she set in the Offer. If the Counter Offer passes validation, her instance of `avax-dex` signs the transaction and broadcasts it to the network.

The trade is then complete. Alice has the AVAX and Bob has the tokens. Both parties got what the wanted, and neither party has the opportunity to take advantage of the other (trustless). The trade completes in a single on-chain transaction (atomic), transferring the tokens and AVAX instantly and at the same time, with no need for escrow.
