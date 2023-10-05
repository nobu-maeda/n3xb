# n3xB Maker Order Note
*See [Architecture](/specs/architecture/architecture.md) for where Maker Order Note fits in the overall n3xB protocol flow*

## Identification

A n3xB Maker Order Note is a Nostr *Aribtrary custom app data* ([NIP-78](https://github.com/nostr-protocol/nips/blob/master/78.md)) note, and as such can be identified by a `kind` value of `30078`, a tag value of `n3xb` for tag name `#d`, and a tag value of `maker-order` for tag name `#k`.

A specific trade for an earmarked lot of liquidity represented by one or more Maker Order Note shall be assigned a unique but randomly generated 32-bytes lowercase hex Trade-UUID as value for tag name `#i`. This is especially important if a Maker wants the same lot of liquidity to be takable by Takers through multiple Maker Order Notes, of which each allows for different sets of parameters with different Trade Engines specified.

## Pre-Defined Tags

To ease the ability for clients to query without introducing specific logic to relays keeping relays 'dumb' and maximally inter-operable, tags shall be used extensively to help clients in querying only the very specific orders it might be interested in. To do so, when a maker chooses a specific parameter and its tag, all the more generic tags that are true will also be applied automatically. For example when specifying Fiat obligation in USD via Strike, `ob-fiat-usd-strike` will be specified, but `ob-fiat-usd` and `ob-fiat` should also be specified.

### Obligations

All Maker obligation tags will be under tag name `#m`, and all Taker obligation tags will be under the tag name `#t`. When a specific is specified, the more generic tags shall also be included. Obligation is limited to the same kind and currency, as such each order can only specify up to a single trade pair at most. For example the Maker can create an order to buy Bitcoin with USD via multiple payment methods, but cannot use a single order to specify the desire to buy Bitcoin with both USD and EUR at the same time.

At the n3xB level, only the trade intent at the user level will be conveyed. More specific parameters, like the Bitcoin Script Sig that might be used (P2PKH, P2WPKH, specific mutli-sig scheme, Lightning hold invoice vs regular invoice, etc), which are typically not what an end user specifies and are aware of, will be the responsibility of the trade engines.

Bitcoin obligation tag values are prefixed with `ob-bitcoin`. For now only 2 further specific tags can be specified - for on-chain and through Lightning Network.

| `ob-bitcoin` prefix | description                    |
| ------------------- | ------------------------------ |
| `onchain`           | Bitcoin on-chain               |
| `lightning`         | Bitcoin thru Lightning Network |

Fiat obligation tag values are prefixed with `ob-fiat`. This is followed by the [ISO 4217 currency codes](https://www.iso.org/iso-4217-currency-codes.html), separated by a `-`, and then the payment method codes as found in Bisq's [PaymentMethod.java](https://github.com/bisq-network/bisq/blob/4f1f6898b8b02dd4ad67042fc814c9e05285f8a8/core/src/main/java/bisq/core/payment/payload/PaymentMethod.java), separated also by a `-`. This is then converted all to lowercase for consistency. Not all currency/payment method combinations might actually be usable in reality, but the protocol nevertheless support the ability for a client to express and query any combinations that can be specified.

| `ob-fiat` prefix examples | description                                           |
| ------------------------- | ----------------------------------------------------- |
| `eur-revolut`             | Euros thru Revolut                                    |
| `usd-cashapp`            | US Dollars thru Cash App                              |
| `cny-alipay`             | Chinese Remembi thru AliPay                           |
| `aud-wechatpay`          | Australian Dollars thru WeChat Pay (probably invalid) |

`ob-custom` prefix shall be a catch all, where any additional appendix afterwards is trade engine specific. Further standardization effort might start including them into the n3xB specification if there are wide adoption and usage.

### Trade Engine

Tag name of `#n` will be used to specify trade engines supported. Multiple engine tags can be specified. However for each engine specified, all other parameters in the Maker Order Note must also be true and be supported. If this is not possible, client should create multiple Maker Order Notes for engines it supports, but that have different sets of Maker Order parameters. A client shall update, or remove, Order Notes that differs in engine and supported parameters, but that pertains to the same set of liquidity, once the liquidity have been taken by a counter-party.

Trade Engine implementations should seek to avoid engine tag value conflicts, including making pull requests to n3xB to list and reserve engine tag values.

### Trade Status

Tag name of #s will be used to specify the status of the trade for the order. Maker should strive to keep all status in sync if there are multiple Maker Order Notes for the same lot of liquidity. Trade status is an open-ended field, with several mandatory statuses that client implementations should support, and several optional statuses. Trade engine are also free to override the value of the status field provided that it does not impact the operation of mandatory statues.

| Trade statuses | description                                                                 |
| -------------- | --------------------------------------------------------------------------- |
| `open`         | Order is open for Takers to take                                            |
| `pending`      | Order is pending another Taker, but can potentially return to `open` status |
| `locked`       | Order is 'locked' and will not become available again                       |
| `failed`       | Order have been locked but trade failed. This is an optional status         |
| `completed`    | Order have been locked and the trade completed. This is an optional status  |

### General Trade Details

General trade details should be specified as tag values under tag name `#p`. Primarily this is to help with querying and filtering. Many of the specifics in relation to these parameters shall be defined in specific Trade Engines until further standardization occurs.

| Trade parameters            | description                                  |
| --------------------------- | -------------------------------------------- |
| `maker-has-reputation`      | Maker has reputation information             |
| `taker-reputation-required` | Taker needs minimum reputation to take order |
| `bonds-required`            | Bonds are required                           |
| `trusted-escrow`            | Trade is escrowed by a trusted entity        |
| `trustless-escrowed`        | Trade is escrowed in a trustless manner      |
| `trusted-arbitration`       | Trade is arbitrated by a trusted entity      |
| `accepts-partial-take`      | Taking of partial amount is accepted         |

If `accepts-partial-take` is set, the Maker Order Note is to be replaced with the amounts adjusted instead of being deleted after a partial trade is considered [*locked*](/specs/architecture/architecture.md#locking-of-a-trade). Also see the later sections on [**Invalidation**](#invalidation), [**Updates**](#updates) and [**Expiry**](#expiry) for further details.

A trade can time-out after being taken if the trade have not completed within the time-out value. This is typically seen in on-chain timelock arbitration scripts, and Lightning hold invoices. Tags can help query for time-out attributes, with predefined values as follows. Note that with either `trade-times-out-4-days` and `trade-times-out-24-hours`, `trade-times-out` is considered the generic case and should also be specified. If only `trade-times-out` is specified, a non-standard trade engine specific timeout is assume applied.

| `trade-times-out` prefix | description                                                 |
| ------------------------ | ----------------------------------------------------------- |
| `4-days`                 | Times out after 4 days, similar to Bisq                     |
| `24-hours`               | Times out after 24 hours, limited by Lightning hold invoice |
| `none`                   | Trade does not timeout                                      |

## Content JSON
```
{
  ...
  "content": {
    "maker_obligation": {
      "amount": <amount in integer. Sats if Bitcoin. Minor unit is not supported for fiat>
      "amount_min": <minimum amount in integer. Use in conjunction with tag `accepts-partial_take`. Omit if n/a>
    }

    "taker_obligation: {
      "limit_rate": <limit rate of taker currency / maker currency, in 64 bit double. Omit if n/a>
      "market_offset_pct": <percentage offset from market price for the trade pair, in 64 bit double. Omit if n/a>
      "market_oracles": <array of URL of market oracles / oracle aggregation algorithm to fetch a market price. Omit if n/a>
    }

    "trade_details: {
      "maker_bond_pct": <numerator of percentage out of 100 of the bond maker needs to pay as fraction of trade amount, in integer. Use in conjunction with tag `bonds-required`. Omit if n/a>
      "taker_bond_pct": <numerator of percentage out of 100 of the bond taker needs to pay as fraction of trade amount, in integer.  Use in conjunction with tag `bonds-required`. Omit if n/a>
      "trade_timeout": <minutes trade needs to be completed by, used in conjunction with tag `trade-times-out`, in integer. Omit if n/a>
    }

    "trade_engine_specifics: {
      "type": <trade engine specifics identifier, as string>
      ... trade engine specific JSON fields ...
    }

    "pow_difficulty": <minimum difficulty for a NIP-13 note to take the order, in integer>
  }
  ...
}
```

### Specifying the Trade Amount

The `amount` is the maximum amount to be traded. `amount_min` is the minimum amount for a Taker to take, and only applicable if tag `#p` has `accepts-partial_take` specified. If a Taker have taken less than the full `amount`, and the remaining amount is greater than `amount_min`, the Maker should replace all applicable Maker Order Notes and adjust the amounts accordingly.

### Specifying Trade Exchange Rate

There are two ways for the Maker Order Note to specify an exchange rate, by providing a hard `limit_rate` that a taker must use to take the order, or to allow for a `market_offset_pct` from a market rate.

If `market_offset_pct` is provided, the Maker must also provide a list of approved `market_oracles` that a taker can use to query for market exchange rate where the offset will be based off of. It is ultimately up to the Maker to respond to the Taker's Take message on whether the final determined exchange rate is acceptable before going into the Trade Phase with the Taker.

### Trade Engine Specifics

A section is left for details that would be specific to a particulate Trade Engine implementation. Trade Engine implementation should be careful to not reveal any personally identifiable or secret information inside a Taker Order Note, as it is plaintext and widely distributed.

## Order Types

Numerous order types should be possible with the protocol as defined, namely

### Limit Order (Maker)

Maker creates an order at a certain limit price to wait for a Taker to later take it with a Market Order.

### Market Order (Taker)

Taker buys from any existing order on the market. Usually buying at the lowest price available, or selling at the highest price available.

### Market Offset Order (Maker)

Maker creates an order to trade at an offset from the market price of when a Taker takes the order.

### Market Offset Order with Limit (Maker)

Maker creates an order to trade at an offset from the market price of when a Taker takes the order, but with an upper limit price for a buy order and a lower limit price for a sell order

Caution that market offset orders can be open to Takers cherry picking from a price dataset. They can potentially select the most advantageous price between the time the order was made and when the order gets taken, with no ability for the Maker to detect when the order was actually taken and thus which price is the agreed market price. Even some sort of open timestamping technique will not help as the Taker can always repeatedly timestamp trades and not broadcast it until an advantageous price disparity exists, and only then will they broadcast the trade to capture a guaranteed spread. With all the above said, this isn't that much different from a Taker taking a limit order from a low liquidity exchange only when there is a spread against the market price at high liquidity exchanges. As liquidity improves, there will be little difference between a market offset order and that of a market order.

## Reliability

To ensure discoverability and robustness before potentially entering into a Trade Phase, Maker should make sure to have a Relay List Metadata Note ([NIP-65](https://github.com/nostr-protocol/nips/blob/master/65.md)) for every Nostr relay it is publishing a Maker Order Note to. 

## Updates

Maker Order Notes shall be updatable via [NIP-16 Replaceable Events](https://github.com/nostr-protocol/nips/blob/master/16.md). This is especially important if `partial_take` is specified as the amounts needs to be updated as liquidity is taken. Clients should take care of any potential race conditions, including scenarios where Taker decide to take an order the Maker considers expired and/or invalid, by sending a Peer Message. Client should at least verify the Trade-UUID and also the trade details before determining whether to accept, initiate the trade and take the Order Notes off the relays.

## Invalidation

Once a Trade have been considered [locked](/specs/architecture/architecture.md#locking-of-a-trade) or cancelled, Maker Order Notes for the Trade (same Trade-UUID) can be deleted via [NIP-09 Event Deletion](https://github.com/nostr-protocol/nips/blob/master/09.md). This is to improve privacy and to reduce the storage burden on relays. One potential reason for not deleting an order would be to preserve it potentially for reputation purposes, this however would be outside the scope of the n3xB protocol and would be trade engine specific.

## Expiry

Maker should enforce the setting of [NIP-40 Expiry Timestamp](https://github.com/nostr-protocol/nips/blob/master/40.md) so Maker Order Notes will not linger forever in a relay if the Maker somehow fails at some point to continue to keeps the Order Note maintained. Exact duration of this is implementation dependent. From a naive guestimation, there shouldn't be any reasons for an Order Note to be outstanding for more than 30 days. More volatile markets might demand this time-window be further constrained. Clients can potentially replay an Order Note upon expiry after notifying the user for intervention.

## Privacy

For strongest privacy, its best if clients creates a new pubkey for every single Trade (unique Trade-UUID) created. Pubkeys can be part of an Hierarchical Deterministic key tree. This might however impair the ability to implement a reputation system. Exact implementation and where the trade-off should be drawn shall be up to the specific Trade Engine implementation.

## Proof of Work

Proof of Work should be generated as according to [NIP-13](https://github.com/nostr-protocol/nips/blob/master/13.md). The difficulty should be at least 4 bits higher than what is asked of the Takers in the `pow_difficulty` field. The difficulty that Takers will demand as a minimum difficulty in their query will need to be established out of band, and not part of the scope of the n3xB specification.