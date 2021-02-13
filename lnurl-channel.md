# LNURL-channel

## Incoming payment channel request

Suppose user has a balance on a certain service which he wishes to turn into an incoming channel and service supports such functionality. This would require many parameters so the resulting QR may be overly dense and cause scanning issues. Additionally, the user has to make sure that a connection to target LN node is established before an incoming channel is requested.

### Wallet to service interaction flow:

1. User scans a LNURL QR code or accesses an `lightning:LNURL..` link with `LN WALLET` and `LN WALLET` decodes LNURL.
2. `LN WALLET` makes a GET request to `LN SERVICE` using the decoded LNURL.
3. `LN WALLET` gets JSON response from `LN SERVICE` of form:

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
	{"status": "ERROR", "reason": "error details..."}
	```

4. `LN WALLET` opens a connection to the target node using `uri` field.

5. `LN WALLET` issues a GET request to `LN SERVICE` using `<callback>?k1=<k1>&remoteid=<Local LN node ID>&private=<1/0>` if user decides to proceed or `<callback>?k1=<k1>&remoteid=<Local LN node ID>&cancel=<1/0>` if user decides to cancel a channel midway.
6. `LN SERVICE` sends a `{"status": "OK"}` or `{"status": "ERROR", "reason": "error details..."}` JSON response.
7. `LN WALLET` awaits for incoming `OpenChannel` message from the target node which would initiate a channel opening.

# LNURL-hosted-channel

## Hosted channel request

## Wallet to service interaction flow:

1. User scans a LNURL QR code or accesses an `lightning:LNURL..` link with `LN WALLET` and `LN WALLET` decodes LNURL.
2. `LN WALLET` makes a GET request to `LN SERVICE` using the decoded LNURL.
3. `LN WALLET` gets JSON response from `LN SERVICE` of form:

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
    {"status": "ERROR", "reason": "error details..."}
    ```
4. `LN WALLET` opens a connection to the target node using `uri` field.
5. Once connected, `LN WALLET` sends an `InvokeHostedChannel` message to the target node using `k1` converted to byte array.
6. The rest is handled by hosted channel protocol.
