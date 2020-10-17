# LNURL-auth

## Authorization with Bitcoin Wallet

A special `linkingKey` can be used to login user to a service or authorise sensitive actions. This preferrably should be done without compromising user identity so plain LN node key can not be used here. Instead of asking for user credentials a service could display a "login" QR code which contains a specialized `LNURL`.

### Server-side signature verification:

Once `LN SERVICE` receives a call at the specified `LNURL-auth` handler, it should take `k1`, `key` and a DER-encoded `sig` and verify the signature using `secp256k1`, storing somehow `key` as the user identifier, either in a session, database or however it sees fit.

`LN SERVICE` must make sure that:
 - `k1` values are randomly generated per each auth attempt, they can not be predictable or static.
 - Unexpected `k1`s are not accepted: it is advised for `LN SERVICE` to have a cache of unused `k1`s, only proceed with verification for `k1`s present in that cache and remove used `k1`s on successful auth attempts.

 `LN SERVICE` should carefully choose which subdomain (if any) will be used as LNURL-auth endpoint and stick to chosen subdomain in future. For example, if `auth.site.com` was initially chosen then changing it to, say, `login.site.com` will result in different account for each user because full domain name is used by wallets as material for key derivation.

 `LN SERVICE` should consider giving meaningful names to chosen subdomains since `LN WALLET` may show a full domain name to users on login attempt. For example, `auth.site.com` is less confusing than `ksf03.site.com`.

### Key derivation for Bitcoin wallets:

Once "login" QR code is scanned `linkingKey` derivation in user's `LN WALLET` should happen as follows:
1. There exists a private `hashingKey` which is derived by user `LN WALLET` using `m/138'/0` path.
2. `LN SERVICE` full domain name is extracted from login `LNURL` and then hashed using `hmacSha256(hashingKey, full service domain name)`. Full domain name here means FQDN with last comma omitted (Example: for `https://x.y.z.com/...` it would be `x.y.z.com`).
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
val linkingKey = linkingPrivKey.publicKey
```

`LN WALLET` may choose to use a different derivation scheme but doing so will make it unportable. That is, users won't be able to switch to a different wallet and keep using a service bound to existing `linkingKey`.

### Wallet to service interaction flow:

1. `LN WALLET` scans a QR code and decodes an URL which must contain the following query parameters:
	- `tag` with value set to `login` which means no GET should be made yet.
	- `k1` (hex encoded 32 bytes of challenge) which is going to be signed by user's `linkingPrivKey`.
2. `LN WALLET` displays a "Login" dialog which must include a domain name extracted from `LNURL` query string.
3. Once accepted, user `LN WALLET` signs `k1` on `secp256k1` using `linkingPrivKey` and DER-encodes the signature. `LN WALLET` Then issues a GET to `LN SERVICE` using `<LNURL_hostname_and_path>?<LNURL_existing_query_parameters>&sig=<hex(sign(k1.toByteArray, linkingPrivKey))>&key=<hex(linkingKey)>` 
4. `LN SERVICE` responds with the following Json once client signature is verified: 
    ```
    {
        status: "OK", 
        event: "REGISTERED | LOGGEDIN | LINKED | AUTHED" // An optional enum indication of which exact action has happened, 3 listed types are supported
    }
    ```
    or
    
    ```
    {"status":"ERROR", "reason":"error details..."}
    ```
    
`linkingKey` should henceforth be used as user identifier by service. 

`event` enums meaning:
- `REGISTERED`: service has created a new account linked to user provided `linkingKey`.
- `LOGGEDIN`: service has found a matching existing account linked to user provided `linkingKey`.
- `LINKED` service has linked a user provided `linkingKey` to user's existing account (if account was not originally created using `lnurl-auth`).
- `AUTHED`: user was requesting some stateless action which does not require logging in (or possibly even prior registration) and that request was granted.
