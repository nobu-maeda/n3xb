# Philosophy of N3XB

## What is a Marketplace?

A marketplace is a venue that maximizes liquidity and minimizes friction in matching coincidences of wants from 2 or more parties whom are looking to transact. Often a marketplace also provide a mean to increase confidence for parties to transact and minimize the probability and severity of malice that might otherwise occur.

An example of a marketplace is the groceries store. A groceries store becomes a gathering of grocery inventory in an easily identifiable location for individuals that wishes to exchange money for grocery goods. The ease of identification for a specific type of good helps increase liquidity and attracts potential customers. A groceries store would also sort grocery items in a manner so that discovery of specific items that matches a customer's requirement can easily be found, reducing friction to transaction. Last but not least, confidence of transaction is often established through reputation of the grocery store or chain, but also via the expectation that the local legal system would enforce fair exchange and fraud prevention through a trusted monopoly on violence.

Unfortunately, the protection by the legal system can sometimes be a double edge sword for some marketplaces. The monopoly on violence that a state has can become a force for censorship. States also often demand the removal of anonymity before allowing individuals to transact. This becomes especially problematic with marketplaces that might facilitate what might be perceived as competition to a state's monopoly in the control of it's population. How can we create marketplaces that can provide the assurances that market participants usually expects, but not only be independent from the state, but be resistant to their potential disruptions?

## Bitcoin Fidelity Bonds

A fidelity bond is a form of insurance to incentivize a contractual obligation to actually be carried out. The bond is posted by a party who is agreeing to an obligation. If the obligation is fulfilled, the bond is returned to the party. However if the obligation is not fulfilled, the bond can be slashed, or sometimes used to compensate the counter-party. Fidelity bonds can be required and posted by both parties in a transaction to deter either side from breaking the contract.

There have been several ways fidelity bonds have been implemented using various Bitcoin script.

- 2 of 3 multisig between the 2 transacting parties and an arbitrator - was implemented by Bisq at one point
- 2 of 2 multisig between the 2 transacting parties inside a HTLC where the arbitrator can redeem if the 2 parties does not come into consensus on their own - current Bisq bond mechanism.
- 2 of 2 multisig with mutually assured destruction, where the bond is inaccessible if the 2 parties cannot come into consensus to unlock the bond - theoretical, no such implementation known
- Hodl invoice from each party to the arbitrator via Lightning Network. Where the arbitrator can unilaterally redeem if see fit within the time-lock time-window which the transaction is expected to complete - current Robosat bond mechanism

Fidelity bonds along with a semi-trusted 3rd party arbitrator have been used successfully in various platforms to incentivize fulfillment of contractual obligations without the involvement of any legal system of any states. For the case of Robosat, its the Robosat team acting in this trusted role. In the case of Bisq, the 3rd party arbitrator is the Bisq DAO, which minimizes the trust required on a specific individual entity to a more limited degree.

## Decentralization

Decentralization is often used as a defense against attacks and coercion by the state. It increases the cost and surface area of attack, while decreasing the damage to an ecosystem once an attack is successfully conducted. The ultimate decentralization would be a true peer to peer network. However such networks are usually costly to implement, comes with high performance penalty, and is usually completely incompatible with challenging mobile operating environments that most end user uses. True peer to peer system often become relegated to usage only by competent enthusiasts with always-on servers or desktop computers. And even so for these users, the user experience is often poor at best.

To increase usability for clients with resource and performance constraint like mobile environments, a relay is often employed. Notable examples of this includes Bitcoin Full-nodes vs Bitcoin SPV light clients, where the full-node acts as a relay for mobile devices which acts as a light client. While Nostr is an explicit relay vs client topology to begin with. All clients are dependent on relays, similar to centralized client-server models, except that multiple redundant relays is supported first class to achieve a degree of decentralization and censorship resistance.

## Interoperability

Many social media and marketplaces alike employs proprietary and closed datasets and protocols. They are not designed from the ground up for alternative projects and implementations to spring up. At best a monolithic implementation is created, and a limited external API, entirely dependent and controlled by the main project, is opened up. Between usability, platform risks, control and ownership, these APIs are rarely used. Those who chooses to use them sometimes gets 'rug-pulled' with the API unilaterally change, or be closed off without notice. This is great if the objective is for these projects to defend their network effects, but this comes at the detriment of diverse experimentation, innovation and any significant network effect to be bootstrapped at all.

An open protocol designed for interoperability and multiple compatible implementations allow multiple projects and effort to contribute towards a common network effect. This type of architecture usually are not conducive to allow a monopoly to form and enable monetization of said network effect. In-spite of usually lacking investment from return seeking VCs, they often still innovate in rapid pace once product market fit is achieved. Best examples includes Bitcoin and Nostr, where the value from the network effect is captured by the user, not the founders. Incentivizing every user to contribute in building the network. The barrier to entry in creating an alternative client or experiment with protocol enhancements is also greatly lowered, as anyone can quickly tap into the network effect and innovate.

## Abstraction

Instead of building out a highly specific, rigid, singular purpose protocol and architecture. Another alternative is to start with an extremely simple and abstract base protocol, and build more specificity and applications on top of it. Nostr is a great example, where the initial specification only specified an idea. NIP-001 only specified the initial event object along with several generic request and subscription commands. Allowing subsequent NIP to flesh out and adapt the base protocol for specific applications. This allowed independent projects the room to use the generic abstract base protocol as they see fit. As a result we have seen P2P chess applications built on top of Nostr, and potentially the N3X protocol as we are describing, amongst other use-cases.

For N3X and N3XB, the idea is to create a base protocol regarding what the most abstract generic marketplace transaction might look like (N3X), and then build additional specificity on-top. For example, specificity regarding Bitcoin <=> fiat trading (N3XB), versus a marketplace for extra-legal contract and arbitration, or an eBay type marketplace for goods, etc. 