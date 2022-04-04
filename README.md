LNURL Documents
===============

These are all the individual documents describing each small piece of protocol that can be implemented under the LNURL umbrella. Different wallets and services may implement different sets of protocols.

| Number      | Description                                                 | Wallets (in alphabetical order) |
|-------------|-------------------------------------------------------------|---------|
| [01](01.md) | Base LNURL encoding and decoding.                           | _all the ones listed below_ |
| [02](02.md) | `channelRequest` base spec.                                 | [Blixt][blixt], [Breez][breez], [SimpleBitcoinWallet][sbw], [Zap Android][zap], [Zap Desktop][zap], [Zeus][zeus] |
| [03](03.md) | `withdrawRequest` base spec.                                | [Blixt][blixt], [BlueWallet][bluewallet], [Breez][breez], [coinos][coinos], [LNbits][lnbits], [@lntxbot][lntxbot], [Muun][muun], [Phoenix][phoenix], [ShockWallet][shockwallet], [SimpleBitcoinWallet][sbw], [ThunderHub][thunderhub], [Wallet of Satoshi][wos], [Zap Android][zap], [Zap Desktop][zap], [Zap iOS][zap], [ZBD Discord][zbd], [ZBD Extension][zbd], [ZBD Telegram][zbd], [ZEBEDEE][zbd], [Zeus][zeus] |
| [04](04.md) | Auth base spec.                                             | [Alby][alby], [Blixt][blixt], [Breez][breez], [coinos][coinos], [LNbits][lnbits], [@lntxbot][lntxbot], [Phoenix][phoenix], [SeedAuth](seedauth), [SeedAuthExtension](sae), [SimpleBitcoinWallet][sbw], [ThunderHub][thunderhub], [Zap Desktop][zap], [Zeus][zeus] |
| [05](05.md) | BIP32-based seed generation for auth protocol.              | [coinos][coinos] [SimpleBitcoinWallet][sbw] |
| [06](06.md) | `payRequest` base spec.                                     | [Alby][alby], [Blixt][blixt], [BlueWallet][bluewallet], [Bottlepay][bottlepay], [Breez][breez], [BTCPayServer][btcp], [coinos][coinos], [Galoy][galoy], [LNbits][lnbits], [LNLink][lnlink], [LNPay.co](lnpay), [@lntxbot][lntxbot], [LightningTipBot][ltb], [Phoenix][phoenix], [ShockWallet][shockwallet], [SimpleBitcoinWallet][sbw], [ThunderHub][thunderhub], [Wallet of Satoshi][wos], [Zap Android][zap], [ZBD Discord][zbd], [ZBD Extension][zbd], [ZBD Telegram][zbd], [ZEBEDEE][zbd], [Zeus][zeus] |
| [07](07.md) | `hostedChannelRequest` base spec.                           | [SimpleBitcoinWallet][sbw] |
| [08](08.md) | Fast `withdrawRequest`.                                     | [@lntxbot][lntxbot], [SimpleBitcoinWallet][sbw], [ZBD Extension][zbd] |
| [09](09.md) | `successAction` field for `payRequest`.                     | [Alby][alby], [Blixt][blixt], [BlueWallet][bluewallet], [Breez][breez], [coinos][coinos], [LNbits][lnbits], [@lntxbot][lntxbot], [Phoenix][phoenix], [ShockWallet][shockwallet], [SimpleBitcoinWallet][sbw], [ThunderHub][thunderhub], [Wallet of Satoshi][wos], [Zap Android][zap], [ZBD Discord][zbd], [ZBD Extension][zbd], [ZBD Telegram][zbd], [ZEBEDEE][zbd], [Zeus][zeus] |
| [10](10.md) | `aes` success action in `payRequest`.                       | [Alby][alby], [Blixt][blixt], [BlueWallet][bluewallet], [Breez][breez], [coinos][coinos], [LNbits][lnbits], [@lntxbot][lntxbot], [Phoenix][phoenix], [ShockWallet][shockwallet], [SimpleBitcoinWallet][sbw], [ThunderHub][thunderhub], [Wallet of Satoshi][wos], [Zap Android][zap], [Zeus][zeus] |
| [11](11.md) | Disposable and storeable `payRequest`s.                     | [Blixt][blixt], [SimpleBitcoinWallet][sbw], [ZBD Extension][zbd] |
| [12](12.md) | Comments in `payRequest`.                                   | [Alby][alby], [Blixt][blixt], [BlueWallet][bluewallet], [Breez][breez], [LNbits][lnbits], [@lntxbot][lntxbot], [Phoenix][phoenix], [ThunderHub][thunderhub],  [Wallet of Satoshi][wos], [ZBD Discord][zbd], [ZBD Extension][zbd], [ZBD Telegram][zbd], [ZEBEDEE][zbd], [Zeus][zeus] |
| [13](13.md) | `signMessage`-based seed generation for auth protocol.      | [Alby][alby], [Blixt][blixt], [Zeus][zeus] |
| [14](14.md) | `balanceCheck`: reusable `withdrawRequest`s.                | [Alby][alby], [Blixt][blixt], [LNbits][lnbits], [@lntxbot][lntxbot] |
| [15](15.md) | `balanceNotify`: services hurrying up the withdraw process. | [LNbits][lnbits], [@lntxbot][lntxbot] |
| [16](16.md) | Paying to static internet identifiers.                      | [Alby][alby], [Blixt][blixt], [BTCPayServer][btcp], [LNbits][lnbits], [LNLink][lnlink], [@lntxbot][lntxbot], [LightningTipBot][ltb], [Phoenix][phoenix], [SimpleBitcoinWallet][sbw], [Zap Android][zap], [ZBD Discord][zbd], [ZBD Extension][zbd], [ZBD Telegram][zbd], [ZEBEDEE][zbd], [Zeus][zeus] |
| [17](17.md) | Scheme prefixes and raw (non bech32-encoded) URLs.          | [Alby][alby], [Blixt][blixt], [BTCPayServer][btcp], [@lntxbot][lntxbot], [ZBD Discord][zbd], [ZBD Telegram][zbd] |
| [18](18.md) | Payer identity in `payRequest` protocol.                    | [Blixt][blixt], [@lntxbot][lntxbot], [ZBD Discord][zbd], [ZBD Telegram][zbd] |
| [19](19.md) | Pay link discoverable from withdraw link.                   | [Blixt][blixt], [SimpleBitcoinWallet][sbw] |
| [20](20.md) | Long payment description for pay protocol.                  | [Alby][alby], [Blixt][blixt], [@lntxbot][lntxbot], [Phoenix][phoenix] |

[alby]: https://github.com/getAlby/lightning-browser-extension
[blixt]: https://blixtwallet.github.io
[bluewallet]: https://bluewallet.io
[bottlepay]: https://bottlepay.com
[btcp]: https://btcpayserver.org
[breez]: https://breez.technology
[coinos]: https://coinos.io
[galoy]: https://galoy.io
[lnbits]: https://lnbits.org
[lnlink]: https://lnlink.app
[lnpay]: https://lnpay.co
[lntxbot]: https://lntxbot.com
[ltb]: https://ln.tips
[muun]: https://muun.com
[phoenix]: https://phoenix.acinq.co
[sae]: https://github.com/pseudozach/seedauthextension
[sbw]: https://lightning-wallet.com
[seedauth]: https://seedauth.etleneum.com/
[shockwallet]: https://shockwallet.app
[thunderhub]: https://www.thunderhub.io
[wos]: https://www.walletofsatoshi.com
[zap]: https://zaphq.io/
[zbd]: https://zebedee.io/wallet
[zeus]: https://zeusln.app

Services
--------

| Name                                                                            | LUDs                                                           |
| ----                                                                            | ----                                                           |
| [Azteco](https://azte.co/)                                                      | [01][01] [03][03]                                              |
| [BTC Origin Stories](https://btcoriginstories.com/)                             | [01][01] [06][06]                                              |
| [Bitcoin Bounce](https://thndr.games/)                                          | [01][01] [03][03] [08][08]                                     |
| [Bitrefill](https://bitrefill.com/)                                             | [01][01] [02][02] [06][06] [16][16]                            |
| [Bottlepay](https://bottlepay.com/)                                             | [01][01] [06][06]                                              |
| [CoinCorner](https://www.coincorner.com)                                        | [01][01] [02][02] [03][03] [06][06] [16][16]                   |
| [Etleneum](https://etleneum.com/)                                               | [01][01] [03][03] [04][04] [06][06] [09][09] [14][14] [15][15] |
| [Galoy](https://galoy.io/)                                                      | [01][01] [06][06]                                              |
| [Going Dutch](https://goingdutch.pm/)                                           | [01][01] [03][03] [06][06]                                     |
| [HangarSix](https://www.hangarsixgaming.com/)                                   | [01][01] [03][03]                                              |
| [Infuse](https://zebedee.io/infuse/)                                            | [01][01] [03][03]                                              |
| [Kollider](https://kollider.xyz/)                                               | [01][01] [03][03] [04][04]                                     |
| [Kriptode](https://kriptode.com/)                                               | [01][01] [03][03] [04][04]                                     |
| [LNBIG](https://lnbig.com/)                                                     | [01][01] [02][02]                                              |
| [ln.cash](https://ln.cash)                                                      | [01][01] [03][03]                                              |
| [LNMarkets](https://lnmarkets.com/)                                             | [01][01] [03][03] [04][04] [06][06] [16][16]                   |
| [LNPay.co](https://lnpay.co)                                                    | [01][01] [03][03] [06][06] [14][14]                            |
| [LNbits.com](https://lnbits.com/)                                               | [01][01] [03][03] [04][04] [14][14] [15][15]                   |
| [Lightning.Video](https://lightning.video/)                                     | [01][01] [04][04]                                              |
| [Lightning Gifts](https://lightning.gifts/)                                     | [01][01] [03][03] [06][06] [12][12]                            |
| [LightningTipBot](https://ln.tips/)                                             | [01][01] [06][06] [09][09] [12][12] [16][16]                   |
| [@lntxbot](https://t.me/lntxbot)                                                | [01][01] [03][03] [06][06] [09][09] [12][12] [16][16] [18][18] |
| [Lnurl-Pay Chat](https://chat.blixtwallet.com/)                                 | [01][01] [06][06] [11][11] [12][12] [18][18]                   |
| [Loft](https://loft.trade/)                                                     | [01][01] [04][04]                                              |
| [Microlancer](https://microlancer.io/)                                          | [01][01] [03][03] [14][14]                                     |
| [OpenNode](https://developers.opennode.com/reference/initiate-lnurl-withdrawal) | [01][01] [03][03]                                              |
| [Paywall](https://paywall.link)                                                 | [01][01] [03][03]                                              |
| [Rewards to Bitcoin](rtb)                                                       | [01][01] [03][03] [04][04]                                     |
| [Satsback.com](https://satsback.com)                                            | [01][01] [03][03]                                              |
| [SouthXchange](https://www.southxchange.com/)                                   | [01][01] [03][03]                                              |
| [Surfcity Parking](https://surfcity.app/)                                       | [01][01] [03][03]                                              |
| [Wheel of Fortune](https://fortune.lngames.net)                                 | [01][01] [03][03] [04][04]                                     |
| [bridgeaddr](https://bridgeaddr.fiatjaf.com)                                    | [01][01] [06][06] [09][09] [12][12] [16][16]                   |
| [coinos](https://coinos.io/)                                                    | [01][01] [03][03] [04][04] [06][06] [16][16]                   |
| [lnshort.it](https://lnshort.it/)                                               | [01][01] [04][04] [06][06]                                     |
| [lnsms.world](https://lnsms.world/)                                             | [01][01] [06][04] [11][11] [12][12] [16][16]                   |
| [lnurl-pay.me](https://lnurl-pay.me)                                            | [01][01] [06][06] [16][16]                                     |
| [pollofeed](https://pollofeed.com)                                              | [01][01] [06][06] [16][16]                                     |
| [stacker.news](https://stacker.news/)                                           | [01][01] [03][03] [04][04] [06][06] [16][16]                   |
| [zbd.gg](https://zbd.gg/)                                                       | [01][01] [03][03] [06][06] [09][09] [12][12] [16][16]          |

[rtb]: https://play.google.com/store/apps/details?id=com.pseudozach.rewardstobitcoin

Self-hosted
-----------

| Name                                                                                       | LUDs                                         |
| ----                                                                                       | ----                                         |
| [Addressless](https://github.com/futurepaul/addressless)                                   | [01][01] [06][06] [09][09] [16][16]          |
| [Bleskomat](https://github.com/samotari/bleskomat)                                         | [01][01] [03][03]                            |
| [BTCPayServer](https://btcpayserver.org/)                                                  | [01][01] [06][06] [16][16] [17][17]          |
| [Citadel Bitcoin Node](https://github.com/runcitadel)                                      | [01][01] [06][06] [16][16]                   |
| [LNURLPoS](https://github.com/arcbtc/LNURLPoS)                                             | [01][01] [06][06] [09][09]                   |
| [LNURLp](https://github.com/lnbits/lnbits/tree/master/lnbits/extensions/lnurlp)            | [01][01] [06][06] [12][12]                   |
| [LNURLw](https://github.com/lnbits/lnbits/tree/master/lnbits/extensions/withdraw)          | [01][01] [03][03]                            |
| [LNbits.com](https://github.com/fiatjaf/lnbits)                                            | [01][01] [03][03] [04][04] [14][14] [15][15] |
| [LightningATM](https://github.com/21isenough/LightningATM)                                 | [01][01] [03][03]                            |
| [NextPay](https://github.com/apotdevin/NextPay)                                            | [01][01] [06][06] [16][16]                   |
| [OfflineShop](https://github.com/lnbits/lnbits/tree/master/lnbits/extensions/offlineshop)  | [01][01] [06][06] [09][09]                   |
| [Satdress](https://github.com/fiatjaf/satdress)                                            | [01][01] [06][06] [09][09] [16][16]          |
| [Streamer Copilot](https://github.com/lnbits/lnbits/tree/master/lnbits/extensions/copilot) | [01][01] [06][06] [12][12]                   |
| [go-host-lnaddr](https://github.com/hieblmi/go-host-lnaddr)                                | [01][01] [06][06] [09][09] [16][16]          |
| [ligess](https://github.com/Dolu89/ligess/)                                                | [01][01] [06][06] [09][09] [16][16]          |
| [lnme](https://github.com/bumi/lnme)                                                       | [01][01] [06][06] [09][09] [16][16]          |

Libraries
---------

| Name                                                  | LUDs                                                                                                                          |
| ----                                                  | ----                                                                                                                          |
| [IMMORTAN](https://github.com/btcontract/IMMORTAN)    | [01][01] [02][02] [03][03] [04][04] [05][05] [06][06] [07][07] [08][08] [09][09] [10][10] [11][11] [12][12] [16][16]          |
| [LNURL C#](https://github.com/Kukks/LNURL)            | [01][01] [02][02] [03][03] [04][04] [06][06] [08][08] [09][09] [10][10] [11][11] [12][12] [14][14] [15][15] [16][16] [19][19] |
| [dart-lnurl](https://github.com/bottlepay/dart_lnurl) | [01][01] [02][02] [03][03] [04][04] [06][06]                                                                                  |
| [go-lnurl](https://github.com/fiatjaf/go-lnurl)       | [01][01] [02][02] [03][03] [04][04] [06][06] [08][08] [09][09] [10][10] [11][11] [12][12] [14][14] [16][16] [18][18] [20][20] |
| [js-lnurl](https://github.com/fiatjaf/js-lnurl)       | [01][01] [02][02] [03][03] [04][04] [06][06] [08][08] [09][09] [10][10] [11][11] [12][12]                                     |
| [lnurl-node](https://github.com/chill117/lnurl-node)  | [01][01] [02][02] [03][03] [04][04] [06][06] [08][08] [09][09] [10][10] [11][11] [12][12]                                     |
| [lnurl-platformio](platformio)                        | [01][01] [03][03]                                                                                                             |
| [lnurl-ruby](https://github.com/bumi/lnurl-ruby)      | [01][01] [06][06] [09][09]                                                                                                    |
| [lnurlauth](https://github.com/xplorfin/lnurlauth)    | [01][01] [04][04]                                                                                                             |
| [php-lnurl](https://github.com/tkijewski/php-lnurl)   | [01][01]                                                                                                                      |
| [python-lnurl](https://github.com/python-ln/lnurl)    | [01][01] [02][02] [03][03] [04][04] [06][06]                                                                                  |
| [rust-lnurl](rust)                                    | [01][01] [03][03] [04][04]                                                                                                    |
| [spring-lnurl](springlnurl)                           | [01][01] [04][04]                                                                                                             |

[rust]: https://github.com/edouardparis/rust-lnurl
[platformio]: https://github.com/chill117/lnurl-platformio
[springlnurl]: https://github.com/theborakompanioni/bitcoin-spring-boot-starter#spring-lnurl

Tools for developers
--------------------

| Name                                                                   | LUDs                                                                                                        |
| ----                                                                   | ----                                                                                                        |
| [LNURLProxyAPI](https://github.com/21isenough/LNURLProxyAPI)           | [01][01] [03][03]                                                                                           |
| [Lightning Decoder](https://lightningdecoder.com/)                     | [01][01] [02][02] [03][03] [04][04] [06][06] [16][16]                                                       |
| [Lightning Login](https://lightninglogin.live/)                        | [01][01] [04][04]                                                                                           |
| [bifrost](https://github.com/takinbo/bifrost)                          | [01][01] [02][02]                                                                                           |
| [lnch-vekslak](https://github.com/Kixunil/lnch-vekslak)                | [01][01] [02][02]                                                                                           |
| [lnurl codec JS](https://lnurl.fiatjaf.com/codec)                      | [01][01]                                                                                                    |
| [lnurl codec Scala](https://j-chimienti.github.io/lnurl_codec/)        | [01][01]                                                                                                    |
| [lnurl playground](https://lnurl.fiatjaf.com)                          | [01][01] [02][02] [03][03] [04][04] [06][06] [09][09] [10][10] [12][12] [14][14] [15][15] [18][18] [20][20] |
| [lnurl-toolbox](https://lnurl-toolbox.degreesofzero.com/)              | [01][01] [02][02] [03][03] [04][04] [06][06]                                                                |
| [passport-lnurl-auth](https://github.com/chill117/passport-lnurl-auth) | [01][01] [04][04]                                                                                           |

[01]: 01.md
[02]: 02.md
[03]: 03.md
[04]: 04.md
[05]: 05.md
[06]: 06.md
[07]: 07.md
[08]: 08.md
[09]: 09.md
[10]: 10.md
[11]: 11.md
[12]: 12.md
[13]: 13.md
[14]: 14.md
[15]: 15.md
[16]: 16.md
[17]: 17.md
[18]: 18.md
[19]: 19.md
[20]: 20.md

Dependency Tree
---------------

The entire idea of the suite of LNURL protocols is that they are optional. Each new LUD may be implemented by some wallets and not others, some services and not others, but they should still maintain compatibility at all times (except, of course, if the service _requires_ the new functionality).

There is also a hierarchy of internal dependencies between the LUDs -- for example, you cannot implement sending comments in `payRequest`s if you haven't implemented the base `payRequest` spec also. The following graph displays that hierarchy of internal dependencies (you can safely ignore it though).

[![dependencies](dependencies.png)](dependencies.dot)

How to add your own document
----------------------------

Open a pull request with your proposal. Pick the next unreserved number for your proposal. To be accepted, it just has to be generally decent and make sense and be implemented or currently being implemented by 2 or more wallets.

Join https://t.me/lnurl if you just want to talk about your ideas or chat about LNURL protocols in general.
