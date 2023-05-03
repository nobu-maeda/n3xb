# n3xB Peer Messaging
*See [Architecture](/specs/architecture/architecture.md) for where Peer Messaging fits in the overall n3xB protocol flow*

n3xB standardizes how peer messaging can be conducted. Its is not mandatory for Trade Engines to use n3xB messaging schemes, but it is available. For n3xB Take Order Messages and Trade Response Messages, this is the assumed default.

## Carrier

Until a more privacy preserving direct messaging method is widely available (see [Privacy Improvement](#privacy-improvement) below), Nostr Encrypted Direct Messages [NIP-04](https://github.com/nostr-protocol/nips/blob/master/04.md) is to be used to facilitate peer messaging.

To enhance robustness of peer messaging, clients should ensure that at least several of the relays listed by the counterparty's ([NIP-65](https://github.com/nostr-protocol/nips/blob/master/65.md)) Relay List Metadata is added as backup relays in the client's own relay list.

## Identification

A n3xB Peer Message can be defined by the tag value `n3xb` for tag name `#d`, and the tag value `peer-message` for the tag name `#k`.

## Push Notifications

Often times manual user actions, or a wake-up of a mobile client, is required to make sure the trade continues. Mobile push notification will be very valuable in these cases. As it is Nostr does not have any support for generating mobile push notifications. A NIP proposal have been made for mobile push [here](https://github.com/nostr-protocol/nips/issues/257). This is not required to make n3xB possible, but it would be a huge improvement to the overall experience and usability if the capability is available. n3xB Peer Messaging shall adopt Nostr push notifications once available.

## Pre-Defined Tags

Aside from the mandatory `#p` tag as specified in [NIP-04](https://github.com/nostr-protocol/nips/blob/master/04.md). Tags are not used whatsoever, as they are in plain text and reduces secrecy and privacy for the trade and participants.

## Content JSON

Note that this is the plaintext of what the decrypted content would be. The actual content that goes into a message would be encrypted as according to [NIP-04](https://github.com/nostr-protocol/nips/blob/master/04.md).
```
{
  ...
  "content": {
    "peer_message_id": <32-bytes lowercase hex id of the Peer Message being responded to. Omit if not applicable>
    "maker_order_note_id": <32-bytes lowercase hex id of the Maker Order Note this message corresponds to>
    "trade_uuid": <32-bytes lowercase hex Trade-UUID this message corresponds to>
    "message_type": <string code of message type, specific to n3xB or a Trade Engine>
    "message": <arbitrary JSON depending on the message type>
  }
  ...
}
```

### Message Types

If a Trade Engine chooses to use the n3xB Peer Messaging scheme, it can define its own Message Types and Message JSON formats. The following are Message Type string codes defined by n3xB, and a link to the corresponding specification of the Message JSON format itself.

| `message_type` string code | Message specification |
| -------------------------- | --------------------- |
| n3xB-take-order | [Take Order Message](/specs/taker-message/taker-message.md) |
| n3xB-trade-response | [Trade Response Message](/specs/trade-response/trade-response.md) |

## Proof of Work

Proof of Work should be generated as according to [NIP-13](https://github.com/nostr-protocol/nips/blob/master/13.md). The difficulty for all messages pertaining to a particular Maker Order Note needs to meet the minimum specified in the `pow_difficulty` field. Otherwise all parties involved has the right to not respond to any peer messages pertaining to that Order.

## Privacy Improvement

[NIP-24 Private, Encrypted Direct Messaging](https://github.com/jeffthibault/nips/blob/private-messages-v2/24.md) is also a work in progress to make Nostr direct messaging more anonymous. However if push notification is to be employed by relays, there is no way to obfuscate who the actual recipient is. This might be a privacy trade-off that a client implementation must make when NIP-24 actually becomes available.