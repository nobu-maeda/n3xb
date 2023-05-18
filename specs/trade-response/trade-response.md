# n3xB Trade Response Message
*See [Architecture](/specs/architecture/architecture.md) for where Trade Response Message fits in the overall n3xB protocol flow*

This specifies the message format inside a n3xB Peer Message for `message_type = n3xB-trade-response`.

A Trade Response is in response to a Take Order Message to let the Taker know whether the take order have been accepted and the trade should proceed to the Trade Phase. If that's not possible, the Maker should provide best effort to let the Taker know the reason why.

## Reliability

To improve robustness before entering into a Trade Phase, Maker should also ensure that at least several of the relays listed by the Taker's Relay List Metadata ([NIP-65](https://github.com/nostr-protocol/nips/blob/master/65.md)) is added as backup relays in the Maker's own relay list.

## Maker Considerations

Maker should carefully check all parameters specified from the Taker's Take Order Message against its copy and view of the trade to ensure their validity. Particularly the Take Order parameters should be checked to ensure they are within the allowable range and options initially specified in the Maker Order Note. In general, the Maker should always cross check any incoming messages against its own view and state of the trade before further proceeding with a trade. Trade Engines should be designed in a similarly trustless manner to maintain the decentralization properties of the protocol.

## Trade Engine Considerations

Trade Engine implementations should be careful to minimize the amount of personally identifiable information inside Trade Engine specific details despite all being encrypted. Trade Engine should definitely avoid transferring actual value at this point, as there is still no guarantee from the Taker whatsoever at this point of the trade.

## Trade Response Message JSON
```
{
  ...
  "message_type": "n3xB-trade-response"
  "message": {
    "trade_response_code": <response string code>
    "reject_reason_code": <array of reason string codes. Omit if n/a>
    "trade_engine_specifics: <trade engine specific arbitrary JSON>
  }
  ...
}
```


| `trade_response_code` | description                                   |
| --------------------- | --------------------------------------------- |
| accepted              | Take order accepted. Proceed to Trading Phase |
| rejected              | Take order rejected                           |
| not_available         | Order no longer available                     |


| `reject_reason_code`       | description                                                     |
| -------------------------- | --------------------------------------------------------------- |
| pending                    | Order is pending another Taker                                  |
| invalid-maker-currency     | Maker currency is invalid or not in the acceptable set          |
| invalid-maker-settlement   | Maker settlement method is invalid or not in the acceptable set |
| invalid-taker-currency     | Taker currency is invalid or not in the acceptable set          |
| invalid-taker-settlement   | Taker settlement method is invalid or not in the acceptable set |
| invalid-market-oracle      | Market oracle URL is invalid or not in the acceptable set       |
| maker-amount-out-of-range  | Maker amount is out of acceptable range                         |
| exchange-rate-out-of-range | Exchange rate is out of acceptable range                        |
| maker-bond-out-of-range    | Maker bond is out of acceptable range                           |
| taker-bond-out-of-range    | Taker bond is out of acceptable range                           |
| trade-engine-specific      | Reason provided in `trade_engine_specifics` JSON                |
| pow-too-high               | The Taker desired minimum PoW is too high for the Maker         |

One way to view a `rejected` response vs a `not_available` response is that a Taker might want to update the Take Order message according to the reject reason and retry. Where-as if the order is no longer available, there is no point for the Taker to retry.

## Taker Handling

Given a rejection, the Taker is free to retry taking the order again, unless the order is not available all together. [PoW in the n3xB Peer Messaging scheme](/specs//peer-messaging/peer-messaging.md#proof-of-work) should help to prevent Take Order Message SPAM/DoS attacks.