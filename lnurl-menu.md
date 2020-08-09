# LNURL-menu

## Navigating through a service

Provides a way for services to supply wallets of other LNURL options they have to offer.

### Wallet to service interaction flow:

1. User scans a LNURL QR code or accesses an `lightning:LNURL..` link with `LN WALLET` and `LN WALLET` decodes LNURL.  

2. `LN WALLET` makes a GET request to `LN SERVICE` using the decoded LNURL.  

3. `LN WALLET` gets JSON response from `LN SERVICE` of form:  
    ```
    {
        options: [
          ["Option A", "/path/to/option/a"],
          ["Option B", "/path/to/option/b"],
          ...
        ],
        tag: "menuRequest" // type of LNURL
    }
    ```
    or
    
    ```
    {"status":"ERROR", "reason":"error details..."}
    ```

4. `LN WALLET` Displays all options in the `"options"` array as buttons.
5. When a button is clicked, `LN WALLET` proceeds to call the given URL path (prefixed by the same domain it had received the initial menu options from) as if it was an LNURL just scanned from a QR code.

This allows for `LN SERVICE`s to offer a fully in-wallet interface to their services. A user can choose between multiple items, each leading to an [lnurl-pay](lnurl-pay.md) endpoint, choose between an [lnurl-channel](lnurl-channel.md) and an [lnurl-withdraw](lnurl-withdraw.md) when getting funds from a fiat-to-LN service or an ATM machine, or perhaps leading to another **lnurl-menu** so options can be chained.
