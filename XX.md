LUD-XX: LNURL Over Nostr.
==============================================

`author: kukks`

---

## Using LNURL over Nostr

Many opponents of LNURL always point to the DNS requirement of LNURL as its biggest weakness, as it makes wallets require a publicly accessible HTTP endpoint, often resulting in centralized custodial solutions offering dominantly offering LNURL support.

This LUD presents an alternative, where neither `SERVICE` nor `WALLET` requires even an HTTP server, using [Nostr](https://github.com/nostr-protocol).

By utilizing [NIP-04](https://github.com/nostr-protocol/nips/blob/master/04.md)), [NIP-19](https://github.com/nostr-protocol/nips/blob/master/19.md)), [NIP-21](https://github.com/nostr-protocol/nips/blob/master/21.md)), and [NIP-33](https://github.com/nostr-protocol/nips/blob/master/33.md)), we can broadcast lnurl parameters and have its callbacks trigger the respective specification.

  ### Various NIP19 scenarios
  
  NIP19 offers various note options that can point to public keys (`nprofile`), specific events (`nevent`), or parameters of a particular event (`naddr`). All 3 can be used to encode sufficient data to form a communication line for LNURL LUD-01 to function. We will also prefix with the uri `nostr:` scheme from NIP-21 to stay uri compliant within LNURL.

#### Nprofile example
When using `nprofile` for LNURL, it is assumed that the pubkey is exclusive for this lnurl usage.

* Generate a new key for nostr: `b7226fb958fc69906a20ee5029bb15500f92f47136a5a4fff653a1186cc4ef3b`
* Derive its public key: `fce2b7c9aa3019e65e808f47fe8cf99ae465103737f294a0444e6c5b53dee0c7`
* Create nprofile (specifying `wss://r.x.com` as a relay hint) `nprofile1qqs0ec4hex4rqx0xt6qg73l73nue4er9zqmn0u555pzyumzm200wp3cpp4mhxue69uhhytnc9e3k7mgj4um4e`
* Create nip21 uri : `nostr:nprofile1qqs0ec4hex4rqx0xt6qg73l73nue4er9zqmn0u555pzyumzm200wp3cpp4mhxue69uhhytnc9e3k7mgj4um4e`
* Encode it in bech32 as per LUD01 `lnurl1dehhxarj8fh8qun0ve5kcef3w9chxvr9vv6xsetcx3e8z7ps0p6rvut8xuekcdende6k2dr9wguh5utddcc82df4x4c857t4d4ax6v3sxpmhqvmrwpcrgmtg0p6k2d3ew45xs7t5de3njefndvmk6em2x36k6dr9ynmp4m`
* or as a url as per LUD17 `lnurlp:nprofile1qqs0ec4hex4rqx0xt6qg73l73nue4er9zqmn0u555pzyumzm200wp3cpp4mhxue69uhhytnc9e3k7mgj4um4e`

You can now advertise this LNURL.

* Wallet scans/loads LNURL
* Wallet connects to the relay specified in nprofile
* Wallet creates a new key for nostr and send a NIP4 to pubkey specified in nprofile, with empty content or a randomly generated string (to discuss)
* Wallet watches for a reply to event id created in previous step from nprofile pubkey
* Service creates a NIP4 event replying to wallet nostr event, with json parameters. In this case 
```
{
  "callback": "nostr:nprofile1qqs0ec4hex4rqx0xt6qg73l73nue4er9zqmn0u555pzyumzm200wp3cpp4mhxue69uhhytnc9e3k7mgj4um4e",
  "metadata": "[[\"text/plain\",\"hello world\"]]",
  "tag": "payRequest",
  "minSendable": 1000,
  "maxSendable": 1000000,
  "commentAllowed": 200
}
```
Please note that the `callback` property is optional in this variant. If specified that next step is addressed to it, else it is addressed to service nostr event itself.
* Wallet detects the event, reads parameters, and generates the query string parameters in the same format as usual, meaning `amount=1010&comment=whatever`
* Wallet creates a new nostr event, with the query string parameters as its content. and replies to either the wallet nostr event, or to the callback nostr note definition. This callback helps prevent metadata leakage of public lnurl usage.
* Service detects event, and send the lnurl response json back in NIP4 as a reply.
* Wallet handles it as per lnurl specs

#### NEvent example
* Service creates a nostr event, where its content is the lnurl json, with event id being `c0e9e10110659220b25f83961d297194e596d2188943153918663d4904c478c7`
* Services encodes it to nevent, `nevent1qqsvp60pqygxty3qkf0c89sa99cefevk6gvgjsc48yvxv02fqnz833cpp4mhxue69uhhytnc9e3k7mgalulyu` 
* Create nip21 uri : `nostr:nevent1qqsvp60pqygxty3qkf0c89sa99cefevk6gvgjsc48yvxv02fqnz833cpp4mhxue69uhhytnc9e3k7mgalulyu`
* Encode to lnurl bech32 or LUD17
* Wallet scans/loads LNURL
* Wallet connects to the relay specified in nevent
* Wallet creates a new key for nostr
* Wallet scans for event with id from nevent
* Wallet detects the event, reads parameters, and generates the query string parameters in the same format as usual, meaning `amount=1010&comment=whatever`
* Wallet creates a new nostr event, with the query string parameters as its content. and replies to either the wallet nostr event, or to the callback nostr note definition. This callback helps prevent metadata leakage of public lnurl usage.
* Service detects event and sends the lnurl response json back in NIP4 as a reply.
* Wallet handles it as per lnurl specs

#### NAddr example

Same as nevent, except that scanning for event requires using kind, author, and `d` identifier. The benefits of using naddr over nevent, is that it allows you to update the parameters without needing to issue a new lnurl


### Npub can also be used using the identical flow of `nprofile`, though no relay can be provided to guide the wallet where to connect to and would be less likely to find the event.

### Callback property

As mentioned, the `callback` (or other similar properties found for example in lnurl-auth) property is optional, and if left out, the callback mechanism is to tag the event that gave the parameters. 
If however it is included, it could be set to an `nevent`, `nprofile`, `naddr`, `npub`. If the note had a public key, it would tag it in the event, if it had an event, it would tag it. This allows the Service to decouple the lnurl callback events, which works well when the lnurl is of npub or nprofile and reduces possible analysis of usage metrics.


### To Discuss
* Replace NIP4 with an ephemeral event?
* Should we reduce the options as many duplicate functionality? To what?
