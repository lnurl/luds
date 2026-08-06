LUD-XX: `paymentOptions` for `payRequest`.
===========================================

`author: andrerfneves` `discussion: https://github.com/lnurl/luds/pull/303`

---

This LUD extends [LUD-06](06.md) `payRequest` so a `SERVICE` can advertise multiple payment options for the same LNURL-pay flow, while preserving the existing Lightning/BOLT11 behavior as the default.

It also extends [LUD-21](21.md) verification responses so payment options that do not return a BOLT11 invoice can be verified using the same `verify` mechanism already used for Lightning invoices.

The goal is to let a Lightning Address or LNURL-pay endpoint say:

> This payment can be completed over Lightning/BOLT11, onchain Bitcoin, Liquid, Arkade, BOLT12, Spark, or another supported payment method, without adding another discovery endpoint and without changing existing wallets that only support LUD-06.

This LUD defines payment method selection only. It does not define non-BTC assets, currency conversion, quotes, per-option fees, surcharges, asset identifiers, asset decimals, or asset metadata. Those concerns can be handled by other LUDs, such as a currencies/quotes LUD or a future assets-related LUD. Non-BTC assets and asset identifiers are intentionally out of scope, but this LUD is designed to compose with future currencies/assets LUDs by keeping rail selection (`paymentOption`) separate from amount denomination and asset selection.

## Summary

A `SERVICE` MAY add `paymentOptions` to the initial LUD-06 response.

A `WALLET` that supports this LUD MAY choose one advertised option and pass its `id` to the callback using the `paymentOption` query parameter.

If `paymentOption` is absent, the `SERVICE` SHOULD behave as a normal LUD-06 service and return a BOLT11 invoice in `pr`.

For `type: "lightning"`, the callback response MUST remain backwards compatible with LUD-06 by including `pr`. It MAY also include the generic `paymentDestination` and `paymentURI` fields defined by this LUD.

For all non-`pr` payment options, the callback response returns `paymentDestination` and/or `paymentURI` instead of `pr`. A successful response MUST include at least one of `paymentDestination` or `paymentURI`, and MAY include `verify`.

The `verify` URL follows LUD-21 semantics. For non-`pr` payment options, the verify response MUST include `paymentOption`, SHOULD echo the `paymentDestination` or `paymentURI` issued in the callback response, and MAY include `paymentReference`.

## Initial response

A `SERVICE` MAY include a `paymentOptions` array in the initial `payRequest` response:

```diff
 {
     "callback": "https://service.com/lnurl/pay/callback",
     "maxSendable": 100000000,
     "minSendable": 1000,
     "metadata": "[[\"text/plain\",\"Pay alice@example.com\"]]",
-    "tag": "payRequest"
+    "tag": "payRequest",
+    "paymentOptions": [
+        {
+            "id": "lightning",
+            "type": "lightning"
+        },
+        {
+            "id": "onchain",
+            "type": "onchain"
+        },
+        {
+            "id": "liquid",
+            "type": "liquid"
+        },
+        {
+            "id": "arkade",
+            "type": "arkade"
+        },
+        {
+            "id": "bark",
+            "type": "bark"
+        },
+        {
+            "id": "bolt12",
+            "type": "bolt12"
+        },
+        {
+            "id": "spark",
+            "type": "spark"
+        },
+        ...
+        ...
+        ...
+    ]
 }
```

If `paymentOptions` is absent, `WALLET` behavior is unchanged from LUD-06.

A `WALLET` that does not understand `paymentOptions` MUST ignore it and continue the normal LUD-06 flow.

## `paymentOptions` fields

Each entry in `paymentOptions` is a payment option object.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | yes | Unique selectable option identifier. This is the value sent as `paymentOption` to the callback. |
| `type` | string | yes | Payment method type, e.g. `lightning`, `bolt12`, `onchain`, `liquid`, `arkade`, `spark`, or `bark`. |
| `available` | boolean | no | Whether this option is currently available. If absent, assume `true`. |
| `minSendable` | number | no | Option-specific minimum amount in millisatoshis. If absent, use top-level `minSendable`. |
| `maxSendable` | number | no | Option-specific maximum amount in millisatoshis. If absent, use top-level `maxSendable`. |

`minSendable` and `maxSendable` in a payment option are always denominated in millisatoshis, matching LUD-06. They narrow the LUD-06 amount range for that option. They do not define fiat, currency, or asset-denominated input limits.

A `SERVICE` MAY include additional fields. A `WALLET` MUST ignore unknown fields.

The `id` field MUST be unique within the `paymentOptions` array.

The `type` field identifies the payment method category. The `id` field identifies the exact selectable option.

For the simple case, `id` and `type` MAY have the same value:

```json
{
    "id": "liquid",
    "type": "liquid"
}
```

If a future LUD defines multiple selectable options for the same method, such as rail-specific assets, finality policies, providers, quotes, or fee policies, those options can use distinct `id` values while sharing the same `type`.

This LUD does not define those asset, quote, or policy fields.

## Initial `type` registry

This LUD defines the following initial `type` values:

| Type | Description |
|------|-------------|
| `lightning` | Existing LUD-06 BOLT11 Lightning payment. Callback response MUST include `pr`. It MAY also include `paymentDestination` and `paymentURI`. This is the default legacy option. |
| `onchain` | Bitcoin base-layer payment. Callback response may return a Bitcoin address and/or Bitcoin URI. |
| `liquid` | Liquid Network payment. Callback response may return a Liquid address and/or Liquid URI. |
| `arkade` | Arkade offchain payment. Callback response may return an Arkade address and/or Arkade-specific payment URI if one is defined. |
| `bark` | Bark payment. Callback response may return a Bark payment destination and/or Bark-specific payment URI. |
| `bolt12` | BOLT12 payment. Callback response may return a BOLT12 offer, invoice, or other BOLT12 payment instruction using `paymentDestination` and/or `paymentURI`. |
| `spark` | Spark payment. Callback response may return a Spark payment destination and/or Spark-specific payment URI. |

Future LUDs MAY define additional `type` values.

Services MAY advertise unknown `type` values. A `WALLET` MUST ignore unsupported payment options.

## Callback request

A `WALLET` that selects a payment option calls the LUD-06 callback with the `paymentOption` query parameter:

```diff
- <callback><?|&>amount=<milliSatoshi>
+ <callback><?|&>amount=<milliSatoshi>&paymentOption=<paymentOption.id>
```

Example:

```http
GET https://service.com/lnurl/pay/callback?amount=100000&paymentOption=liquid
```

The value of `paymentOption` MUST match the `id` of one entry in the `paymentOptions` array.

If `paymentOption` is absent, the `SERVICE` SHOULD use its default LUD-06 behavior. The default SHOULD be `lightning` when the service supports Lightning.

If `paymentOption` is present but unknown, unavailable, or incompatible with the requested amount, the `SERVICE` MUST return an error response:

```json
{
    "status": "ERROR",
    "reason": "Unsupported paymentOption"
}
```

## Amount semantics

This LUD does not redefine the meaning of the LUD-06 `amount` callback parameter.

Unless another LUD defines alternate denomination or quote semantics, `amount` remains denominated in millisatoshis.

A `SERVICE` MUST treat the selected `paymentOption` as a way to satisfy the requested LUD-06 amount, not as a way to reinterpret the unit of `amount`.

For a non-`pr` payment option, the `SERVICE` MUST honor the requested `amount` exactly. All payment methods in scope of this LUD are BTC-denominated, so the unit conversion is fixed and exact: 1 satoshi = 1000 millisatoshis, and 1 BTC = 100,000,000,000 millisatoshis. When the method's URI form carries an amount, the `SERVICE` MUST encode the exact converted amount in `paymentURI`. Otherwise, the `SERVICE` MUST treat the exact converted amount as the expected payment amount.

If the requested `amount` is not exactly representable in the payment method's smallest unit (for example, a sub-satoshi `amount` for `onchain`), the `SERVICE` MUST return the error response defined in "Callback request" and MUST NOT round, truncate, or otherwise alter the amount.

A `WALLET` that executes a `paymentURI` carrying an amount MUST verify that the encoded amount, converted back to millisatoshis, equals the requested `amount` exactly. If the encoded amount is missing where the method requires one, malformed, or not exactly equal, the `WALLET` MUST NOT execute the payment without explicit user confirmation of the actual amount.

For payment methods whose callback response contains only `paymentDestination`, the `WALLET` MUST construct the payment for the requested `amount` converted exactly to the method's unit. If the requested amount is not exactly representable in that unit, the `WALLET` MUST NOT attempt the payment and SHOULD treat the payment option as unusable for that amount. In all cases, the `SERVICE` MUST NOT report `settled: true` at the `verify` URL for a payment whose received amount differs from the requested `amount`.

These amount requirements apply only to non-`pr` payment options. The Lightning/BOLT11 flow, in which `pr` binds the amount, is unchanged.

This LUD does not define per-option surcharges, fees, or alternate requested amounts. Services that require different economic amounts per payment option SHOULD compose this LUD with a currencies/quotes LUD or a future payment-option quote extension.

## Callback response for BOLT11 Lightning

For `type: "lightning"`, the callback response remains backwards compatible with LUD-06 and LUD-21.

A `SERVICE` MUST include `pr` for `type: "lightning"`:

```json
{
    "status": "OK",
    "paymentOption": "lightning",
    "routes": [],
    "pr": "lnbc10...",
    "verify": "https://service.com/verify/pay_123"
}
```

A `SERVICE` MAY also include the generic fields defined by this LUD:

```diff
{
    "status": "OK",
    "paymentOption": "lightning",
    "routes": [],
    "pr": "lnbc10...",
+   "paymentDestination": "lnbc10...",
+   "paymentURI": "lightning:lnbc10...",
    "verify": "https://service.com/verify/pay_123"
}
```

For `type: "lightning"`, `pr` is authoritative for backwards compatibility. If `paymentDestination` is present, it SHOULD match `pr`.

The `paymentOption` field is optional for legacy Lightning responses, but services supporting this LUD SHOULD include it when the wallet selected an explicit Lightning payment option.

A legacy LUD-06 response without `paymentOption` remains valid:

```json
{
    "routes": [],
    "pr": "lnbc10..."
}
```

## Callback response for non-`pr` payment options

For any other payment options that do not return `pr`, the callback response MAY include `paymentDestination`, `paymentURI`, and `verify`.

A successful non-`pr` callback response MUST include `paymentDestination`, `paymentURI`, or both.

```json
{
    "status": "OK",
    "paymentOption": "onchain",
    "paymentDestination": "bc1q...",
    "paymentURI": "bitcoin:bc1q...?amount=0.001",
    "verify": "https://service.com/verify/pay_123"
}
```

Fields:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `paymentOption` | string | yes | Selected payment option `id`. |
| `paymentDestination` | string | no | Destination issued by the service, such as an address, account, offer, invoice, or rail-specific payment target. Required if `paymentURI` is absent. |
| `paymentURI` | string | no | Full wallet-executable payment URI/instruction, if the method defines one. Required if `paymentDestination` is absent. |
| `verify` | string | no | LUD-21 verification URL for this payment. |

`paymentDestination` is the bare destination or target when the payment method has a meaningful standalone destination.

`paymentURI` is the executable wallet instruction. It may include method-specific data such as amount, memo, message, structured request data, or other information needed by a compatible wallet.

Examples:

### Onchain Bitcoin

```json
{
    "status": "OK",
    "paymentOption": "onchain",
    "paymentDestination": "bc1p5d7rj...",
    "paymentURI": "bitcoin:bc1p5d7rjq...?amount=0.001",
    "verify": "https://service.com/verify/pay_123"
}
```

### Liquid

```json
{
    "status": "OK",
    "paymentOption": "liquid",
    "paymentDestination": "lq1qq2ftgv...",
    "paymentURI": "liquidnetwork:lq1qq2ftgv...?amount=0.001",
    "verify": "https://service.com/verify/pay_123"
}
```

### Arkade

```json
{
    "status": "OK",
    "paymentOption": "arkade",
    "paymentDestination": "ark1qg5...",
    "verify": "https://service.com/verify/pay_123"
}
```

### Bark

```json
{
    "status": "OK",
    "paymentOption": "bark",
    "paymentDestination": "ark1qn8zq...",
    "verify": "https://service.com/verify/pay_123"
}
```

### BOLT12

```json
{
    "status": "OK",
    "paymentOption": "bolt12",
    "paymentDestination": "lno1dp3wqk...",
    "paymentURI": "lightning:lno1dp3wqk...",
    "verify": "https://service.com/verify/pay_123"
}
```

### Spark

```json
{
    "status": "OK",
    "paymentOption": "spark",
    "paymentDestination": "spark1pyul5...",
    "verify": "https://service.com/verify/pay_123"
}
```

### URI-only instruction

Some payment methods may represent the complete instruction as a URI or method-specific request, without a useful separate destination field.

```json
{
    "status": "OK",
    "paymentOption": "example-contract-call",
    "paymentURI": "paymethod:contract/...?...",
    "verify": "https://service.com/verify/pay_123"
}
```

## Verification

This LUD reuses the `verify` URL introduced by LUD-21.

For `type: "lightning"`, existing LUD-21 verification responses remain valid:

```json
{
    "status": "OK",
    "settled": false,
    "preimage": null,
    "pr": "lnbc10..."
}
```

```json
{
    "status": "OK",
    "settled": true,
    "preimage": "123456...",
    "pr": "lnbc10..."
}
```

For non-`pr` payment options, a `SERVICE` MAY return the following LUD-21-compatible extension:

```json
{
    "status": "OK",
    "settled": false,
    "paymentOption": "onchain",
    "paymentDestination": "bc1q...",
    "paymentReference": null
}
```

If the original callback response contained only `paymentURI`, a verify response MAY return `paymentURI` instead of `paymentDestination`:

```json
{
    "status": "OK",
    "settled": false,
    "paymentOption": "example-contract-call",
    "paymentURI": "paymethod:contract/...?...",
    "paymentReference": null
}
```

After the service observes a payment reference but does not yet consider the payment complete:

```json
{
    "status": "OK",
    "settled": false,
    "paymentOption": "onchain",
    "paymentDestination": "bc1p...",
    "paymentReference": "bitcoin_txid_here"
}
```

After the service considers the payment complete:

```json
{
    "status": "OK",
    "settled": true,
    "paymentOption": "onchain",
    "paymentDestination": "bc1p...",
    "paymentReference": "bitcoin_txid_here"
}
```

Fields:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `settled` | boolean | yes | Whether the service considers the payment complete according to its own policy. |
| `paymentOption` | string | yes for non-`pr` | Selected payment option `id`. |
| `paymentDestination` | string | no | Destination issued in the callback response. SHOULD be present if it was present in the callback response. |
| `paymentURI` | string | no | Payment URI/instruction issued in the callback response. SHOULD be present if it was present and `paymentDestination` was absent in the callback response. |
| `paymentReference` | string or null | no | Method-specific payment reference once known by the service. |

For transaction-based methods such as onchain Bitcoin and Liquid, `paymentReference` SHOULD be the transaction id.

For Arkade, `paymentReference` SHOULD be the Arkade transaction id or other canonical transaction reference exposed by Arkade. For Spark, `paymentReference` SHOULD be the canonical Spark transfer/payment reference. For Bark, `paymentReference` SHOULD be the canonical Bark transfer/payment reference if one is available.

The meaning of `paymentReference` is scoped by `paymentOption`.

If `settled` is `false` and `paymentReference` is absent or `null`, the service has not observed a payment reference yet. If `settled` is `false` and `paymentReference` is present, the service has observed a method-specific payment reference but does not yet consider the payment complete. If `settled` is `true`, the service considers the payment complete according to its own policy.

For payment methods with multiple finality levels, such as multiple-confs, preconfirmation, and later base-layer settlement, `settled` follows LUD-21 semantics and means the LNURL `SERVICE` considers the payment complete. It does not necessarily mean base-layer finality unless the service's policy requires base-layer finality.

## Lightning Address compatibility

[LUD-16](16.md) Lightning Address resolves to a normal LUD-06 `payRequest` response.

Therefore, no change to LUD-16 is required. A Lightning Address endpoint may include `paymentOptions` in the LUD-06 response returned from:

```http
GET https://domain.com/.well-known/lnurlp/name
```

Wallets that do not support this LUD will continue to use the default LUD-06 Lightning flow.

Wallets that support this LUD may choose a non-default payment option using the existing callback flow.

## Security considerations

A `WALLET` MUST display enough information for the user to understand which payment option is being used.

A `WALLET` MUST NOT silently switch from Lightning to another payment option without user awareness unless the user or wallet policy explicitly allows it.

If a selected payment option changes the total economic cost to the payer, that cost MUST be made aware to the user before they confirm payment.

A `SERVICE` MUST ensure that the `paymentDestination` and/or `paymentURI` returned by the callback is bound to the selected payment option, amount, metadata, and verification URL.

A `SERVICE` SHOULD issue unique destinations, URIs, or payment sessions when needed to correlate payments reliably.

A `SERVICE` MUST reject unknown or unavailable `paymentOption` values.

A `WALLET` MUST ignore unknown fields in `paymentOptions`, callback responses, and verify responses.

## Dependencies

`06 -> XX` — extends [LUD-06](06.md) `payRequest`.

`21 -> XX` — extends [LUD-21](21.md) verification for non-`pr` payment options.
