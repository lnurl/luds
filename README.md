LNURL Documents
===============

These are all the individual documents describing each small piece of protocol that can be implemented under the LNURL umbrella. Different wallets and services may implement different sets of protocols.

| Number      | Description                                                          |
|-------------|----------------------------------------------------------------------|
| [01](01.md) | Base LNURL encoding and decoding.                                    |
| [02](02.md) | `channelRequest` base spec.                                          |
| [03](03.md) | `withdrawRequest` base spec.                                         |
| [04](04.md) | Auth base spec.                                                      |
| [05](05.md) | BIP32-based seed generation for auth protocol.                       |
| [06](06.md) | `payRequest` base spec.                                              |
| [07](07.md) | `hostedChannelRequest` base spec.                                    |
| [08](08.md) | Fast `withdrawRequest`.                                              |
| [09](09.md) | `successAction` field for `payRequest`.                              |
| [10](10.md) | `aes` success action in `payRequest`.                                |
| [11](11.md) | Disposable and storeable `payRequest`s.                              |
| [12](12.md) | Comments in `payRequest`.                                            |
| [13](13.md) | LND-based seed generation for auth protocol.                         |
| [14](14.md) | `balanceCheck`: reusable `withdrawRequest`s.                         |
| [15](15.md) | `balanceNotify`: services hurrying up the withdraw process.          |
| [16](16.md) | Paying to static internet identifiers.                               |
| [17](17.md) | Scheme prefixes and raw (non bech32-encoded) URLs.                   |
| [18](18.md) | Proof-of-payer in `payRequest` protocol.                             |
| [19](19.md) | Mutually discoverable pay and withdraw links.                        |

Dependency Tree
---------------

[![dependencies](dependencies.png)](dependencies.dot)

Wallets
-------

| Wallet                                                            | 01 | 02 | 03 | 04 | 05 | 06 | 07 | 08 | 09 | 10 | 11 | 12 | 13 | 14 | 15 | 16 | 17 | 18 | 19 |
|-------------------------------------------------------------------|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|
| [BLW](https://lightning-wallet.com/)                              |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |
| [Blixt](https://github.com/hsjoberg/blixt-wallet)                 |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |
| [BlueWallet](https://bluewallet.io/)                              |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |
| [Breez](https://breez.technology/)                                |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |
| [coinos](https://coinos.io/)                                      |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |
| [LNbits](https://lnbits.org/)                                     |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |
| [@lntxbot](https://t.me/lntxbot)                                  |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |
| [Muun](https://muun.com/)                                         |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |
| [Phoenix](https://phoenix.acinq.co/)                              |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |
| [Shockwallet](https://shockwallet.app/)                           |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |
| [ThunderHub](https://github.com/apotdevin/thunderhub)             |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |
| [Wallet of Satoshi](https://www.walletofsatoshi.com/)             |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |
| [Zap-Android](https://www.zaphq.io/)                              |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |
| [Zap-iOS](https://www.zaphq.io/)                                  |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |
| [ZEBEDEE](https://zbd.gg) (and [bots](https://zebedee.io/bots/))  |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |
| [Zeus](https://github.com/ZeusLN/zeus)                            |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |    |

Services
--------

How to add your own document
----------------------------

Open a pull request with your proposal. Pick the next unreserved number for your proposal. To be accepted it just have to be generally decent and make sense and be implemented or currently being implemented by 2 or more wallets.

Join https://t.me/lnurl if you just want to talk about your ideas or chat about LNURL protocols in general.
