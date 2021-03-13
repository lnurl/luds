# LNURL-withdraw

## Withdrawing funds from a service

Today users are asked to provide a withdrawal Lightning invoice to a service, this requires some effort and is especially painful when user tries to withdraw funds into mobile wallet while using a desktop website. Instead of asking for Lightning invoice a service could display a "withdraw" QR code which contains a specialized `LNURL`.

### Wallet to service interaction flow:

1. User scans a LNURL QR code or accesses an `lightning:LNURL..` link with `LN WALLET` and `LN WALLET` decodes LNURL.

2. `LN WALLET` makes a GET request to `LN SERVICE` using the decoded LNURL.

3. `LN WALLET` gets JSON response from `LN SERVICE` of form:
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
    {"status": "ERROR", "reason": "error details..."}
    ```

4. `LN WALLET` Displays a withdraw dialog where user can specify an exact sum to be withdrawn which would be bounded by:

	```
	max can receive = min(maxWithdrawable, local estimation of how much can be routed into wallet)
	min can receive = max(minWithdrawable, local minimal value allowed by wallet)
	```
5. Once accepted by the user, `LN WALLET` sends a GET to `LN SERVICE` in the form of

	```
	<callback><?|&>k1=<k1>&pr=<lightning invoice, ...>
	```
6. `LN SERVICE` sends a `{"status": "OK"}` or `{"status": "ERROR", "reason": "error details..."}` JSON response and then attempts to pay the invoices asynchronously.
7. `LN WALLET` awaits for incoming payment if response was successful.

Note that service will withdraw funds to anyone who can provide a valid ephemeral `k1`. In order to harden this a service may require autorization (LNURL-auth, email link etc.) before displaying a withdraw QR.

Additionally, `LN SERVICE` and `LN WALLET` developers can also choose to implement a _fast LNURL-withdraw_. This implementation reduces the steps involved in the process by putting the data that would be sent by `LN SERVICE` in step 3. as query parameters of the `LN SERVICE` URL accessed in step 1., before it is bech32-encoded.

Eg:

	https://LNserviceURL
	?tag=withdrawRequest
	&k1=String
	&minWithdrawable=MilliSatoshi
	&maxWithdrawable=MilliSatoshi
	&defaultDescription=String
	&callback=String

This fast LNURL-withdraw method is not to be confused as an alternative to the original LNURL-withdraw, and is designed to be only be used for `lightning:`-type links that work between apps. It is not suitable for QR code implementations.

If a `LN SERVICE` developer chooses to implement fast LNURL-withdraw in their app, the encoded URL with query params must still return a JSON response containing data that would be sent in step 3. when a GET request is made to it. This is required so as to be backwards-compatible with `LN WALLET`s which have only implemented the original LNURL-withdraw method.

If a `LN WALLET` developer chooses to implement fast LNURL-withdraw in their app, they will need to handle for both fast and original LNURL-withdraw methods as `LN SERVICE`s mostly use the original method.
