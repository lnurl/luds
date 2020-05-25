![Logo](media/logo/logo_600.png)

# LNURL: Lightning Network UX protocol

`LNURL` is a bech32-encoded HTTPS/Onion query string which is supposed to help payer interact with payee and thus simplify a number of standard scenarios such as:

- Requesting incoming channels ([LNURL-channel](lnurl-channel.md)) 
- Logging in ([LNURL-auth](lnurl-auth.md))
- Withdrawing funds ([LNURL-withdraw](lnurl-withdraw.md))
- Paying for a service ([LNURL-pay](lnurl-pay.md))

An example `LNURL`: 
> https://service.com/api?q=3fc3645b439ce8e7f2553a69e5267081d96dcd340693afabe04be7b0ccd178df

would be bech32-encoded as:
> LNURL1DP68GURN8GHJ7UM9WFMXJCM99E3K7MF0V9CXJ0M385EKVCENXC6R2C35XVUKXEFCV5MKVV34X5EKZD3EV56NYD3HXQURZEPEXEJXXEPNXSCRVWFNV9NXZCN9XQ6XYEFHVGCXXCMYXYMNSERXFQ5FNS

and presented as the following QR:

![A QR encoded LNURL example](https://i.imgur.com/HbB7U1K.png)

Once `LNURL` is decoded:
- If `tag` query parameter is present then this `LNURL` has a special meaning, further actions will be based on `tag` parameter value.
- Otherwise a GET request should be executed which must return a Json object containing a `tag` field, further actions will be based on `tag` field value.

## HTTPS or Onion

`LNURL` is acceptable in two forms: either an `https://` clearnet link (no self-signed certificates allowed) or an `http://` v2/v3 onion link.

## Fallback scheme

`LNURL` can be used as fallback inside of other URI schemes, with the key 'lightning' and the value equal to the bech32-encoding, an example: `https://service.com/giftcard/redeem?id=123&lightning=LNURL1...`

## Decoding examples

In Scala:
```scala
import fr.acinq.bitcoin.Bech32
val bech32lnurl: String = "LNURL1DP68GURN8GHJ7UM9WFMXJCM99E3K7MF0V9CXJ0M385EKVCENXC6R2C35XVUKXEFCV5MKVV34X5EKZD3EV56NYD3HXQURZEPEXEJXXEPNXSCRVWFNV9NXZCN9XQ6XYEFHVGCXXCMYXYMNSERXFQ5FNS"
  
val (hrp, dataPart) = Bech32.decode(bech32lnurl)
  
val requestByteArray = Bech32.five2eight(dataPart)
  
new String(requestByteArray, "UTF-8") // https://service.com/api?q=3fc3645b439ce8e7f2553a69e5267081d96dcd340693afabe04be7b0ccd178df
```

## Getting help

If you have any questions about implementing LNURL as a wallet or service, join us in the [Telegram group](https://t.me/lnurl) and get help from the creators and other LNURL implementors.

## Protocols

1. [LNURL-channel](lnurl-channel.md)
2. [LNURL-auth](lnurl-auth.md)
3. [LNURL-withdraw](lnurl-withdraw.md)
4. [LNURL-pay](lnurl-pay.md)
