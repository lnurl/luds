`lnurl` is a bech32-encoded HTTPS query string which is supposed to help payer interact with payee and thus simplify a number of standard scenarios such as requesting incoming channels, withdrawing funds, doing atomic swaps etc.

This example `lnurl`: 
> https://service.com/api?q=3fc3645b439ce8e7f2553a69e5267081d96dcd340693afabe04be7b0ccd178df

would be bech32-encoded as:

> LNURL1DP68GURN8GHJ7UM9WFMXJCM99E3K7MF0V9CXJ0M385EKVCENXC6R2C35XVUKXEFCV5MKVV34X5EKZD3EV56NYD3HXQURZEPEXEJXXEPNXSCRVWFNV9NXZCN9XQ6XYEFHVGCXXCMYXYMNSERXFQ5FNS

A QR encoded form:

![A QR encoded LNUrl example](https://i.imgur.com/HbB7U1K.png)

`lnurl` can be presented either directly or be embedded in Lightning invoice if respected `lnurl` usage scenario can be gracefully degraded to just using an invoice. When embedded in Lightning invoice a letter `l` is used as tag identifier with decoding rules identical to Description tag `d`. A `lightning:` prefix may be used how it's currently used in Lightning invoices.

Once decoded user software should make sure an `LNUrlTag` indeed contains an `https` query string, then it should make an HTTP GET request which must return a Json object containing a on obligatory `tag` field, this way user software would know what this `lnurl` is about. 


# Example usage scenarios

## Incoming payment channel request  
Suppose user has a balance on a certain service which he wishes to turn into an incoming channel and service supports such functionality. This would require many parameters so resulting QR may be overly dense and cause scanning issues. Other than that, when using a mobile wallet user has to make sure that a connection to target LN node is established from mobile client before an incoming channel is requested.

User software:
1. Scans a QR code and decodes a query string.
2. Makes an HTTP GET request to a service.
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
5. Issues an HTTP GET request using `<callback>?k1=<k1>&remoteid=<Local LN node ID>&private=<1/0>`
6. Awaits for incoming `OpenChannel` message via Lightning socket connection which would initiate a channel opening.


## Withdrawing funds
Today users are asked to provide a withdrawal Lightning invoice to a service, this requires some effort and is especially painful when user tries to withdraw funds into mobile wallet using a website on a desktop. Instead of asking for Lightning invoice a service could display a "withdraw" QR code which contains a specialized `lnurl`.

User software:
1. Scans a QR code and decodes a query string.
2. Makes an HTTP GET request to a service.
3. Gets Json response of form: 
```
{
	callback: String, // a second-level url which would accept a withdrawal Lightning invoice as query parameter
	k1: String, // a second-level secret to authorize user request 
	max: MilliSatoshi, // max withdrawable amount for a given user on a given service
	tag: "withdrawRequest" // Now user software knows what to do next...
}
```
4. Displays a withdraw dialog where user can specify an exact sum to be withdrawn which would be bounded by: 
```
min(max amount withdrawable from service, local estimation of how much can be routed into wallet)
```
5. Issues an HTTP GET request using `<callback>?k1=<k1>&pr=<Lightning invoice with user defined amount>`
6. Awaits for incoming payment.
