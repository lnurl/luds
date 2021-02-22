# LNURL-deposit

## Direct deposit of funds from a service

Today users are asked to provide a withdrawal Lightning invoice to a service, this requires some effort and is especially painful when user tries to withdraw funds into mobile wallet while using a desktop website. Instead of asking for Lightning invoice a service could link a "direct deposit" relationship with a wallet through a URL-based callback scheme.

### Wallet to service interaction flow:

1. User scans a LNURL QR code or accesses an `lightning:LNURL..` link with `LN WALLET` and `LN WALLET` decodes LNURL.

2. `LN WALLET` makes a GET request to `LN SERVICE` using the decoded LNURL.

3. `LN WALLET` gets JSON response from `LN SERVICE` of form:
    ```
    {
        callback: String, // the confirmation URL which the LN SERVICE expects to receive the depositing parameters
        k1: String, // random or non-random string to identify the user's LN WALLET when using the callback URL
        maxWithdrawable: MilliSatoshi, // max withdrawable amount for a given user on LN SERVICE
        defaultDescription: String, // A default direct deposit invoice description
        keysend: Boolean, // Set true to use keysend instead of regular invoices
        minWithdrawable: MilliSatoshi // An optional field, defaults to 1 MilliSatoshi if not present, can not be less than 1 or more than `maxWithdrawable`
        tag: "directDepositRequest" // type of LNURL
    }
    ```
    or

    ```
    {"status": "ERROR", "reason": "error details..."}
    ```

4. `LN WALLET` Displays a direct deposit confirmation dialog allowing `LN_SERVICE` to send payments to the user's wallet without interactions. This permission can be revoked by the user at any time.

5. Once accepted by the user, the `LN SERVICE` can use the callback (or keysend) given by `LN WALLET` to send funds to the user at any time in the future (until the direct deposit relationship is revoked by the wallet). An example of a URL callback to make a deposit would be initiated as:

	```
	<callback>?k1=<k1>&action=getInvoice&amount=...
	```
  
An example of a non-URL keysend to make a deposit would be initiated as:

  ```
  lncli keysend <wallet_node> "{userKey: ...}"
  ```

6. `LN WALLET` responds with an invoice

  ```
  {"status": "OK", "directDepositInvoice": "lnbc1..."}
  ```
  or
   
  ```
  {"status": "ERROR", "reason": "error details..."}
  ```
  
7. `LN SERVICE` then attempts to pay the invoices asynchronously.

7. `LN WALLET` awaits for incoming payment and automatically credits the user's balance without any interaction (or with notification).

Note that service will withdraw funds to anyone who can provide a valid ephemeral `k1`. In order to harden this a service may require autorization (LNURL-auth, email link etc.) before registering the direct deposit.

Additionally, `LN SERVICE` and `LN WALLET` developers can also choose to implement a _fast LNURL-deposit_. This implementation reduces the steps involved in the process by putting the data that would be sent by `LN SERVICE` in step 3. as query parameters of the `LN SERVICE` URL accessed in step 1., before it is bech32-encoded.

Eg:

	https://LNserviceURL
	?tag=directDepositRequest
	&k1=String
	&minWithdrawable=MilliSatoshi
	&maxWithdrawable=MilliSatoshi
	&defaultDescription=String
	&callback=String

This fast LNURL-deposit method is not to be confused as an alternative to the original LNURL-deposit, and is designed to be only be used for `lightning:`-type links that work between apps. It is not suitable for QR code implementations.

If a `LN SERVICE` developer chooses to implement fast LNURL-deposit in their app, the encoded URL with query params must still return a JSON response containing data that would be sent in step 3. when a GET request is made to it. This is required so as to be backwards-compatible with `LN WALLET`s which have only implemented the original LNURL-deposit method.

If a `LN WALLET` developer chooses to implement fast LNURL-deposit in their app, they will need to handle for both fast and original LNURL-deposit methods as `LN SERVICE`s mostly use the original method.
