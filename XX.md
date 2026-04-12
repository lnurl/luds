LUD-XX: LNURL Over Nostr

Author: kukks

---

## Introduction

LNURL requires DNS and HTTPS, which creates a dependency on centralized infrastructure. This LUD offers an alternative transport for LNURL using [Nostr](https://github.com/nostr-protocol/nostr), removing the need for an HTTP server on either side.

## Prerequisites

This specification uses:
- [NIP-17](https://github.com/nostr-protocol/nips/blob/master/17.md) — Private Direct Messages (Gift Wrap)
- [NIP-19](https://github.com/nostr-protocol/nips/blob/master/19.md) — bech32-encoded entities (`naddr`, `nprofile`)
- [NIP-21](https://github.com/nostr-protocol/nips/blob/master/21.md) — `nostr:` URI scheme
- [NIP-33](https://github.com/nostr-protocol/nips/blob/master/33.md) — Parameterized Replaceable Events
- [NIP-44](https://github.com/nostr-protocol/nips/blob/master/44.md) — Versioned Encryption
- [NIP-59](https://github.com/nostr-protocol/nips/blob/master/59.md) — Gift Wrap

## Addressing

Two NIP-19 addressing modes are supported. The wallet detects which mode is in use from the NIP-19 prefix in the `nostr:` URI.

### `naddr` — Public LNURL Parameters (recommended)

The service publishes its LNURL parameters as a **parameterized replaceable event** ([NIP-33](https://github.com/nostr-protocol/nips/blob/master/33.md)). The `naddr` encodes a coordinate (kind + pubkey + `d` tag + relay hints) that always resolves to the latest version of that event.

This is the recommended mode for LNURL types with static parameters (`payRequest`, `channelRequest`) because:
- The wallet fetches parameters directly from the relay — no round trip to the service.
- The service can update parameters at any time by publishing a replacement event.
- The `naddr` itself never changes.
- The relay serves the event instantly, even if the service is offline.

#### LNURL Parameter Event

The service publishes a Kind **31120** parameterized replaceable event:

```json
{
  "kind": 31120,
  "pubkey": "<service-pubkey>",
  "created_at": 1234567890,
  "tags": [["d", "payRequest"]],
  "content": "{\"tag\":\"payRequest\",\"minSendable\":1000,\"maxSendable\":1000000,\"metadata\":\"[[\\\"text/plain\\\",\\\"hello world\\\"]]\",\"commentAllowed\":200}",
  "sig": "..."
}
```

The `d` tag MUST match the LNURL tag value (e.g. `payRequest`, `withdrawRequest`, `channelRequest`). The `content` field contains the JSON LNURL response exactly as it would appear over HTTP, with one difference: the `callback` property is optional (see [Callback](#the-callback-property)).

#### Constructing an `naddr` LNURL

1. Generate a new Nostr key pair dedicated to this LNURL endpoint.
2. Publish the Kind 31120 event to one or more relays.
3. Construct the `naddr`: kind `31120`, pubkey, `d` tag (e.g. `payRequest`), relay hints.
4. Create a [NIP-21](https://github.com/nostr-protocol/nips/blob/master/21.md) URI: `nostr:<naddr>`.
5. Encode as bech32 per [LUD-01](01.md) or as a [LUD-17](17.md) scheme URI.

#### Example

Encoded as bech32 LNURL:
```
lnurl1dp68gurn8ghj7...
```

Or as LUD-17 scheme:
```
lnurlp:naddr1qvzqqqr4gupzp...
```

### `nprofile` — Interactive DM Flow

For LNURL types that require per-session parameters (e.g. `login` with a unique `k1` challenge), the service uses `nprofile` addressing. The wallet must perform a [NIP-17](https://github.com/nostr-protocol/nips/blob/master/17.md) DM exchange to obtain parameters.

The `nprofile` encodes a pubkey + relay hints. It is assumed that the public key is dedicated to this LNURL endpoint.

## Flows

### Direct Fetch (`naddr`)

1. The wallet scans/loads the LNURL and parses the `naddr`.
2. The wallet connects to a relay from the `naddr` relay hints.
3. The wallet sends a `REQ` for the parameterized replaceable event:
   ```json
   {"kinds": [31120], "authors": ["<pubkey>"], "#d": ["payRequest"]}
   ```
4. The relay returns the event. The wallet parses `content` as LNURL parameters.
5. The wallet constructs the callback query string as usual. For example: `amount=1010&comment=whatever`
6. The wallet creates a fresh ephemeral Nostr key and sends a [NIP-17](https://github.com/nostr-protocol/nips/blob/master/17.md) Gift Wrapped DM (Kind 14 inside Seal inside Gift Wrap) to the service pubkey, with the query string as content.
7. The wallet subscribes to Kind 1059 (Gift Wrap) events addressed to its ephemeral pubkey:
   ```json
   {"kinds": [1059], "#p": ["<wallet-ephemeral-pubkey>"]}
   ```
8. The service unwraps the Gift Wrap, processes the request, and sends the LNURL response JSON back as a NIP-17 DM to the wallet's ephemeral key.
9. The wallet unwraps the response and handles it per the relevant LUD specification.

### Interactive Flow (`nprofile`)

1. The wallet scans/loads the LNURL and parses the `nprofile`.
2. The wallet connects to a relay from the `nprofile` relay hints.
3. The wallet creates a fresh ephemeral Nostr key and sends a NIP-17 DM to the service pubkey with empty content (or the LNURL `tag` if known from the encoding, e.g. `tag=login`).
4. The wallet subscribes to Kind 1059 (Gift Wrap) events addressed to its ephemeral pubkey:
   ```json
   {"kinds": [1059], "#p": ["<wallet-ephemeral-pubkey>"]}
   ```
5. The service unwraps the request, generates session-specific parameters (e.g. a `k1` challenge), and sends them back as a NIP-17 DM.
6. The wallet unwraps the response and reads the LNURL parameters.
7. Subsequent callback steps follow the same NIP-17 exchange as steps 5–9 in the Direct Fetch flow.

## The Callback Property

The `callback` property is **optional** in LNURL-over-Nostr responses. If omitted, the wallet sends callback requests to the service pubkey from the original `naddr` or `nprofile`.

If specified, `callback` SHOULD be a `nostr:nprofile1...` URI. This allows the service to decouple the advertised LNURL identity from the callback identity, reducing correlation of usage across different payers.

## LUD-17 Scheme Parsing

When a [LUD-17](17.md) scheme URI has a host starting with `naddr1` or `nprofile1`, the wallet MUST treat it as a `nostr:` URI rather than converting to `https:`. For example:

```
lnurlp:naddr1qvzqqqr4gupzp...  →  nostr:naddr1qvzqqqr4gupzp...
```

## Asynchronous Delivery

Nostr is asynchronous by nature. NIP-17 Gift Wrap events are stored by relays, enabling fully asynchronous LNURL flows:

- With `naddr`, the wallet can fetch parameters at any time — the relay has the event cached regardless of the service's availability.
- A wallet can send a callback request even if the service is offline. The service processes it when it comes back online.
- Services SHOULD periodically query their relays for unprocessed Gift Wrap events addressed to their LNURL pubkey.

## Error Handling

- If the service cannot process a request, it SHOULD reply with a standard LNURL error response: `{"status": "ERROR", "reason": "..."}`.
- If no response is received within a reasonable timeout (RECOMMENDED: 30 seconds for interactive flows), the wallet SHOULD inform the user and MAY retry.
- If a relay from the hints is unreachable, the wallet SHOULD try other relays listed in the `naddr`/`nprofile`.

## Security Considerations

- **Key isolation**: The Nostr key used for an LNURL endpoint SHOULD be dedicated to that purpose — not the user's main Nostr identity.
- **Ephemeral wallet keys**: The wallet MUST generate a fresh key for each LNURL interaction to prevent correlation across uses.
- **Replay protection**: Services SHOULD reject requests with `created_at` timestamps more than 10 minutes in the past.
- **NIP-17 privacy**: Gift Wrap (NIP-59) hides the sender's identity from relays via an ephemeral key on the outer wrap and randomizes timestamps. Only the recipient can unwrap the message.
