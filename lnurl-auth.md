# LNURL-auth


## Authorization with Bitcoin Wallet

A special `linkingKey` can be used to login user to a service or authorise sensitive actions. This preferrably should be done without compromising user identity so plain LN node key can not be used here. Instead of asking for user credentials a service could display a "login" QR code which contains a specialized `LNURL`.


### Server-side generation of auth URL and signature verification:

When creating an `LNURL-auth` handler `LN SERVICE` must include in it a `k1` query parameter which in its simplest form can be a randomly generated 32 bytes of data, an example is `https://site.com?tag=login&k1=hex(32 bytes of random data)`.

`LN SERVICE` may also go further and add the following three query parameters: `https://site.com?tag=login&raw=<hex(32 bytes of random data)>&action=<base64(short human-readable description of action to be performed)>&k1=<hex(sha256(utf8ToBytes(raw + action))>`. By doing so `LN SERVICE` informs user which exact action is about to be performed and makes user commit to that action when signing `k1`.

Later, once `LN SERVICE` receives a call at the specified `LNURL-auth` handler, it should take `k1`, `key` and a DER-encoded `sig` and verify the signature using `secp256k1`. Once signature is successfully verified a user provided `key` can be used as identifier and may be stored in a session, database or however `LN SERVICE` sees fit.

`LN SERVICE` must make sure that unexpected `k1`s are not accepted: it is strongly advised for `LN SERVICE` to have a cache of unused `k1`s, only proceed with verification of `k1`s present in that cache and remove used `k1`s on successful auth attempts.


### Wallet to service interaction flow:

1. `LN WALLET` scans a QR code and decodes an URL which is expected to have the following query parameters:
    - `tag` with value set to `login` which means no GET should be made yet.
    - `k1` (hex encoded 32 bytes of challenge) which is going to be signed by user's `linkingPrivKey`.
    - optional `raw` (hex encoded 32 bytes of random data).
    - optional `action` (a base64 encoded human-readable action to be performed e.g "login to site" or "unlock the door").

2. If both `raw` and `action` query parameters are present then wallet must make sure that `k1` equals `hex(sha256(utf8ToBytes(raw + action)))`.

2. `LN WALLET` displays a "Login" dialog which must include a domain name extracted from `LNURL` query string and decoded `action` if both `raw` and `action` were present in URL.

3. Once accepted by user, `LN WALLET` signs `k1` on `secp256k1` using `linkingPrivKey` and DER-encodes the signature. `LN WALLET` Then issues a GET to `LN SERVICE` using `<LNURL_hostname_and_path>?<LNURL_existing_query_parameters>&sig=<hex(sign(utf8ToBytes(k1), linkingPrivKey))>&key=<hex(linkingKey)>` 
4. `LN SERVICE` responds with the following Json once client signature is verified: 
    ```
    {
        status: "OK"
    }
    ```
    or
    
    ```
    {"status":"ERROR", "reason":"error details..."}
    ```


## `linkingKey` derivation

`LNURL-auth` works by deriving domain-specific `linkingKey`s from user seed. This approach has two goals: first one is simplicity (user only needs to keep mnemonic to preserve both funds and identity), second one is portability (user should be able to switch a wallet by entering the same mnemonic and get the same identity).

However, second goal is not reachable in practice because there exist different formats of seeds which can't be transferred across all existing wallets. As such a practical approach is to have a recommended ways to derive `linkingKey` for different wallet types.


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

1. The following canonical phrase is defined: `DO NOT EVER SIGN THIS TEXT WITH YOUR PRIVATE KEYS! IT IS ONLY USED FOR DERIVATION OF LNURL-AUTH HASHING-KEY, DISCLOSING ITS SIGNATURE WILL COMPROMISE YOUR LNURL-AUTH IDENTITY AND MAY LEAD TO LOSS OF FUNDS!`.
2. `LN WALLET` obtains an `RFC6979` deterministic signature of `sha256(utf8ToBytes(canonical phrase))` using `secp256k1` with node private key.
3. `LN WALLET` defines `hashingKey` as `PrivateKey(sha256(obtained signature))`.
4. `SERVICE` domain name is extracted from auth `LNURL` and then service-specific `linkingPrivKey` is defined as `PrivateKey(hmacSha256(hashingKey, service domain name))`.

`LN WALLET` must make sure it is not possible to accidentally or automatically sign and hand out a signature of canonical phrase.