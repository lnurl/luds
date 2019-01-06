LNUrl is a bech32-encoded HTTPS query string which is supposed to help payer interact with payee and thus simplify a number of standard scenarios.

LNUrl has a special `action` query parameter so user software can determine what it is about and an `lnurl` prefix in encoded form, this example query string: 
> https://service.com/api?q=3fc3645b439ce8e7f2553a69e5267081d96dcd340693afabe04be7b0ccd178df&action=openChannel 

would be bech32-encoded as:

> lnurl1dp68gurn8ghj7um9wfmxjcm99e3k7mf0v9cxj0m385ekvcenxc6r2c35xvukxefcv5mkvv34x5ekzd3ev56nyd3hxqurzepexejxxepnxscrvwfnv9nxzcn9xq6xyefhvgcxxcmyxymnserxyeskxarfdahr6mmsv4hyx6rpdehx2mqvucphd

A QR encoded LNUrl example:

![A QR encoded LNUrl example](https://i.imgur.com/2WwUzAJ.jpg)

LNUrl can be presented either directly or be embedded in Lightning invoice if respected LNUrl usage scenario can be gracefully degraded to just using an invoice. When embedded in Lightning invoice a letter `l` is used as tag identifier with decoding rules identical to Description tag `d`. Once decoded user software should make sure an LNUrlTag indeed contains an `https` query string.


# Example usage scenarios

## Incoming payment channel request  
Suppose user has a balance on a certain service which he wishes to turn into an incoming channel and service supports such functionality. This would require many parameters so resulting QR may be overly dense and cause scanning issues. Other than that, when using a mobile wallet user has to make sure that a connection to target LN node is established from mobile client before an incoming channel is requested.

LNUrl would look like this: 
```
https://service.com/api?q=<unique identifier to authorize user request>&action=openChannel
```

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
	feeProportionalMillionths: Long
}
```
4. Opens a Lightning socket connection to a target node using `uri` field.
5. Issues an HTTP GET request using `$callback?k1=<k1>&remoteid=<Local LN node ID>&private=<1/0>`
6. Awaits for incoming `OpenChannel` message via Lightning socket connection which would initiate a channel opening.


## Withdrawing funds
Today users are asked to provide a withdrawal Lightning invoice to a service, this requires some effort and is especially painful when user tries to withdraw funds into mobile wallet using a website on a desktop. Instead of asking for Lightning invoice a service could display a "withdraw" QR code which contains a specialized LNUrl.

LNUrl would look like this: 
```
https://service.com/api?q=<unique identifier to authorize user request>&max=<max amount withdrawable>&action=withdrawFunds
```

User software:
1. Scans a QR code and decodes a query string.
2. Displays a withdraw dialog where user can specify an exact sum to be withdrawn which would be bounded by: 
```
min(max amount withdrawable from LNUrl, local estimation of how much can be routed into wallet)
```
3. Creates a Lightning invoice with user provided sum, attaches it to decoded LNURL and makes an HTTP GET request of the form: 
```
https://service.com/api?q=<unique identifier to authorize user request>&max=<max amount withdrawable>&action=withdrawFunds&pr=<bech32-encoded invoice>
```
4. Awaits for incoming payment.
