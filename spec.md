`lnurl` is a bech32-encoded HTTPS query string which is supposed to help payer interact with payee and thus simplify a number of standard scenarios such as requesting incoming channels, withdrawing funds, doing atomic swaps etc.

This example `lnurl`: 
> https://service.com/api?q=3fc3645b439ce8e7f2553a69e5267081d96dcd340693afabe04be7b0ccd178df

would be bech32-encoded as:

> LNURL1DP68GURN8GHJ7UM9WFMXJCM99E3K7MF0V9CXJ0M385EKVCENXC6R2C35XVUKXEFCV5MKVV34X5EKZD3EV56NYD3HXQURZEPEXEJXXEPNXSCRVWFNV9NXZCN9XQ6XYEFHVGCXXCMYXYMNSERXFQ5FNS

A QR encoded form:

![A QR encoded LNUrl example](https://i.imgur.com/HbB7U1K.png)

`lnurl` can be presented either directly or be embedded in Lightning invoice if respected `lnurl` usage scenario can be gracefully degraded to just using an invoice. When embedded in Lightning invoice `lnurl` is not bech32-encoded and a letter `v` is used as tag identifier with decoding rules identical to Description tag `d`. When presented directly a `lightning:` prefix may be used similar to how it's currently used with Lightning invoices.

Once `lnurl` is decoded:
- If `tag` query parameter is present then this `lnurl` has a special meaning, further actions will be based on `tag` parameter value.
- Otherwise an HTTPS GET request should be issued which must return a Json object containing a `tag` field, further actions will be based on `tag` field value.


# Example usage scenarios

## 1. Incoming payment channel request  
Suppose user has a balance on a certain service which he wishes to turn into an incoming channel and service supports such functionality. This would require many parameters so resulting QR may be overly dense and cause scanning issues. Other than that, when using a mobile wallet user has to make sure that a connection to target LN node is established from mobile client before an incoming channel is requested.

User software:
1. Scans a QR code and decodes a query string.
2. Makes an HTTPS GET request to a service.
3. Gets Json response of form: 
```
{
	uri: String, // Remote node address of form node_key@ip_address:port_number
	callback: String, // a second-level url which would initiate an OpenChannel message from target LN node
	k1: String, // a second-level secret to authorize user request 
	capacity: Long,
	push: Long, 
	cltvExpiryDelta: Int, 
	htlcMinimumMsat: Long, 
	feeBaseMsat: Long, 
	feeProportionalMillionths: Long,
	tag: "channelRequest" // Now user software knows what to do next...
}
```
4. Opens a Lightning socket connection to a target node using `uri` field.
5. Issues an HTTPS GET request using `<callback>?k1=<k1>&remoteid=<Local LN node ID>&private=<1/0>`
6. Receives a `{"status":"OK"}` / `{"status":"ERROR", "reason":"error details..."}` Json response.
7. Awaits for incoming `OpenChannel` message via Lightning socket connection which would initiate a channel opening.


## 2. Withdrawing funds from a service
Today users are asked to provide a withdrawal Lightning invoice to a service, this requires some effort and is especially painful when user tries to withdraw funds into mobile wallet using a website on a desktop. Instead of asking for Lightning invoice a service could display a "withdraw" QR code which contains a specialized `lnurl`.

User software:
1. Scans a QR code and decodes a query string.
2. Makes an HTTPS GET request to a service.
3. Gets Json response of form: 
```
{
	callback: String, // a second-level url which would accept a withdrawal Lightning invoice as query parameter
	k1: String, // a second-level secret to authorize user request 
	maxWithdrawable: MilliSatoshi, // max withdrawable amount for a given user on a given service
	defaultDescription: String, // A default withdrawal invoice description
	tag: "withdrawRequest" // Now user software knows what to do next...
}
```
4. Displays a withdraw dialog where user can specify an exact sum to be withdrawn which would be bounded by: 
```
min(max amount withdrawable from service, local estimation of how much can be routed into wallet)
```
5. Issues an HTTPS GET request using `<callback>?k1=<k1>&pr=<Lightning invoice with user defined amount>`
6. Receives a `{"status":"OK"}` / `{"status":"ERROR", "reason":"error details..."}` Json response.
7. Awaits for incoming payment if response was successful.

Security note: withdrawal `lnurl` should be ephemeral and re-generated from scratch for each new withdrawal request.

An example server-side code which processes such withdrawal requests: https://gist.github.com/whiteyhat/85048e46db618c697e1e9a9f8b49426b


## 3. Linkable payments
What exists currently is a Proof-of-Payment which is a payment preimage but no such thing as Proof-of-Payer. The following scheme may be used by a service to link multiple payments as belonging to a single payer without compromising payer's identity. Related `lnurl` must be embedded in a Lightning invoice.

Linking is achieved by payer providing a `linkingKey` ahead of payment. Key derivation is as follows:
1. There exists a private `hashingKey` which is derived by user wallet using `m/138'/0` path.
2. Service domain name is extracted from `lnurl` and then hashed using `hmacSha256(hashingKey, service domain name)`.
3. First 8 bytes are taken from resulting hash and then turned into a `Long` which is in turn used to derive a service-specific `linkingKey` using `m/138'/0/<long value>` path, a Scala example:
```Scala
import scala.math.BigInt
import fr.acinq.bitcoin.DeterministicWallet._

val prefix = hmacSha256(hashingKey, serviceDomainName).take(8)
val linkingPrivKey = derivePrivateKey(walletMasterKey, hardened(138L) :: 0L :: BigInt(prefix).toLong :: Nil)
val linkingKey = linkingPrivKey.publicKey
```

User software:
1. Scans a QR code and extracts `lnurl` from payment request: it must contain a `tag` query parameter with value set to `link` which means no HTTPS GET should be made yet.
2. Displays a "Linkable payment" dialog which must include the following additional elements:
	- Domain name extracted from `lnurl` query string.
	- An ability to opt out into usual payment. This entails that service which offers linkable payments must be prepared for some users opting out, there has to be a way to still handle such situations correctly.
3. Once accepted user software issues an HTTPS GET request using `<lnurl>&key=<hex(linkingKey)>`
4. Receives a `{"status":"OK"}` / `{"status":"ERROR", "reason":"error details..."}` Json response.
5. Fulfills a scanned Lightning invoice.


## 4. Log in with Bitcoin Wallet
`linkingKey` described in a previous use case can also be used to login user to a service or authorise sensitive actions (such as withdrawal) by signing a challenge.

User software:
1. Scans a QR code and decodes a query string which must contain the following query parameters:
	- `tag` with value set to `login` which means no HTTPS GET should be made yet.
	- `c` (challenge) with value set to random 32 bytes of hex encoded data which is going to be signed by user's `linkingKey`.
2. Displays a "Login" dialog which must include a domain name extracted from `lnurl` query string.
3. Once accepted user software issues an HTTPS GET request using `<lnurl>&key=<hex(linkingKey)>&sig=<hex(sign(<challenge>, linkingPrivKey))>` which results in a successful service login.

This example login `lnurl`: 
> https://service.com?tag=login&c=158ea41f653d6447b7f8ca980363985d0a9a5b72d41bb8b7f5119c7c308a372c

would be bech32-encoded as:

> LNURL1DP68GURN8GHJ7UM9WFMXJCM99E3K7MFLW3SKW0TVDANKJM3XVV7NZDFCV4SNGVTXXC6NXEPKXS6RWC3HVCUXXCFE8QCRXD3N8YUR2EPSVYUKZDTZXUEXGDP3VF3RSC3HVC6NZVFEVVMKXVES8PSNXDEJVVDT40VT
