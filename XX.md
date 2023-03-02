LUD-XX: LNURL Over Nostr

Author: kukks

---

## Introduction

LNURL has a requirement for DNS, which some see as a major weakness, as it can lead to centralized solutions. This LUD offers an alternative to LNURL by using [Nostr](https://github.com/nostr-protocol/nostr), which does not require an HTTP server.

## LNURL over Nostr

By using Nostr [NIP-04](https://github.com/nostr-protocol/nips/blob/master/04.md), [NIP-19](https://github.com/nostr-protocol/nips/blob/master/19.md), [NIP-21](https://github.com/nostr-protocol/nips/blob/master/21.md), it is possible to broadcast LNURL parameters and trigger the relevant specifications. This can be done without requiring an HTTP server for either the service or wallet.

### Using NIP-19 for LNURL

NIP-19 has several options, but for this proposal, we will focus on "nprofile," which includes a public key and relay hints. This is sufficient to establish a communication line for LNURL [LUD-01](01.md)  to function.

#### Example of Nprofile

When using nprofile for LNURL, it is assumed that the public key is exclusive to this LNURL usage. Here are the steps to create a nprofile:

1. Generate a new key for Nostr: `b7226fb958fc69906a20ee5029bb15500f92f47136a5a4fff653a1186cc4ef3b`.
2. Derive its public key: `fce2b7c9aa3019e65e808f47fe8cf99ae465103737f294a0444e6c5b53dee0c7`.
3. Create nprofile specifying `wss://r.x.com` as a relay hint: `nprofile1qqs0ec4hex4rqx0xt6qg73l73nue4er9zqmn0u555pzyumzm200wp3cpp4mhxue69uhhytnc9e3k7mgj4um4e`.
4. Create a NIP-21 URI: `nostr:nprofile1qqs0ec4hex4rqx0xt6qg73l73nue4er9zqmn0u555pzyumzm200wp3cpp4mhxue69uhhytnc9e3k7mgj4um4e`.
5. Encode it in bech32 as per [LUD-01](01.md) `lnurl1dehhxarj8fh8qun0ve5kcef3w9chxvr9vv6xsetcx3e8z7ps0p6rvut8xuekcdende6k2dr9wguh5utddcc82df4x4c857t4d4ax6v3sxpmhqvmrwpcrgmtg0p6k2d3ew45xs7t5de3njefndvmk6em2x36k6dr9ynmp4m`
6. Alternatively, use a URL as per [LUD-17](17.md): `lnurlp:nprofile1qqs0ec4hex4rqx0xt6qg73l73nue4er9zqmn0u555pzyumzm200wp3cpp4mhxue69uhhytnc9e3k7mgj4um4e`.

You can now advertise this LNURL.

### The Flow

1. The wallet scans/loads the LNURL.
2. The wallet connects to the relay specified in nprofile.
3. The wallet creates a new key for Nostr and sends a NIP-4 to the public key specified in nprofile, with empty content or a randomly generated string.
4. The wallet watches for a reply to the event ID created in the previous step from the nprofile public key.
5. The service creates a NIP-4 event replying to the wallet Nostr event, with JSON parameters. For example:
```json
{
"callback": "nostr:nprofile1qqs0ec4hex4rqx0xt6qg73l73nue4er9zqmn0u555pzyumzm200wp3cpp4mhxue69uhhytnc9e3k7mgj4um4e",
"metadata": "[[\"text/plain\",\"hello world\"]]",
"tag": "payRequest",
"minSendable": 1000,
"maxSendable": 1000000,
"commentAllowed": 200
}
```
Note that the `callback` property is optional in this variant. If specified, the next step is addressed to it; otherwise, it is addressed to the same Nostr LNURL.
6. The wallet detects the event, reads the parameters, and generates the query string parameters in the same format as usual. For example: `amount=1010&comment=whatever`
7. The wallet creates a new NIP-4 Nostr event with the query string parameters as its content and replies to either the wallet Nostr event or the callback Nostr note definition. This callback helps prevent metadata leakage of public LNURL usage. Tagging the previous event from the service is also optional and helps reduce the metadata leakage.
8. The service detects the event and sends the LNURL response JSON back in a NIP-4 reply.
9. The wallet handles it as per LNURL specs.

### Using Npub

Npub can also be used using the same flow as nprofile, but without providing a relay to guide the wallet where to connect. Supporting only one note reduces overall complexity.

### The Callback Property

The `callback` (or other similar properties found in lnurl-auth) property is optional. If left out, the callback property is the original nostr lnurl URI. If included, it could be set to an `nprofile` note. This allows the service to decouple the LNURL callback events, which works well when the LNURL is of npub or nprofile and reduces possible analysis of usage metrics.

### Asynchronous

As Nostr is asynchronous in nature, lnurl protocols also become asynchronous. When an lnurl "provider" is offline, a lnurl "taker" may still send his request. Once the provider is back online, they can query all events relating to the lnurl pubkey that were broadcast since they last processed.

## To Discuss

Should we replace NIP-4 with an ephemeral event? (this voids the async functionality mentioned above)

