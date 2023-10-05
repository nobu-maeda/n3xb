# n3xB Take Order Message

_See [Architecture](/specs/architecture/architecture.md) for where Take Order Message fits in the overall n3xB protocol flow_

This specifies the message format inside a n3xB Peer Message for `message_type = n3xB-take-order`

## Reliability

To improve robustness before entering into a Trade Phase, Taker should make sure to at least have a Relay List Metadata Note ([NIP-65](https://github.com/nostr-protocol/nips/blob/master/65.md)) at the relay that it have taken the Maker Order Note from. Taker should also ensure that at least several of the relays listed by the Maker's Relay List Metadata ([NIP-65](https://github.com/nostr-protocol/nips/blob/master/65.md)) is added as backup relays in the Taker's own relay list.

## Taker Considerations

Some outstanding orders have ranges of parameters to choose from, or some parameter might adjust based on market conditions. For example, a Taker can choose to take a partial amount. Or the order have specified a market offset which a Taker needs to query from one of the approved oracles to calculate what the final exchange rate might be. It is up to the taker to actually come up with an acceptable, valid and concrete final trade proposal and send it back to the Maker as part of the Take Order Message.

For all parameters and states related to the trade, the Taker should always have its own copy and view of the trade without relying on any information in the relays or information remitted by the Maker as the trade progresses. The Taker should always cross check any incoming messages against its own view and state of the trade before further proceeding with a trade. Trade Engines should be designed in a similarly trustless manner to maintain the decentralization properties of the protocol.

## Trade Engine Considerations

Trade Engine implementations should be careful to minimize the amount of personally identifiable information inside Trade Engine specific details despite all being encrypted. Trade Engine should definitely avoid transferring actual value at this point, as there is no guarantee from the Maker whatsoever at this point of the trade.

## Take Order Message JSON

```
"message": {
  "type": "n3xB-taker-offer"

  "maker_obligation": {
    "amount": <amount in integer>
    "currency": <ISO 4217 currency code the 'maker_amount' is denominated in, as string>
    "payment": <payment method string code>
    "bond_amount": <amount of satoshis in integer. Omit if n/a>
  }

  "taker_obligation": {
    "amount": <amount in integer>
    "currency": <ISO 4217 currency code the 'maker_amount' is denominated in, as string>
    "payment": <payment method string code>
    "bond_amount": <amount of satoshis in integer. Omit if n/a>
  }

  "market_oracle_used": <URL of market oracle used. Omit if n/a>

  "trade_engine_specifics: {
    "type": <trade engine specifics identifier, as string>
     ... trade engine specific JSON fields ...
  }

  "pow_difficulty": <A new PoW difficulty for the rest of the trade if the Taker wishes to raise it. Omit if n/a>
}

```
