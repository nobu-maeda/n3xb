# Attack Vectors
This is a compendium of attack vectors brainstormed, along with potential strategies available or chosen for mitigation.

## Maker Offer SPAM
### Attack
Attacking Makers SPAMs the network with large amount of offers, potentially for fraudulent purposes or as a DoS attack.

### Mitigation
Attack at the Nostr layer is out of the scope of the DEX design. In terms of Maker SPAM as a way to conduct fraud or as a fishing attack on Takers, DEX should be design so that client only reveals information or even payment only after the Maker has proven sufficient stake has been burned or committed. For example to ensure Proof of Work have been conducted before bond commitment, and bond commitment has been done before payment.

## Taker SPAM & PII Fishing
### Attack
Attacking Taker SPAMs Makers by responding to a large number of offers, potentially for fraudulent purposes or to dry up liquidity.

### Mitigation
DEX should be designed so that client only reveals information, or conducts any payment, only when the Taker has proven sufficient stake has been burned or committed. For example to ensure Proof of Work have been conducted before bond commitment, and bond commitment has been done before payment/information, etc

To prevent liquidity draining attacks, offers should not be removed from the market until the Taker has committed some sort of stake, like a bond. Maker should be allowed to service multiple takers for the same offer until a taker successfully committed a bond. Trade mechanism should make it safe so that a trade can still be cancelled with no penalty (aside from wasted PoW computation) from either side up until that point.

## Identity Doxxing
### Attack
Maker & taker pubkey in plaintext can encourage analysis and doxxing. Also Maker IP might be visible to either the relays or to the public.

### Mitigation
There is probably no good way to hide a Maker's pubkey, especially if a Taker is to be able to respond to the Maker to take the offer. Taker's pubkey can be hidden if all of the taker/maker interactions are conducted only through Encrypted DMs with NIP-24 or equivalent. Client can potentially make sure there is no pubkey reuse between trades. However this can get in the way if a a trader is trying to build up a reputation. If a reputation system is to be built, Hierarchial Deterministic key structure should be considered to be adopted, along with potentially the use of Zero Knowledge Proofs.

With all this said however, privacy might have to be degraded significantly regardless if push notification is to be employed. As ultimately phones and push providers will know who the ultimate identity of most mobile devices are. This might be a design trade off that has to be made with no good win-win outcomes. See a similar [issue](https://github.com/bisq-network/bisq/issues/2441) discussion from Bisq which do have push notifications implemented discussing such privacy trade-offs.

## Arbitrator Trust
### Attack
A level of trust is unavoidable in the 3rd party arbitrator. The arbitrator can potentially take the traders' bonds, or even the full amount in some cases, depending on the bond scheme employed.

### Mitigation
The currently deployed arbitration schemes on the market do require limited trust on the arbitrators. In the case of Bisq if the trade does not complete in 4 days, the arbitrator gets essentially full control of all on-chain funds, bonds and the traded amount alike. Trades that goes into arbitration is a low percentage however so this is not a huge problem. Moreover in the case of Bisq, the arbitration role is overseen by the Bisq DAO, which is an organization that has pretty good decentralization, checks, and balances.

In the case of Robosats, the arbitrator there is fully trusted with the bond amount from the startup up until the HTLC expires. Once a trade goes into arbitration, it does seem that they also have at least the ability to unilaterally release Bitcoins to the buyer if wanted. There seems to be no way however for the arbitrator to ever get to the actual Bitcoin funds at any point. The arbitrator is a fully centralized and trusted role in this case.

The ideal endpoint for an arbitrator role in the N3XB ecosystem is a market of reputable and broadly trusted 3rd parties that traders' can select and choose as the arbitrator for trades. The marketplace should be a good check against dishonest arbitrators, and will reward arbitrators with either a good track record or an innovative governance structures like a DAO. There also needs to be incentives (per trade / per arbitration / subscription based arbitration fee) for arbitrators so that there will be an actual vibrant market where incentives between arbitrators and traders as a broad group aligns.

## Arbitrator Doxxing
### Attack
Arbitrators can be easily doxxed and be an attack vector for the ecosystem.

### Mitigation
This is unfortunately very hard to avoid as the arbitrator is a trusted role that relies on a somewhat public reputation for the trust to occur. A diverse free market of arbitrators hopefully will allow for those with the best balance between trustworthiness and censorship resistance to thrive. There also needs to be incentives (per trade / per arbitration / subscription based arbitration fee) for arbitrators so that there will be an actual vibrant market where incentives between arbitrators and traders as a broad group aligns.

## Relay Centralization and Shutdown
### Attack
There is a level of risk for Relay centralization and risk where Relays be under attack and either be shutdown, or be pressured to censor trade traffic

### Mitigation
Clients should always default to, if not enforce, a minimum number of relays being used, this minimizes relay centralization risk. Incentivizing relays to host orders and facilitate trades also helps to develop a healthy market of relays that will not censor trade traffic. 

## Fiat Chargeback Risk
### Attack
There is a risk where buyer chargebacks fiat payment, which does not settle immediately, against the Bitcoin they received as part of the trade, causing the seller's payout to be clawed back. 

### Mitigation
There are potentially 2 mitigations to this:
1. Keeping the on-chain payout and bond to be released only after a sufficient time have passed, where the chargeback risk is sufficiently low
2. Building a reputation system for traders so those with more of a good track record of successful non-fradulent trades will be rewarded.