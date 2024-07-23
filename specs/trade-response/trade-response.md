# n3xB Trade Response Message

_See [Architecture](/specs/architecture/architecture.md) for where Trade Response Message fits in the overall n3xB protocol flow_

This specifies the message format inside a n3xB Peer Message for `message_type = n3xB-trade-response`.

A Trade Response is in response to a Take Order Message to let the Taker know whether the take order have been accepted and the trade should proceed to the Trade Phase. If that's not possible, the Maker should provide best effort to let the Taker know the reason why.

## Timeout

Note that a Maker has a specific amount of time to accept or reject an offer from a Taker, otherwise the Taker shall time-out and assume failure. The Taker Trade Engine might want to consider marking the Order as bad to avoid repeated attempt in trying to take the Order. The specific time for this time-out needs to be Trade Engine dependent, but such a time-out shall be implemented at the n3xB protocol level.

## Reliability

To improve robustness before entering into a Trade Phase, Maker should also ensure that at least several of the relays listed by the Taker's Relay List Metadata ([NIP-65](https://github.com/nostr-protocol/nips/blob/master/65.md)) is added as backup relays in the Maker's own relay list.

## Maker Considerations

Maker should carefully check all parameters specified from the Taker's Take Order Message against its copy and view of the trade to ensure their validity. Particularly the Take Order parameters should be checked to ensure they are within the allowable range and options initially specified in the Maker Order Note. In general, the Maker should always cross check any incoming messages against its own view and state of the trade before further proceeding with a trade. Trade Engines should be designed in a similarly trustless manner to maintain the decentralization properties of the protocol.

## Trade Engine Considerations

Trade Engine implementations should be careful to minimize the amount of personally identifiable information inside Trade Engine specific details despite all being encrypted. Trade Engine should definitely avoid transferring actual value at this point, as there is still no guarantee from the Taker whatsoever at this point of the trade.

## Trade Response Message JSON

```
"message": {
  "type": "n3xB-trade-response"
  "offer_event_id": <32-bytes lowercase hex id of the Offer being accepted>
  "trade_response": <response string code>
  "reject_reason": <array of reason string codes. Omit if n/a>
  "trade_engine_specifics: {
    "type": <trade engine specifics identifier, as string>
    ... trade engine specific JSON fields ...
  }
}
```

| `trade_response` | description                                   |
| ---------------- | --------------------------------------------- |
| accepted         | Take order accepted. Proceed to Trading Phase |
| rejected         | Take order rejected                           |
| not_available    | Order no longer available                     |

| `reject_reason`              | description                                                   |
| ---------------------------- | ------------------------------------------------------------- |
| Pending                      | Order is pending another Taker Offer                          |
| DuplicateOffer               | Offer already previously received                             |
| MakerObligationKindInvalid   | Maker obligation kind is invalid or not in acceptable set     |
| MakerObligationAmountInvalid | Maker obligation amount is invalid or not in acceptable range |
| MakerBondInvalid             | Maker bond is invalid or not in acceptable range              |
| TakerObligationKindInvalid   | Taker obligation kind is invalid or not in acceptable set     |
| TakerObligationAmountInvalid | Taker obligation amount is invalid or not in acceptable range |
| TakerBondInvalid             | Taker bond is invalid or not in acceptable range              |
| ExchangeRateInvalid          | Exchange rate specified is invalid                            |
| MarketOracleInvalid          | Market oracle specified is invalid or not in acceptable set   |
| TradeEngineSpecific          | Reason provided in `trade_engine_specifics`` JSON             |
| PowTooHigh                   | The Taker desired minimum PoW is too high for the Maker       |

One way to view a `rejected` response vs a `not_available` response is that a Taker might want to update the Take Order message according to the reject reason and retry. Where-as if the order is no longer available, there is no point for the Taker to retry.

## Taker Handling

Given a rejection, the Taker is free to retry taking the order again, unless the order is not available all together. [PoW in the n3xB Peer Messaging scheme](/specs//peer-messaging/peer-messaging.md#proof-of-work) should help to prevent Take Order Message SPAM/DoS attacks.
