# LNURL-allowance

The LNURL Allowance scheme allows a `LN SERVICE` and a `LN WALLET` to establish a `pairing` over a WebSocket connection. The idea is that `LN SERVICE` will provide an LNURL that returns some parameters (including a socket URL) to `LN WALLET`. `LN WALLET` will diplay a pairing dialog and once accepted by the user, `LN WALLET` will start a socket connection to `LN SERVICE`. This will allow for `LN WALLET` to pay/receive funds to/from `LN SERVICE` without user interaction or QR Code scanning, up to the established allowance amount.

## Scan QR code to establish allowance/pairing

### WALLET to SERVICE interaction flow:

1. User scans a LNURL QR code or accesses a `lightning:LNURL..` link with `LN WALLET` and `LN WALLET` decodes LNURL.

2. `LN WALLET` makes a GET request to `LN SERVICE` using the decoded LNURL.

3. `LN WALLET` gets JSON response from `LN SERVICE` of form:

```
{
    socket: String, // the URL from LN SERVICE which will accept the WebSocket connections from the WALLET
    k1: String, // random or non-random string to identify the WALLET when receiving connection to socket
    recommendedAllowanceAmount: Millsatoshis, // amount of satoshis the SERVICE recommends WALLET to ask user for this allowance/pairing
    image: Base64, // base64-encoded LN SERVICE image for LN WALLET to display to user when establishing connection
    description: String, // LN SERVICE description that LN WALLET should showcase to user
    tag: "allowanceRequest" // type of LNURL
}
```
or

```
{
  status: "ERROR",
  reason: "Error message"
}
```

4. `LN WALLET` displays a `pairing/allowance` dialog where user can specify an exact sum to be `spendable`:

```
- allowance amount = or(recommendedAllowanceAmount, local value entered by user)
```

Additionally, the pairing dialog must include:
- Domain name extracted from `LNURL` query string
- A way to view the image and description fields from LNURL response

5. `LN WALLET` initiates a WebSocket connection to the `socket` URI provided in previous LNRUL response, and if connection is successful `LN WALLET` sends first identifying JSON strigified payload message. WebSocket connection must be SSL'ed (e.g. `wss://...` instead of `ws://...`).

> e.g. wss://server.btc/websocket-server

```
payload = {
  // k1 to identify LN WALLET in LN SERVICE
  // received from previous `k1` property
  k1: '1ah6bceef7891...', <String>

  // total millisatoshi amount for all invoices
  // LN WALLET is able to spend to SERVICE
  balance: 20000000, <Number>

  // type of LNURLAllowance message
  type: "allowanceRequest" <Enum>
}

e.g. JSON.stringify(payload)
```

6. `LN SERVICE` receives message, parses the JSON payload and checks the `type` property of the message, finding an `allowanceRequest` type. For this message type `LN SERVICE` must check that `k1` value passed matches a real `k1` inside `LN SERVICE`. If all checks are performed on `LN SERVICE` side and the connection should remain established, `LN SERVICE` sends a JSON stringified message payload to `LN WALLET` with the following format:

```
payload = {
  // success message to show on the WALLET
  message: 'Allowance setup successful', <String>

  // UNIX timestamp of pairing success time
  timestamp: 1593453067, <Number/UNIX Date>

  // type of LNURLAllowance message
  type: "allowanceSuccess" <Enum>
}

e.g. JSON.stringify(payload)
```

or

```
{
  status: "allowanceError",
  reason: "Error message" // e.g. Invalid `k1` value
}
```

`LN SERVICE` should disconnect the socket connection with `LN WALLET` after pushing an `allowanceError` message.

7. Assuming `LN SERVICE` and `LN WALLET` can pair, and `LN SERVICE` sends the `allowanceSuccess` payload back to `LN WALLET`, `LN WALLET` shows a successsful socket pairing message with the `message` text from the `LN SERVICE` payload.

8. `LN WALLET` must now be responsible for allowance amount:
  - current amount spent < `LN WALLET`'s `balance`

Now that `LN WALLET` is `paired` with `LN SERVICE`, the two can send messages containing Lightning invoices that can be paid automatically until the `balance` criteria of the `LN WALLET` is met.

## WebSocket Persistence

In order to maintain a WebSocket connection alive for long periods of time, it is advised/necessary that the `LN WALLET` (client) and the `LN SERVICE` (server) send `ping/pong` messages to each other every so often.

Given this requirement the `LN WALLET` must send a `balance` message payload to the `LN SERVICE` periodically (e.g. every 15 seconds).

```
payload = {
  // millisatoshis amount in the allowance balance
  balance: 15000000, <Number>

  // current UNIX timestamp of message
  timestamp: 1593453067, <Number/UNIX Date>

  // type of LNURLAllowance message
  type: "balance" <Enum>
}

e.g. JSON.stringify(payload)
```

## Requesting invoice from `LN WALLET` (SERVICE to WALLET)

On an already established WebSocket connection between `LN WALLET` and `LN SERVICE`:

1. `LN SERVICE` sends a JSON stringified message payload with the following format:

```
payload = {
  // message type requesting balance
  type: "invoiceRequest", <Enum>

  // amount for invoice in millisatoshis
  amount: 4000, <Number>
}

e.g. JSON.stringify(payload)
```

2. `LN WALLET` receives message, parses its contents and finds the `invoiceRequest` message type, responding with the following JSON stringified payload:

```
payload = {
  // message type requesting balance
  type: "invoiceSuccess", <Enum>

  // payment request lnbc
  invoice: "lnbc1500n", <String>
}

e.g. JSON.stringify(payload)
```

3. `LN SERVICE` must check whether the invoice received actually matches the amount `LN SERVICE` asked for, and if so attempts to pay it.

## Requesting payment from `LN WALLET` (SERVICE to WALLET)

On an already established WebSocket connection between `LN WALLET` and `LN SERVICE`:

1. `LN SERVICE` sends a JSON stringified message payload with the following format:

```
payload = {
  // message type requesting payment
  type: "paymentRequest", <Enum>

  // amount for invoice in millisatoshis
  amount: 4000, <Number>

  // payment request
  invoice: 'lnbc...' <String>
}

e.g. JSON.stringify(payload)
```

2. `LN WALLET` receives message, parses its contents and finds the `paymentRequest` message type, verifies that the amount asked for matches the invoice details, and that the current allowance balance is greater than the amount for the invoice, then attempts to pay it.

### LNURL Allowance Socket Message Types

These are the support WebSocket message types that WALLET and SERVICE should be aware of.

- Allowance
  - `allowanceRequest` --> WALLET identifying/asking for allowance with SERVICE
  - `allowanceSuccess` --> SERVICE confirming allowance paired with WALLET
- Payment
  - `paymentRequest` --> SERVICE asking WALLET to pay an invoice
- Invoice
  - `invoiceRequest` --> SERVICE asking WALLET for an invoice
  - `invoiceSuccess` --> WALLET responding to SERVICE with invoice
- Balance
  - `balance`        --> WALLET sending SERVICE current balance

### Known Issues

While it is trivial to maintain a WebSocket connection opened in Web environments, it becomes a hurdle in Mobile environments. For iOS devices, there isn't really a proper way to maintain that connection active, apart from forcing the device to stay awake during the duration of the `allowance`. For Android devices the recommended approach is to run a [Service](https://developer.android.com/guide/components/services.html) that handles the flow of messages to/from the device on the background.


