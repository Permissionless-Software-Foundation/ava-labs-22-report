# AVAX DEX Summary
In 2021, [Ava Labs](https://www.avalabs.org/) awarded a grant to the [Permissionless Software Foundation](https://psfoundation.cash/) to develop a decentralized exchange (DEX) for the AVAX X-Chain. This report captures the progress that was made as a result of that grant.

## Scope
The application for the grant included [this presentation to Shapeshift DAO](https://youtu.be/XNvGjH57wdc), and this [proof of concept code](https://gist.github.com/christroutner/ac8810146ee3664c4ee8d6cb8bd66afe). The goal was to port the [SWaP protocol](https://github.com/vinarmani/swap-protocol) created by Vin Armani ([video presentation](https://youtu.be/jypfYJkdJ1k)) to the X-Chain. This protocol allows on-chain trading of tokens, peer-to-peer, with no intermediary. The protocol is *trustless*, meaning neither party can gain an advantage on the other. It is also *atomic*, meaning the trade is either complete or not. There is no in-between state where the trade can get stuck.

The scope of the grant was to create a graphical user interface (GUI) for trading tokens on the X-Chain, leveraging the SWaP protocol. At the start it was simply a proof-of-concept. The GUI now exists and functions, but additional funding will be required to improve the current user experience.

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

## Contributors
The creation of this app was a collaboration by three developers:
- [Chris Troutner](https://github.com/christroutner)
- [Gary Nadir](https://github.com/MezzMar)
- [Daniel Gonzalez](https://github.com/danielhumgon)

Gary did the heavy-lifting by developing most of the back-end and X-chain specific code. Daniel and Chris both contributed heavily to the graphical user interface. Chris was the lead architect and researcher.

## Challenges
Any crypto project has it's challenges, and this one was no different.

- **Geographical Challenges** - Gary and Daniel are both located in Venezuela. Although both are fluent in English, the spotty internet in Venezuela often made meetings and communication difficult.

- **Financial Challenges** - We were unprepared for the financial structure of the grant, with half being paid upfront and half being paid at the end. This meant we effectively had half the amount of money we had budgeted. In addition, there was a massive crash in cryptocurrency prices, which negatively effected the financial cushion that we had set aside.

- **Dependency Challenges** - There was a three month gap between Gary finishing the proof of concept code, and Chris integrating his work into the final app. During that time, [AvalanchJS stopped compiling](https://github.com/ava-labs/avalanchejs/issues/622) forcing us to use an older version. Also the full node transitioned from [cb58 to hex format](https://github.com/ava-labs/avalanchejs/issues/623) which broke a lot of code and requied significant refactoring.

Despite these challenges, our team was able to complete primary objective of the grant: To create an app for peer-to-peer trading of tokens on the X-Chain.

## Future Progress
It is hoped that this work will lead to additional grants from AVA Labs in order to complete the progress on this app. While it's functional, it's clunky and prone to errors. The user experience is very poor. Here are some of the major features that could be improved with additional funding:

- **Robust Error Handling** - To complete each phase of the SWaP protocol, a complex series of network calls are made. If any of these network calls fail, the user is required to start over. Better handling of errors, and automatic retry of network calls, would go a long way towards creating a better user experience.
- **Port P2WDB to AVAX** - The [P2WDB](https://p2wdb.com) exists only on the Bitcoin Cash blockchain, which requires users to have some BCH and PSF tokens in order to write to it and complete a trade. If the P2WDB was ported to the AVAX X-Chain, it would remove this clunky user experience and make every run natively on the X-Chain.
- **Standalone Buyer App** - While a *Maker* must run a copy of `avax-dex` in order to create and complete trades, a *Taker* does not. Will additional work, it would be possible to make a stand-alone Android app that allows users to brows the market and buy tokens, without any need for a back-end server. This would create an ideal user experience.
- **Allow Buy Orders** - Currently the app is only designed for *selling* tokens. But the SWaP protocol can also support *buy* Orders. Currently Makers only create Orders to sell tokens, but Makers could also create Orders to buy tokens.
