# LNURL-allowance

The LNURL Allowance scheme allows a SERVICE and a WALLET to establish a `pairing` over a WebSocket connection. The idea is that SERVICE will provide some recommended criteria (pairing amount and pairing time) to WALLET and once displayed and accepted by the user of WALLET, will allow for WALLET to pay/receive funds to/from SERVICE without user interaction or QR Code scanning.

## Scan QR code to establish allowance/pairing

### WALLET to SERVICE interaction flow:

1. User scans a LNURL QR code or accesses a `lightning:LNURL..` link with `LN WALLET` and `LN WALLET` decodes LNURL.

2. `LN WALLET` makes a GET request to `LN SERVICE` using the decoded LNURL.

3. `LN WALLET` gets JSON response from `LN SERVICE` of form:

```
{
    socket: String, // the URL from LN SERVICE which will accept the WebSocket connections from the WALLET
    k1: String, // random or non-random string to identify the WALLET when receiving connection to socket
    requestAllowanceTime: Seconds, // amount of time the SERVICE recommends WALLET to ask user for this allowance/pairing
    requestAllowanceAmount: Seconds, // amount of seconds the SERVICE recommends WALLET to ask user for this allowance/pairing
    image: Base64, // base64-encoded SERVICE image for WALLET to display to user when establishing connection
    description: String, // SERVICE description that WALLET should showcase to user
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
- allowance amount = or(requestAllowanceAmount, local value entered by user)
```

Additionally, the pairing dialog must include:
- Domain name extracted from `LNURL` query string
- A way to view the image and description fields from LNURL response

5. `LN WALLET` initiates a WebSocket connection to the `socket` URI provided in previous LNRUL response, and if connection is successful `LN WALLET` sends first identifying JSON strigified payload message:

> e.g. wss://server.btc/websocket-server

```
payload = {
  // k1 to identify LN WALLET in SERVICE
  // received from previous `k1` property
  k1: '1ah6bceef7891...', <String>

  // total millisatoshi amount for all invoices
  // WALLET is able to spend to SERVICE
  balance: 20000000, <Number>

  // type of LNURLAllowance message
  type: "allowanceRequest" <Enum>
}

e.g. JSON.stringify(payload)
```

6. `LN Service` receives message, parses the JSON payload and checks the `type` property of the message, find a `allowanceRequest` type. For this message type SERVICE must check that that `k1` value passed matches a real `k1` inside SERVICE. If all checks are performed on SERVICE side and the connection should remain established, SERVICE sends a JSON stringified message payload to WALLET with the following format:

```
payload = {
  // success message to show on the WALLET
  message: 'Allowance setup successfully', <String>

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

SERVICE should disconnect the socket connection with WALLET after pushing an `allowanceError` message.

7. Assuming `SERVICE` and `LN WALLET` can pair, and `SERVICE` sends the `allowanceSuccess` payload back to `LN WALLET`, `LN WALLET` shows a successsful socket pairing message with the `message` text from the `SERVICE` payload.

8. `LN WALLET` must now be responsible for allowance amount:
  - current amount spent < `LN WALLET`'s `balance`

Now that `LN WALLET` is `paired` with `SERVICE`, the two can send messages containing Lightning invoices that can be paid automatically until either the `time` or `amount` criteria of the `LN WALLET` is met.

## WebSocket Persistence

In order to maintain a WebSocket connection alive for long periods of time, it is advised/necessary that the WALLET (client) and the SERVICE (server) send `ping/pong` messages to each other every so often.

Given this requirement the WALLET must send a `balance` message payload to the SERVICE periodically (e.g. every 15 seconds).

```
payload = {
  // millisatoshis amount in the temporary balance
  balance: 15000000, <Number>

  // current UNIX timestamp of message
  timestamp: 1593453067, <Number/UNIX Date>

  // type of LNURLAllowance message
  type: "balance" <Enum>
}

e.g. JSON.stringify(payload)
```

## Requesting invoice from LN WALLET (SERVICE to WALLET)

On an already established WebSocket connection between `LN WALLET` and `SERVICE`:

1. `SERVICE` sends a JSON stringified message payload with the following format:

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

3. `SERVICE` must check whether the invoice received actually matches the amount `SERVICE` asked for, and if so attempts to pay it.

## Requesting payment from LN WALLET (SERVICE to WALLET)

On an already established WebSocket connection between `LN WALLET` and `SERVICE`:

1. `SERVICE` sends a JSON stringified message payload with the following format:

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
  - `allowanceSuccess` --> SERVICE confirming allowance pairsed with WALLET
- Payment
  - `paymentRequest` --> SERVICE asking WALLET to pay an invoice
- Invoice
  - `invoiceRequest` --> SERVICE asking WALLET for an invoice
  - `invoiceSuccess` --> WALLET responding to SERVICE with invoice
- Balance
  - `balance`        --> WALLET sending SERVICE current balance
