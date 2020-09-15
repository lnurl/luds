# LNURL-auth-v2

## Authorization with Bitcoin Wallet

This auth method uses a domain-specific `linkingKey` derived by `LN WALLET` to login user to a `SERVICE` or authorise any other kind of sensitive action.

### Wallet to service interaction flow:

1. `SERVICE` provides an auth `LNURL` string (in a form of bech32-encoded QR code, NFC or any other type of transport it sees fit).
2. `LN WALLET` decodes and parses an obtained `LNURL` which must contain the following query parameters:
    - `tag` with value set to `login` which means no `GET` should be made by `LN WALLET` yet.
    - `k1` (hex encoded 32 bytes of challenge) which is going to be hashed and signed by `LN WALLET`-derived `linkingPrivKey`.
    - `action` with value set to human readable description of action to be performed (e.g. "login to site" or "unlock the door").
    - `features` with value set to `v2`. By doing this `SERVICE` indicates that `LNURL-auth-v2` scheme is supported. `features` may contain many comma-separated named features (e.g. `v2,v3,...`) so `LN WALLET` must parse it accordingly.
3. `LN WALLET` displays a "Login" dialog which must include a domain name extracted from `LNURL` query string and `action` text.
4. Once accepted, user's `LN WALLET` creates a DER-encoded signature to use in GET response. Signature is obtained by signing `sha256(hexToBytes(k1) + utf8ToBytes(action))` on `secp256k1` using `linkingPrivKey`.
5. `LN WALLET` issues a GET to `SERVICE` using `<LNURL_hostname_and_path>?<LNURL_existing_query_parameters>&sig=<hex(DER_encoded_signature)>&key=<hex(linkingPubKey)>&usedfeatures=v2`. Adding `usedfeatures=v2` to response is an indication that `LN WALLET` also supports `v2` and `SERVICE` must verify a provided signature accordingly.
6. `SERVICE` responds with the following Json once client signature is verified: 
    ```
    {
        status: "OK"
    }
    ```
    or
    
    ```
    {"status":"ERROR", "reason":"error details..."}
    ```

### Server-side signature verification:

Once `SERVICE` receives a call at the specified `LNURL-auth` handler, it must extract a `sig`, `key`, `k1` and `action` query parameters, DER-decode a signature and verify it on `sha256(hexToBytes(k1) + utf8ToBytes(action))` data using `secp256k1` and user-provided `key`. If verification passes then `SERVICE` can use and store a provided `key` as user identifier.

`SERVICE` must make sure that:
 - `k1` values are randomly generated per each auth attempt, they can not be predictable or static.
 - Unexpected `k1`s are not accepted: it is advised for `SERVICE` to have a cache of unused `k1`s, only proceed with verification for `k1`s present in that cache and remove used `k1`s on successful auth attempts.

## `linkingKey` derivation

`LNURL-auth` works by deriving domain-specific `linkingKey`s from user seed. This approach has two goals: first one is simplicity (user only needs to keep mnemonic to preserve both funds and identity), second one is portability (user should be able to switch a wallet by entering the same mnemonic and get the same identity).

However, second goal is not reachable in practice because there exist different formats of seeds which can't be transferred across all existing wallets. As such a practical approach is to have a recommended ways to derive `linkingKey` for wallet types depending on a seed format they use.

### `linkingKey` derivation for BIP-32 based wallets:

1. There exists a private `hashingKey` which is derived by user `LN WALLET` using `m/138'/0` path.
2. `SERVICE` domain name is extracted from auth `LNURL` and then hashed using `hmacSha256(hashingKey, service domain name)`.
3. First 16 bytes are taken from resulting hash and then turned into a sequence of 4 `Long` values which are in turn used to derive a service-specific `linkingKey` using `m/138'/<long1>/<long2>/<long3>/<long4>` path, a Scala example:

```Scala
import fr.acinq.bitcoin.crypto
import fr.acinq.bitcoin.Protocol
import java.io.ByteArrayInputStream
import fr.acinq.bitcoin.DeterministicWallet._

val domainName = "site.com"
val hashingPrivKey = derivePrivateKey(walletMasterKey, hardened(138L) :: 0L :: Nil)
val derivationMaterial = hmac256(key = hashingPrivKey.toBin, message = domainName)
val stream = new ByteArrayInputStream(derivationMaterial.slice(0, 16).toArray)
val pathSuffix = Vector.fill(4)(Protocol.uint32(stream, ByteOrder.BIG_ENDIAN)) // each uint32 call consumes next 4 bytes
val linkingPrivKey = derivePrivateKey(walletMasterKey, hardened(138L) +: pathSuffix)
val linkingPubKey = linkingPrivKey.publicKey
```

### `linkingKey` derivation for wallets which don't have an access to master `privKey`:

In this case neither `hashingKey` nor domain-specific `linkingKey`s can be derived by path. To overcome this limitation a different scheme is used for this class of wallets:

1. The following canonical phrase is defined: `DO NOT EVER SIGN THIS TEXT WITH YOUR PRIVATE KEYS! IS IT ONLY USED FOR DERIVATION OF LNURL-AUTH HASHING-KEY, DISCLOSING ITS SIGNATURE WILL COMPROMISE YOUR LNURL-AUTH IDENTITY AND MAY LEAD TO LOSS OF FUNDS!`.
2. `LN WALLET` obtains an `RFC6979` deterministic signature of `sha256(utf8ToBytes(canonical phrase))` using `secp256k1` with node private key.
3. `LN WALLET` defines `hashingKey` as `PrivateKey(sha256(obtained signature))`.
4. `SERVICE` domain name is extracted from auth `LNURL` and then service-specific `linkingPrivKey` is defined as `PrivateKey(hmacSha256(hashingKey, service domain name))`.

`LN WALLET` must make sure it is not possible to accidentally or automatically sign and hand out a signature of canonical phrase.