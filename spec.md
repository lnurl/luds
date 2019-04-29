`lnurl` is a bech32-encoded HTTPS query string which is supposed to help payer interact with payee and thus simplify a number of standard scenarios such as requesting incoming channels, withdrawing funds, logging in etc.

An example `lnurl`: 
> https://service.com/api?q=3fc3645b439ce8e7f2553a69e5267081d96dcd340693afabe04be7b0ccd178df

would be bech32-encoded as:
> LNURL1DP68GURN8GHJ7UM9WFMXJCM99E3K7MF0V9CXJ0M385EKVCENXC6R2C35XVUKXEFCV5MKVV34X5EKZD3EV56NYD3HXQURZEPEXEJXXEPNXSCRVWFNV9NXZCN9XQ6XYEFHVGCXXCMYXYMNSERXFQ5FNS

and presented as the following QR:

![A QR encoded LNUrl example](https://i.imgur.com/HbB7U1K.png)

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
	tag: "channelRequest" // Now user software knows what to do next...
}
```
4. Opens a Lightning socket connection to a target node using `uri` field.
5. Issues an HTTPS GET request using `<callback>?k1=<k1>&remoteid=<Local LN node ID>&private=<1/0>`
6. Receives a `{"status":"OK"}` / `{"status":"ERROR", "reason":"error details..."}` Json response.
7. Awaits for incoming `OpenChannel` message via Lightning socket connection which would initiate a channel opening.


## 2. Log in with Bitcoin Wallet
A special `linkingKey` can be used to login user to a service or authorise sensitive actions. This preferrably should be done without compromising user identity so plain LN node key can not be used here. Instead of asking for user credentials a service could display a "login" QR code which contains a specialized `lnurl`.

Once "login" QR code is scanned `linkingKey` derivation in user's wallet happens as follows:
1. There exists a private `hashingKey` which is derived by user wallet using `m/138'/0` path.
2. Service domain name is extracted from login `lnurl` and then hashed using `hmacSha256(hashingKey, service domain name)`.
3. First 8 bytes are taken from resulting hash and then turned into a `Long` which is in turn used to derive a service-specific `linkingKey` using `m/138'/0/<long value>` path, a Scala example:

```Scala
import scala.math.BigInt
import fr.acinq.bitcoin.DeterministicWallet._

val hashingPrivKey = derivePrivateKey(walletMasterKey, hardened(138L) :: 0L :: Nil)

val prefix = hmacSha256(hashingPrivKey, serviceDomainName).take(8)

val linkingPrivKey = derivePrivateKey(walletMasterKey, hardened(138L) :: 0L :: BigInt(prefix).toLong :: Nil)

val linkingKey = linkingPrivKey.publicKey
```

User software:
1. Scans a QR code and decodes a query string which must contain the following query parameters:
	- `tag` with value set to `login` which means no HTTPS GET should be made yet.
	- `k1` (challenge) which is going to be signed by user's `linkingPrivKey`.
2. Displays a "Login" dialog which must include a domain name extracted from `lnurl` query string.
3. Once accepted user software issues an HTTPS GET request using `<lnurl>?k1=<k1>&sig=<hex(sign(k1.toByteArray, linkingPrivKey))>&key=<hex(linkingKey)>` which results in a successful login once signature is verified by service. `linkingKey` should be used as user identifier in this case.


## 3. Withdrawing funds from a service
Today users are asked to provide a withdrawal Lightning invoice to a service, this requires some effort and is especially painful when user tries to withdraw funds into mobile wallet while using a desktop website. Instead of asking for Lightning invoice a service could display a "withdraw" QR code which contains a specialized `lnurl`.

User software:
1. Scans a QR code and decodes a query string.
2. Makes an HTTPS GET request to a service.
3. Gets Json response of form: 
```
{
	callback: String, // a second-level url which would accept a withdrawal Lightning invoice as query parameter
	k1: String, // (challenge) which is going to be signed by user's `linkingKey`
	maxWithdrawable: MilliSatoshi, // max withdrawable amount for a given user on a given service
	defaultDescription: String, // A default withdrawal invoice description
	tag: "withdrawRequest" // Now user software knows what to do next...
}
```
4. Displays a withdraw dialog where user can specify an exact sum to be withdrawn which would be bounded by: 
```
min(max amount withdrawable from service, local estimation of how much can be routed into wallet)
```
5. Once accepted user software issues an HTTPS GET request using `<callback>?k1=<k1>&sig=<hex(sign(k1.toByteArray, linkingPrivKey))>&pr=<Lightning invoice with user defined amount>`
6. Receives a `{"status":"OK"}` / `{"status":"ERROR", "reason":"error details..."}` Json response.
7. Awaits for incoming payment if response was successful.

Note that in this case only `sig` is present in withdrawal request while `linkingKey` itself is not included. It is assumed that user has already been logged into a service prior to issuing a withdrawal request so related `linkingKey` can be obtained by service internally. In case if a given service does not support login then `sig` should just be ignored by service and withdrawal should be sent to whoever can provide a valid `k1` secret.
