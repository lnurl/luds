# Justifications for various decisions

## Mutually discoverable pay and withdraw links

`lnurl-pay` may contain a `withdrawLink` in its Json response and likewise `lnurl-withdraw` may contain a `payLink` in its Json response. This is done to recognize a fact that `LN SERVICE` may want to allow both deposits and withdrawals to user account and make related links easily discoverable from each other. When `LN WALLET` sees this it may store and show a compound item to user which would allow to both deposit and withdraw (and also to show an up-to-date `LN SERVICE` balance if `lnurl-withdraw` contains a `balanceCheck` field).