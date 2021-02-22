# LNURL-deposit

## Direct deposit of funds from a service

Today users are asked to provide a withdrawal Lightning invoice to a service or use LNURL-withdraw every time they want to withdraw. This requires some manual effort and is especially painful when this is needed repeatedly because they prefer to keep all funds in their mobile wallet and off the service. Instead of asking manually requesting withdrawals, a service could link a long-term "direct deposit" relationship with a wallet through a URL-based callback scheme. Then, the service can send payments to the user's mobile wallet asynchronously.

### Wallet to service interaction flow:

1. User scans a LNURL Direct Deposit QR code or accesses an `lightning:LNURL..` link with `LN WALLET` and `LN WALLET` decodes LNURL.

2. `LN WALLET` makes a GET request to `LN SERVICE` using the decoded LNURL.

3. `LN WALLET` gets JSON response from `LN SERVICE` of form:
    ```
    {
        serviceName: String, // the name of the LN SERVICE for the user to identify payments in the future
	supportsKeysend: Boolean, // whether or not the LN SERVICE supports the optional keysend mechanism
        callback: String, // the confirmation URL which the LN SERVICE expects to receive the depositing details
	depositFrequency: String, // an optional description of how often a deposit will be expected (e.g., "Every 2 weeks", "Instant")
	tag: "directDepositRequest" // type of LNURL
    }
    
    ```
    or

    ```
    {"status": "ERROR", "reason": "error details..."}
    ```

4. `LN WALLET` Displays a direct deposit confirmation dialog allowing `LN_SERVICE` to send payments to the user's wallet without interactions. This permission can be revoked by the user at any time.


5. Once the user accepts, the `LN_SERVICE` POSTs the deposit information using the callback in the previous step.

    ```
    {
        callback: String, // the URL which the LN SERVICE should use when requesting a direct deposit invoice for the user
	k1: String, // random or non-random string to identify the user's LN WALLET when using the callback URL
        maxWithdrawable: MilliSatoshi, // max withdrawable amount for a given user on LN SERVICE
        defaultDescription: String, // A default direct deposit invoice description
        keysend: Boolean, // Set true to use keysend instead of regular invoices
        minWithdrawable: MilliSatoshi // An optional field, defaults to 1 MilliSatoshi if not present, can not be less than 1 or more than `maxWithdrawable`
        
    }
    ```

6. Once accepted by the user, the `LN SERVICE` can use the callback (or keysend) given by `LN WALLET` to send funds to the user at any time in the future (until the direct deposit relationship is revoked by the wallet). An example of a URL callback to make a deposit would be initiated as:

    ```
    <callback>?k1=<k1>&action=getInvoice&amount=...
    ```
  
An example of a non-URL keysend (for wallets and services who choose to support it) to make a deposit would be initiated as:

  ```
  lncli keysend <wallet_node> "{userKey: ...}"
  ```

7. `LN WALLET` respond to the URL callback with an invoice (and optional amount):

  ```
  {"status": "OK", "directDepositInvoice": "lnbc1..."}
  ```
  or
   
  ```
  {"status": "ERROR", "reason": "error details..."}
  ```
  
7. `LN SERVICE` then attempts to pay the invoice asynchronously.

8. `LN WALLET` awaits for incoming payment and automatically credits the user's balance without any interaction (or with soft notification).

Note that service will withdraw funds to anyone who can provide a valid ephemeral `k1`. In order to harden this a service may require autorization (LNURL-auth, email link etc.) before registering the direct deposit.

