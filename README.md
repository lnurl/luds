LNURL Documents
===============

These are all the individual documents describing each small piece of protocol that can be implemented under the LNURL umbrella. Different wallets and services may implement different sets of protocols.

| Number      | Description                                                 | Wallets (in alphabetical order) |
|-------------|-------------------------------------------------------------|---------|
| [01](01.md) | Base LNURL encoding and decoding.                           | [Alby][alby], [Blixt][blixt], [BlueWallet][bluewallet], [Breez][breez], [coinos][coinos], [LNbits][lnbits], [@lntxbot][lntxbot], [Muun][muun], [Phoenix][phoenix], [ShockWallet][shockwallet], [SimpleBitcoinWallet][sbw], [ThunderHub][thunderhub], [Wallet of Satoshi][wos], [Zap Android][zap], [Zap Desktop][zap], [Zap iOS][zap], [ZBD Discord][zbd], [ZBD Extension][zbd], [ZBD Telegram][zbd], [ZEBEDEE][zbd], [Zeus][zeus] |
| [02](02.md) | `channelRequest` base spec.                                 | [Blixt][blixt], [Breez][breez], [SimpleBitcoinWallet][sbw], [Zap Android][zap], [Zap Desktop][zap], [Zeus][zeus] |
| [03](03.md) | `withdrawRequest` base spec.                                | [Blixt][blixt], [BlueWallet][bluewallet], [Breez][breez], [coinos][coinos], [LNbits][lnbits], [@lntxbot][lntxbot], [Muun][muun], [Phoenix][phoenix], [ShockWallet][shockwallet], [SimpleBitcoinWallet][sbw], [ThunderHub][thunderhub], [Wallet of Satoshi][wos], [Zap Android][zap], [Zap Desktop][zap], [Zap iOS][zap], [ZBD Discord][zbd], [ZBD Extension][zbd], [ZBD Telegram][zbd], [ZEBEDEE][zbd], [Zeus][zeus] |
| [04](04.md) | Auth base spec.                                             | [Alby][alby], [Blixt][blixt], [Breez][breez], [coinos][coinos], [LNbits][lnbits], [@lntxbot][lntxbot], [Phoenix][phoenix], [SimpleBitcoinWallet][sbw], [ThunderHub][thunderhub], [Zap Desktop][zap], [Zeus][zeus] |
| [05](05.md) | BIP32-based seed generation for auth protocol.              | [SimpleBitcoinWallet][sbw] |
| [06](06.md) | `payRequest` base spec.                                     | [Alby][alby], [Blixt][blixt], [BlueWallet][bluewallet], [Breez][breez], [coinos][coinos], [LNbits][lnbits], [@lntxbot][lntxbot], [Phoenix][phoenix], [ShockWallet][shockwallet], [SimpleBitcoinWallet][sbw], [ThunderHub][thunderhub], [Wallet of Satoshi][wos], [Zap Android][zap], [ZBD Discord][zbd], [ZBD Extension][zbd], [ZBD Telegram][zbd], [ZEBEDEE][zbd], [Zeus][zeus] |
| [07](07.md) | `hostedChannelRequest` base spec.                           | [SimpleBitcoinWallet][sbw] |
| [08](08.md) | Fast `withdrawRequest`.                                     | [@lntxbot][lntxbot], [SimpleBitcoinWallet][sbw], [ZBD Extension][zbd] |
| [09](09.md) | `successAction` field for `payRequest`.                     | [Alby][alby], [Blixt][blixt], [BlueWallet][bluewallet], [Breez][breez], [coinos][coinos], [LNbits][lnbits], [@lntxbot][lntxbot], [Phoenix][phoenix], [ShockWallet][shockwallet], [SimpleBitcoinWallet][sbw], [ThunderHub][thunderhub], [Wallet of Satoshi][wos], [Zap Android][zap], [ZBD Discord][zbd], [ZBD Extension][zbd], [ZBD Telegram][zbd], [ZEBEDEE][zbd], [Zeus][zeus] |
| [10](10.md) | `aes` success action in `payRequest`.                       | [Alby][alby], [Blixt][blixt], [BlueWallet][bluewallet], [Breez][breez], [coinos][coinos], [LNbits][lnbits], [@lntxbot][lntxbot], [Phoenix][phoenix], [ShockWallet][shockwallet], [SimpleBitcoinWallet][sbw], [ThunderHub][thunderhub], [Wallet of Satoshi][wos], [Zap Android][zap], [Zeus][zeus] |
| [11](11.md) | Disposable and storeable `payRequest`s.                     | [Blixt][blixt], [SimpleBitcoinWallet][sbw], [ZBD Extension][zbd] |
| [12](12.md) | Comments in `payRequest`.                                   | [Alby][alby], [Blixt][blixt], [BlueWallet][bluewallet], [Breez][breez], [LNbits][lnbits], [@lntxbot][lntxbot], [Phoenix][phoenix], [ThunderHub][thunderhub],  [Wallet of Satoshi][wos], [Zap Android][zap], [ZBD Discord][zbd], [ZBD Extension][zbd], [ZBD Telegram][zbd], [ZEBEDEE][zbd], [Zeus][zeus] |
| [13](13.md) | `signMessage`-based seed generation for auth protocol.      | [Alby][alby], [Blixt][blixt], [Zeus][zeus] |
| [14](14.md) | `balanceCheck`: reusable `withdrawRequest`s.                | [Blixt][blixt], [LNbits][lnbits], [@lntxbot][lntxbot] |
| [15](15.md) | `balanceNotify`: services hurrying up the withdraw process. | [LNbits][lnbits], [@lntxbot][lntxbot] |
| [16](16.md) | Paying to static internet identifiers.                      | [Alby][alby], [Blixt][blixt], [LNbits][lnbits], [@lntxbot][lntxbot], [Phoenix][phoenix], [ZBD Discord][zbd], [ZBD Extension][zbd], [ZBD Telegram][zbd], [ZEBEDEE][zbd] |
| [17](17.md) | Scheme prefixes and raw (non bech32-encoded) URLs.          | [Blixt][blixt] |
| [18](18.md) | Payer identity in `payRequest` protocol.                    | [SimpleBitcoinWallet][sbw] |
| [19](19.md) | Mutually discoverable pay and withdraw links.               |  |
| [20](20.md) | Long payment description for pay protocol.                  | [Blixt][blixt] |

[blixt]: https://blixtwallet.github.io
[bluewallet]: https://bluewallet.io
[sbw]: https://lightning-wallet.com
[breez]: https://breez.technology
[alby]: https://github.com/getAlby/lightning-browser-extension
[coinos]: https://coinos.io
[lnbits]: https://lnbits.org
[lntxbot]: https://lntxbot.com
[muun]: https://muun.com
[phoenix]: https://phoenix.acinq.co
[shockwallet]: https://shockwallet.app
[thunderhub]: https://www.thunderhub.io
[wos]: https://www.walletofsatoshi.com
[zap]: https://zaphq.io/
[zbd]: https://zebedee.io/wallet
[zeus]: https://zeusln.app

Dependency Tree
---------------

The entire idea of the suite of LNURL protocols is that they are optional. Each new LUD may be implemented by some wallets and not others, some services and not others, but they should still maintain compatibility at all times (except, of course, if the service _requires_ the new functionality).

There is also a hierarchy of internal dependencies between the LUDs -- for example, you cannot implement sending comments in `payRequest`s if you haven't implemented the base `payRequest` spec also. The following graph displays that hierarchy of internal dependencies (you can safely ignore it though).

[![dependencies](dependencies.png)](dependencies.dot)

How to add your own document
----------------------------

Open a pull request with your proposal. Pick the next unreserved number for your proposal. To be accepted it just have to be generally decent and make sense and be implemented or currently being implemented by 2 or more wallets.

Join https://t.me/lnurl if you just want to talk about your ideas or chat about LNURL protocols in general.
