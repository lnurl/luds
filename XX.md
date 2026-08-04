LUD-XX: Ark address in payRequest response.
=============================================

author: Codepocketdev | discussion: TBD

---

## Motivation

Wallets that support the Ark protocol can route payments via
arkoor (out-of-round) transactions, achieving zero-fee instant
settlement between users on the same Ark server. Currently,
there is no standard way for an LNURL-pay service to advertise
an Ark address alongside a Lightning address, forcing all
payments through the Lightning gateway even when a zero-fee
Ark-native path is available.

## Specification

LN SERVICE MAY include two optional fields in the payRequest
response defined in LUD-06:

  callback: string
  maxSendable: number
  minSendable: number
  metadata: string
  tag: payRequest
  ark: string               (optional: bech32 Ark address)
  ark_server_pubkey: string (optional: hex-encoded Ark server pubkey)

## Field definitions

ark: A bech32-encoded Ark address belonging to the payment
recipient on their Ark server. The address encodes a truncated
version of the server pubkey via its ArkId.

ark_server_pubkey: The full hex-encoded pubkey of the Ark
server the recipient is connected to. Included for maximum
accuracy and to catch faulty data. If present, wallets SHOULD
verify it matches the pubkey encoded in the ark address.

## Wallet behavior

If the ark field is present in the payRequest response,
LN WALLET SHOULD attempt Ark-native routing before falling
back to Lightning:

1. Parse the ark field as an Ark address. If parsing fails,
   ignore and proceed with Lightning.
2. Check if the address belongs to the wallets connected Ark
   server: address.ark_id().is_for_server(server_pubkey)
3. If same server: send via arkoor (zero fee, instant)
4. If different server: fall back to Lightning payment as
   defined in LUD-06

If ark_server_pubkey is present, LN WALLET SHOULD additionally
verify that it matches the server pubkey encoded in the ark
address to detect faulty or mismatched data before attempting
payment.

## Reference implementation

The Ark address server check is implemented in bark-wallet:

  let ark_info = wallet.ark_info().await?;
  if address.ark_id().is_for_server(ark_info.server_pubkey) {
      wallet.send_arkoor_payment(&address, amount).await?;
  } else {
      // fall back to Lightning
  }

## Backwards compatibility

Wallets that do not implement this extension MUST ignore the
ark and ark_server_pubkey fields and proceed with normal
Lightning payment as defined in LUD-06. Fully backwards compatible.

## Security considerations

- A malicious service could return a fraudulent ark field.
  Wallets MUST verify server compatibility before sending.
- If ark_server_pubkey is present, wallets SHOULD cross-check
  it against the pubkey encoded in the ark address.
- Wallets MUST NOT send via arkoor if server verification fails.
  Fall back to Lightning silently.
