LUD-XX: Public key verification of hosted LN addresses
=====================================================

`author: kaloudis`

---

## Introduction

With the rise of Nostr and zaps there's been an increase in the use of lightning addresses. Some people host their own addresses, many people are using fully custodial services, somewhere in the middle are services that run the web server for people who run their nodes at home behind their routers, like [satdress](https://github.com/nbd-wtf/satdress) or [Alby](https://guides.getalby.com/overall-guide/alby-lightning-wallet/features/connect-your-own-node-to-your-alby-account).

The biggest problem with these services is that they can trivially steal funds by presenting senders with invoices to their own nodes. They can even do this very gradually over time, siphoning away a small percentage of funds so people don't realize that the theft is happening.

## Optional public key verification

This solution is a simple one: append the pubkey of the node that should be signing the invoices in the lightning address. Instead of providing the 33-byte pubkey we can provide a 20-byte SHA1 hash of it to save space.

eg. `evan@pay.zeusln.app#f1b5bbfa4072836ecb7f139d0d692c4491f1162b`

Payers' clients would check the verification hash against the SHA1 hash of the destination pubkey in the invoice they were delivered.

## Caveats

In a way this ruins the human readability of LN addresses, but it will be optional, even for users who want to use these types of services. For fully custodial services, this is redundant. For self-hosted users, this is largely unnecessary. Furthermore in the context of something like Nostr zaps, the UX is hardly degraded, if at all.

This solution does not explicitly work with blinded paths as the final destination is not embedded in the invoice.

## Acknowledgements

Bos for [inspiring the idea](https://twitter.com/alexbosworth/status/1630309388446748672). Although the solution proposed above also works for custodial accounts using the same node, whereas the solution proposed in the tweet does not.

