# LNURL: Lightning Network UX protocol   

`LNURL` is a bech32-encoded HTTPS query string which is supposed to help payer interact with payee and thus simplify a number of standard scenarios such as:

- Requesting incoming channels 
- Logging in
- Withdrawing funds
- Paying for a service

An example `LNURL`: 
> https://service.com/api?q=3fc3645b439ce8e7f2553a69e5267081d96dcd340693afabe04be7b0ccd178df

would be bech32-encoded as:
> LNURL1DP68GURN8GHJ7UM9WFMXJCM99E3K7MF0V9CXJ0M385EKVCENXC6R2C35XVUKXEFCV5MKVV34X5EKZD3EV56NYD3HXQURZEPEXEJXXEPNXSCRVWFNV9NXZCN9XQ6XYEFHVGCXXCMYXYMNSERXFQ5FNS

and presented as the following QR:

![A QR encoded LNURL example](https://i.imgur.com/HbB7U1K.png)

Once `LNURL` is decoded:
- If `tag` query parameter is present then this `LNURL` has a special meaning, further actions will be based on `tag` parameter value.
- Otherwise an HTTPS GET request should be issued which must return a Json object containing a `tag` field, further actions will be based on `tag` field value.

# Fallback scheme

`LNURL` can be used as fallback inside of other URI schemes, with the key 'lightning' and the value equal to the bech32-encoding, an example: `https://service.com/giftcard/redeem?id=123&lightning=LNURL1...`

# Decoding examples

In Scala:
```scala
import fr.acinq.bitcoin.Bech32
val bech32lnurl: String = "LNURL1DP68GURN8GHJ7UM9WFMXJCM99E3K7MF0V9CXJ0M385EKVCENXC6R2C35XVUKXEFCV5MKVV34X5EKZD3EV56NYD3HXQURZEPEXEJXXEPNXSCRVWFNV9NXZCN9XQ6XYEFHVGCXXCMYXYMNSERXFQ5FNS"
  
val (hrp, dataPart) = Bech32.decode(bech32lnurl)
  
val requestByteArray = Bech32.five2eight(dataPart)
  
new String(requestByteArray, "UTF-8") // https://service.com/api?q=3fc3645b439ce8e7f2553a69e5267081d96dcd340693afabe04be7b0ccd178df
```

# Getting help

If you have any questions about implementing LNURL as a wallet or service, join us in the [BLW Telegram](https://t.me/lightningwallet) and get help from the creators and other LNURL implementors.


# Sub protocols:

## 1. LNURL-channel
### Incoming payment channel request  

Suppose user has a balance on a certain service which he wishes to turn into an incoming channel and service supports such functionality. This would require many parameters so the resulting QR may be overly dense and cause scanning issues. Additionally, the user has to make sure that a connection to target LN node is established before an incoming channel is requested.

**Wallet to service interaction flow:**

1. User scans a LNURL QR code or accesses an `lightning:LNURL..` link with `LN WALLET` and `LN WALLET` decodes LNURL.
2. `LN WALLET` makes an HTTPS GET request to `LN SERVICE` using the decoded LNURL.
3. `LN WALLET` gets Json response from `LN SERVICE` of form:
	
	```
	{
		uri: String, // Remote node address of form node_key@ip_address:port_number
		callback: String, // a second-level URL which would initiate an OpenChannel message from target LN node
		k1: String, // random or non-random string to identify the user's LN WALLET when using the callback URL
		tag: "channelRequest" // type of LNURL
	}
	```
	or

	```
	{"status":"ERROR", "reason":"error details..."}
	```
	
4. `LN WALLET` opens a connection to the target node using `uri` field.
5. `LN WALLET` issues an HTTPS GET request to `LN SERVICE` using `<callback>?k1=<k1>&remoteid=<Local LN node ID>&private=<1/0>`
6. `LN SERVICE` sends a `{"status":"OK"}` or `{"status":"ERROR", "reason":"error details..."}` Json response.
7. `LN WALLET` awaits for incoming `OpenChannel` message from the target node which would initiate a channel opening.

## 1.1 LNURL-hosted-channel
### Hosted channel request

**Wallet to service interaction flow:**

1. User scans a LNURL QR code or accesses an `lightning:LNURL..` link with `LN WALLET` and `LN WALLET` decodes LNURL.
2. `LN WALLET` makes an HTTPS GET request to `LN SERVICE` using the decoded LNURL.
3. `LN WALLET` gets Json response from `LN SERVICE` of form:
    
    ```
    {
    	uri: String, // Remote node address of form node_key@ip_address:port_number
    	k1: String, // a second-level hex encoded secret byte array to be used by wallet in `InvokeHostedChannel` message, may be random if Host has no use for it
    	alias: String, // Optional remote node alias
    	tag: "hostedChannelRequest" // type of LNURL
    }
    ```
    or
    
    ```
    {"status":"ERROR", "reason":"error details..."}
    ```
4. `LN WALLET` opens a connection to the target node using `uri` field.
5. Once connected, `LN WALLET` sends an `InvokeHostedChannel` message to the target node using `k1` converted to byte array.
6. The rest is handled by hosted channel protocol.


## 2. LNURL-auth
### Authorization with Bitcoin Wallet

A special `linkingKey` can be used to login user to a service or authorise sensitive actions. This preferrably should be done without compromising user identity so plain LN node key can not be used here. Instead of asking for user credentials a service could display a "login" QR code which contains a specialized `LNURL`.

**Server-side signature verification:**
Once `LN SERVICE` receives a call at the specified `LNURL-auth` handler, it should take `k1`, `key` and a DER-encoded `sig` and verify the signature using `secp256k1`, storing somehow `key` as the user identifier, either in a session, database or however it sees fit.

**Key derivation for Bitcoin wallets:**
Once "login" QR code is scanned `linkingKey` derivation in user's `LN WALLET` should happen as follows:
1. There exists a private `hashingKey` which is derived by user `LN WALLET` using `m/138'/0` path.
2. `LN SERVICE` domain name is extracted from login `LNURL` and then hashed using `hmacSha256(hashingKey, service domain name)`.
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

**Wallet to service interaction flow:**

1. `LN WALLET` scans a QR code and decodes an URL which must contain the following query parameters:
	- `tag` with value set to `login` which means no HTTPS GET should be made yet.
	- `k1` (hex encoded 32 bytes of challenge) which is going to be signed by user's `linkingPrivKey`.
2. `LN WALLET` displays a "Login" dialog which must include a domain name extracted from `LNURL` query string.
3. Once accepted, user `LN WALLET` signs `k1` on `secp256k1` using `linkingPrivKey` and DER-encodes the signature. `LN WALLET` Then issues an HTTPS GET to `LN SERVICE` using `<LNURL_hostname_and_path>?<LNURL_existing_query_parameters>&sig=<hex(sign(k1.toByteArray, linkingPrivKey))>&key=<hex(linkingKey)>` 
4. `LN SERVICE` responds with `{"status":"OK"}` sent back to wallet once signature is verified by service. `linkingKey` should be used as user identifier in this case.


## 3. LNURL-withdraw
### Withdrawing funds from a service

Today users are asked to provide a withdrawal Lightning invoice to a service, this requires some effort and is especially painful when user tries to withdraw funds into mobile wallet while using a desktop website. Instead of asking for Lightning invoice a service could display a "withdraw" QR code which contains a specialized `LNURL`.

**Wallet to service interaction flow:**

1. User scans a LNURL QR code or accesses an `lightning:LNURL..` link with `LN WALLET` and `LN WALLET` decodes LNURL.
2. `LN WALLET` makes an HTTPS GET request to `LN SERVICE` using the decoded LNURL.
3. `LN WALLET` gets Json response from `LN SERVICE` of form:
	
	```
	{
		callback: String, // the URL which LN SERVICE would accept a withdrawal Lightning invoice as query parameter
		k1: String, // random or non-random string to identify the user's LN WALLET when using the callback URL
		maxWithdrawable: MilliSatoshi, // max withdrawable amount for a given user on LN SERVICE
		defaultDescription: String, // A default withdrawal invoice description
		minWithdrawable: MilliSatoshi // An optional field, defaults to 1 MilliSatoshi if not present, can not be less than 1 or more than `maxWithdrawable`
		tag: "withdrawRequest" // type of LNURL
	}
	```
	or
	
	```
	{"status":"ERROR", "reason":"error details..."}
	```
4. `LN WALLET` Displays a withdraw dialog where user can specify an exact sum to be withdrawn which would be bounded by: 
	
	```
	max can receive = min(maxWithdrawable, local estimation of how much can be routed into wallet)
	min can receive = max(minWithdrawable, local minimal value allowed by wallet)
	```
5. Once accepted by the user, `LN WALLET` sends an HTTPS GET to `LN SERVICE` in the form of 
	
	```
	<callback>?k1=<k1>&pr=<lightning invoice, ...>
	```
	
	Note that user may send multiple invoices with a splitted total amount in a single request.
6. `LN SERVICE` sends a `{"status":"OK"}` or `{"status":"ERROR", "reason":"error details..."}` Json response.
7. `LN WALLET` awaits for incoming payment if response was successful.

Note that service will withdraw funds to anyone who can provide a valid ephemeral `k1`. In order to harden this a service may require autorization (LNURL-auth, email link etc.) before displaying a withdraw QR.

## 4. LNURL-pay
### Pay to static QR/NFC/link

**Wallet to service interaction flow:**

1. User scans a LNURL QR code or accesses an `lightning:LNURL..` link with `LN WALLET` and `LN WALLET` decodes LNURL.
2. `LN WALLET` makes an HTTPS GET request to `LN SERVICE` using the decoded LNURL.
3. `LN WALLET` gets Json response from `LN SERVICE` of form:
    
    ```
    {
        callback: String, // the URL from LN SERVICE which will accept the pay request parameters
        maxSendable: MilliSatoshi, // max amount LN SERVICE is willing to receive
        minSendable: MilliSatoshi, // min amount LN SERVICE is willing to receive, can not be less than 1 or more than `maxSendable`
        metadata: String, // metadata json which must be presented as raw string here, this is required to pass signature verification at a later step
        tag: "payRequest" // type of LNURL
    }
    ```
    or
    
    ```
    {"status":"ERROR", "reason":"error details..."}
    ```
    
    `metadata` json array must contain one `text/plain` entry, all other types of entries are optional:
    
    ```
    [
        [
            "text/plain", // must always be present
            content // actual metadata content
        ],
        [
            "image/png;base64", // optional 512x512px PNG thumbnail which will represent this lnurl in a list or grid
            content // base64 string, up to 136536 characters (100Kb of image data in base-64 encoding)
        ],
        [
            "image/jpeg;base64", // optional 512x512px JPG thumbnail which will represent this lnurl in a list or grid
            content // base64 string, up to 136536 characters (100Kb of image data in base-64 encoding)
        ],
        ... // more objects for future types
    ]
    ```
    
    and be sent as a string:
    
    ```
    "[[\"text/plain\", \"lorem ipsum blah blah\"]]"
    ```

4. `LN WALLET` displays a payment dialog where user can specify an exact sum to be sent which would be bounded by:

	```
	max can send = min(maxSendable, local estimation of how much can be sent from wallet)
	
	min can send = max(minSendable, local minimal value allowed by wallet)
	```
	Additionally, a payment dialog must include:
	- Domain name extracted from `LNURL` query string.
	- A way to view the metadata sent of `text/plain` format.

5. `LN WALLET` makes a HTTPS GET request using 
	
	```
	<callback>?amount=<milliSatoshi>&fromnodes=<nodeId1,nodeId2,...>
	```
	where `amount` is user specified sum in MilliSatoshi and `fromnodes` is an optional parameter with value set to comma separated `nodeId`s if payer wishes a service to provide payment routes starting from specified LN `nodeId`s.
6. `LN Service` takes the GET request and returns JSON response of form:
	
	```
	{
		pr: String, // bech32-serialized lightning invoice
		successAction: Object or null, // An optional action to be executed after successfully paying an invoice
		routes: 
		[
			[
				{
					nodeId: String,
					channelUpdate: String // hex-encoded serialized ChannelUpdate gossip message
				},
				... // next hop
			],
			... // next route
		] // array with payment routes, should be left empty if no routes are to be provided
	}
	```
	
	or
	
	```
	{"status":"ERROR", "reason":"error details..."}
	```
	
	`pr` must have the [`h` tag (`description_hash`)](https://github.com/lightningnetwork/lightning-rfc/blob/master/11-payment-encoding.md#tagged-fields) set to `sha256(utf8ByteArray(metadata))`.
	
	Currently supported tags for `successAction` object are `url`, `message`, and `aes`. If there is no action then `successAction` value must be set to `null`. 
	
	```
	{
	   tag: String, // action type
	   ... rest of fields depends on tag value
	}
	```
	
	Examples of `successAction`:
    	
    ```
	{
	   tag: 'url'
	   description: 'Thank you for your purchase. Here is your order details' // Up to 144 characters
	   url: 'https://www.ln-service.com/order/<orderId>' // url domain must be the same as `callback` domain at step 3
	}	
	
	{
	   tag: 'message'
	   message: 'Thank you for using bike-over-ln CO! Your rental bike is unlocked now' // Up to 144 characters
	}
	
	{
	   tag: 'aes'
	   description: 'Here is your redeem code' // Up to 144 characters
	   ciphertext: <base64> // an AES-encrypted data where encryption key is payment preimage, up to 4kb of characters
	   iv: <base64> // initialization vector, exactly 24 characters
	}
	
    ```

7. `LN WALLET` Verifies that `h` tag in provided invoice is a hash of `metadata` string converted to byte array in UTF-8 encoding. 
8. `LN WALLET` Verifies that amount in provided invoice equals an amount previously specified by user.
9. If routes array is not empty: verifies signature for every provided `ChannelUpdate`, may use these routes if fee levels are acceptable.
10. If `successAction` is not null: `LN WALLET` makes sure that `tag` value of is of supported type, aborts a payment otherwise.
11. `LN WALLET` pays the invoice, no additional user confirmation is required at this point.
12. Once payment is fulfilled `LN WALLET` excecutes a non-null `successAction`. For `message`, a toaster or popup is sufficient. For `url`, the wallet should give the user a popup which displays `description`, `url`, and a 'open' button to open the `url` in a new browser tab. For `aes`, `LN WALLET` must attempt to decrypt a `ciphertext` with payment preimage. `LN WALLET` should also store `successAction` data on the transaction record.  

### Note on metadata for server-side LNURL-PAY:

**When client makes a first call**

Construct a metadata object, turn it into json, then include in into parent json as string.

**When client makes a second call**

1. Make a hash as follows: `sha256(utf8ByteArray(unescaped_metadata_string))`.
2. Generate a payment request using an obtained hash.

### Note on `aes` action tag
Used encryption type is 256-bit AES in `AES/CBC/PKCS5Padding` mode.
An encryption example in Scala:

```scala
val iv = Tools.random.getBytes(16) // exactly 16 bytes, unique for each secret
val key = Tools.random.getBytes(32) // payment preimage
val data = "Secret data".getBytes

val aesCipher = Cipher getInstance "AES/CBC/PKCS5Padding"
val ivParameterSpec = new IvParameterSpec(iv)
aesCipher.init(Cipher.ENCRYPT_MODE, new SecretKeySpec(key, "AES"), ivParameterSpec)
val cipherbytes = aesCipher.doFinal(data)

val ciphertext64 = ByteVector.view(cipherbytes).toBase64 // Base 64 alphabet as defined by http://tools.ietf.org/html/rfc4648#section-4 RF4648 section 4. Whitespace is ignored.
val iv64 = ByteVector.view(iv).toBase64 // 16 bytes results in exactly 24 characters
```
