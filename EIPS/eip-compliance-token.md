---
eip: TBD
title: YES Compliance Token 
author: Tyson Malchow (@tysonmalchow), Michael Dunworth (@MikeD123)
discussions-to: https://github.com/sendwyre/yes-compliance-token/issues/1
status: Draft
type: ERC
category: ERC
created: 2018-09-08
requires: ERC721
---

<!--You can leave these HTML comments in your merged EIP and delete the visible duplicate text guides, they will not appear and may be helpful to refer to if you edit it again. This is the suggested template for new EIPs. Note that an EIP number will be assigned by an editor. When opening a pull request to submit your EIP, please use an abbreviated title in the filename, `eip-draft_title_abbrev.md`. The title should be 44 characters or less.-->

## Simple Summary
<!--"If you can't explain it simply, you don't understand it well enough." Provide a simplified and layman-accessible explanation of the EIP.-->
We propose a flexible, lightweight on-chain compliance ecosystem. It provides mechanisms 
for _validators_ to attest to and record specific compliance claims on behalf of _end-users_, which in turn may
be used by _integrators_ seeking to know if some specific compliance requirement has been met by the owner of a
particular address. We define these interactions in pursuit of a safe, legally compliant blockchain 
environment.

Think "Twitter Verified Blue Checkmark," but for on-chain compliance.

## Abstract
<!--A short (~200 word) description of the technical issue being addressed.-->
Compliance attestations are distributed within an ERC721-compliant non-fungible token. This interface allows end-users to 
independently and securely associate (or de-associate) their compliance status with one or many Ethereum addresses using
an already established paradigm: any address which holds one of these such tokens has a legally identified party behind it
with any number of specific queryable compliance attestations.

Claims are attested to by _validators_ (who may therefore also mint tokens tied to these attestations). These are 
organizations which provide specific
verifications for end-users; ultimately, the _Ecosystem Owner_ (original contract owner) backs the claims distributed on the 
network.
The specific claims allowed are defined authoritatively by this document and may be country-specific.

When an end-user attempts to interact with some 3rd-party financial service or application which supports this protocol 
(an _integrator_), their application can quickly query the compliance status of the end-user on the blockchain. A  
query API is defined so that the integrator may contextualize their needs and discover if a particular address meets
a specific compliance requirement.

One deployed contract of this token encompasses a single ecosystem of recognized validators in the space. This
ensures that any partner attempting to query compliance status need not ask many validators separately, but rather
query them all through a single token. The Ecosystem Owner ultimately controls the list of authorized validators.

## Motivation
<!--The motivation is critical for EIPs that want to change the Ethereum protocol. It should clearly explain why the existing protocol specification is inadequate to address the problem that the EIP solves. EIP submissions without sufficient motivation may be rejected outright.-->

The problem of compliance across the cryptocurrency landscape is well-known. Traditional financial institutions have rigorous
checks and balances as well as extensive reporting to maintain legal compatibility (and reduce assumed risk). Crypto, on the
 other hand, has so far been lacking much of the formalized oversight necessary to spur such regulation.

When transactions are deemed fradulent
or otherwise illegal (money laundering, etc), mechanisms must be in place to allow authorities to locate the involved parties.
Furthermore, some degree of transaction data must be regularly sent to authorities to assist them in locating such behavior.
These requirements blossom into a fragmented landscape of rules and restrictions that vary by country and industry. 
Combined with the relative difficulty in
proving that a digital actor is a who they claim to be, this represents a significant burden of effort for any organization who wishes 
to innovate in the financial world.

As cryptocurrency innovation accelerates, straightforward solutions to this compliance maze will become increasingly important.
Developers (both dApp and otherwise) need the ability to achieve a confident degree of compliance, and together 
should not have to collectively duplicate the effort required to achieve this. 

## Specification
<!--The technical specification should describe the syntax and semantics of any new feature. The specification should be detailed enough to allow competing, interoperable implementations for any of the current Ethereum platforms (go-ethereum, parity, cpp-ethereum, ethereumj, ethereumjs, and [others](https://github.com/ethereum/wiki/wiki/Clients)).-->

See the YES interface definition 
[here](https://github.com/sendwyre/yes-compliance-token/blob/master/contracts/yes/YesComplianceTokenV1.sol).

### Token Operation

The YES system distinguishes between two high-level concepts, _tokens_ and _entities_. An entity is a
 business or individual who maintains
some verified compliance status (an identified party). An entity may have many tokens, though one token will 
always have a single entity. All
tokens can be used interchangeably as proof-of-compliance. This proof represents
a formally (and rigorously) identified 
entity - business or individual - who has met specific compliance requirements, as attested to by one or more validators
(and therefore transitively by the Ecosystem Owner). The actual identity of the entity is protected by the validators
and is not on the blockchain.

A validator will interact with the end-user in order to validate the identity they are claiming as well as a
set of compliance attestation. They will request documentation, signatures, proof, etc. 
pursuant to the corresponding requirements.
After this process
concludes, the validator assigns the corresponding on-chain YES marks. These marks are attached to the end-users' 
entity and attest to the outcome 
of specific predefined degrees of compliance. They are always boolean; present, or not, without degree. 

> TODO maybe: use [DID](https://w3c-ccg.github.io/did-spec/) identifiers instead of (some of) the following?

Entity identifiers
are validator-generated and their scheme is not mandated by this document - they should be random and unique, and generation 
stability between validators is unnecessary. To alleviate duplicated effort on the network, an end-user having gone
through verification may choose to present their existing tokens to skip such effort elsewhere. It may be necessary for
validators to develop additional off-chain channels to mediate against users acquiring multiple differing entity IDs, if
such acquisition was harmful to the ecosystem. (This helps slightly to protect end-user privacy and also avoids a tedious
rabbithole of difficult naming formalities).

The validator may mint and send one or more new tokens to the end-user, which are linked to the entity that has their YES 
attestations. Once they possess a token, they're then free to create more tokens
and distribute them to any other addresses they would like to identify with (via the increasingly well-known ERC721).

Attempting to move a token to an address that already possesses a token linking it to a different address is forbidden;
one address can be associated with at most a single entity (but one address could own many tokens linked to that single
entity). 

Integrators query the token to discover the compliance status of any particular user. 
(via `isYes` 
or `requireYes`) without specifying an explicit set of allowable validators implies any validator is considered.

***Ecosystem Validators*** are entities with the `129. Ecosystem Validator` YES mark. They are the parties who are
interacting with the end-user to process their proof of identity, and they have reporting (and liability?)
agreements established with the Ecosystem Owner. This gives that entity the ability to create new YES attestations 
for any entity. (Though, only the Ecosystem Owner has the ability to mark entities with with `129`) 
For auditability, all YES attestations marked carry the set of validator ID(s) which validated them. 

***Entity Locking:*** An entity may be locked. This may be invoked by any validator against any entity and suspends that entity from 
receiving any compliance approval. This is intended to function in response to any type of compliance flag that throws
the overall validity of the identity into question - as in the case of possible identity theft or fraud. However, in 
the happy case where the alert is cleared, we save a lot of fees and token-holder effort by flipping one bit instead
of needing to re-issue possibly many coins to many addresses. 

***Token Finalization:*** Permissible to any token holder, this prevents the token from ever being moved. After being
finalized, it may only
get burned. In systems which have no use to move the tokens around once they reach their target, this adds a small 
degree of safety to prevent tokens from getting erroneously (or maliciously) moved.

***Limited Tokens:*** _(Experimental)_: For security, we also provide a _limited_ token. This is a token which cannot be used 
to mint others. In most cases, a token permits its holder to mint new tokens; a limited token cannot. This
is useful as a division of privilege to more strongly safeguard the production new tokens. Integrators who do not support
such a distinction can simply ignore limited tokens.

***Ecosystem Proxying:*** _(Future/Maybe)_: Token 'proxying' to enable the Ecosystem Owner to delegate token recognition to 
a specific whitelist of other YES-compatible tokens so that other top-level validators could maintain their own networks 
of partners, yet remain queryable through a single token interface.

### Regulatory Mechanics

The usage of these tokens ***does not*** relinquish the need for businesses themselves to always remain compliant pursuant
to the regulatory authorities in their industry. 
However,
such businesses may establish an agreement with the Ecosystem Owner in order to help offload portions of the burden. In the case
 of applications which manage user-controlled funds, they are not money services businesses,
as they do not custodialize their customers' funds. Therefore, they are not required to maintain these licenses. 

However, this passes the buck of remaining compliant to the end-user. If someone wants to trade money with a friend, with an 
exchange, with a business, anywhere - they want to know they are properly handing over the liability of 
what happens with those funds. This ensures there are proper channels for legal recourse when required, as well as
following all ongoing reporting requirements.

This proof is provided as a chain of liable, well-identified parties. However, privacy should be maintained as best 
as possible. One should not need to know the details of the person with whom they're interacting; they should merely have 
trustworthy evidence that the counterparty is capable of bearing responsibility for their actions. This comes as a guarantee
from a liable party, the Ecosystem Owner.
 
All ongoing reporting and fraud prevention is then the responsibility of the Ecosystem Owner, which may have 
delegated it further (through agreements) with other parties (validators, usually). An integrator needn't treat
tokens differently which have been validated by differing parties (unless they want to).

In the cases of businesses which custodialize their customers' funds, they must (at the least, in the US) be licensed 
money service businesses. They must follow all relevant reporting requirements as dictated by their business practices
in their own locality, and this protocol does nothing to alleviate those requirements. However, this protocol does
offer an avenue to offload portions of this relatively rigorous process to a set of the validators in the 
ecosystem. By honoring the end-result from the validation process (provided by a validator), they gain confident access to both
their own (validated) end-users as well as any other user ever validated in the ecosystem.

### YES Marks: Compliance Attestation Codes

A YES mark (8-bit unsigned integer) is a code which, by convention, maps to a specific compliance attestation as given 
below. The tiers of permissions required for the outlined mechanics are also encapsulated within this structure via 
codes 128 and 129. All codes are granted in country-specific contexts, except for the owner code 128 (which has a country

Many codes have a generalized meaning along with a country-specific definition, which is contextualized by the local
agencies and regulations of a specific country. Regional distinctions are not supported; attestations must be country-wide, 
though it is the responsibility of any particular _integrator_ to ensure their customers are not accessing a service from
a more restrictive or unsupported region within a country.

    Special - country code 0:
    128. (special) Ecosystem Owner - only one. can attest all marks except `128`
    129. (special) Ecosystem Validator - can attest all marks except `128`, `129`

#### Generalized (Country-Agnostic) Definitions:

These define the context and minimum requirements of each attestation in a 
general, country-agnostic fashion.

Individual:

    (most significant bit is 0)
    1. Tier 1 Individual
       a. Identification
       b. Evidence for Possession of Valid Government ID(s)
       c. AML Compliant
    2. Tier 2 Individual
       a. Tier 1
       b. Video Evidence: Intention to attest (by whom), facial/attribute matching
    10. Accredited Investor 

Business/Organization:

    (most significant bit is 1)
    130. Tier 1 Corporation
      a. Public Company Registration Information
      b. Board Member Verification (all to Tier 1 Individual)
    _131_. ??? GDPR [*] - Generalized validation/recognition of following GDPR practices
    200. Money Services Business
    _201_. ??? (generalized name for ATS license holder?)

#### Per-Country

The definition per country matches the structure of a generalized definition above, further
articulating its requirements. Or, it may define a new code which has no generalized equivalent.

USA (840):

    1. Tier 1 Compliance:
      a. Name, DOB, SSN, Address (CIP1)
      b. Scan of drivers license OR passport
      c. AML: Copy of a bank statement
    2. Tier 2 Compliance - 5 second video with specific verbal authorization, 
       (including verifier organization's name)
    10. Accredited Investor: Follows SEC-defined regulations - Rule 501 of Regulation D
    130. US S or C Corp. (Tier 1) 
      a. Articles of Incorporation
      b. Cap table
    200. FinCEN Registered MSB - https://www.fincen.gov/msb-registrant-search
    201. SEC Registered ATS License Holder


## Rationale
<!--The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made. It should describe alternate designs that were considered and related work, e.g. how the feature is supported in other languages. The rationale may also provide evidence of consensus within the community, and should discuss important objections or concerns raised during discussion.-->

In drafting this document, our primary goal has been to materialize a direct, simple, clean implementation which solves
the most glaring problems facing us and our partners in the space. We need an effective mechanism to foster compliant 
behavior for innovation both on and off chain. We seek to minimize gas costs and integration friction, while maintaining 
privacy and simplicity.

#### Related Works

In order to better contextualize our approach to other relevant activity in the industry, we present some brief notes on a handful of specific 
standards/projects:

*ERC-725/735 - Identity, Claim Holder*: The ERC725 standard merges identity and key managment into a single concept that allows a stable identity (address)
to maintain a published catalog of keys associated with that identity (and those keys' individual permissions/roles). It gives a person (or more generally,
any _thing_) its own representation on the blockchain.

ERC735 is a closely related standard that defines a set of mechanics for requesting, issuing, and querying a set 
of signed claims. Claims are address-specific signed structures that are one-per-topic-per-issuer, where topics is a (to-be?) standardized 
identifier namespace of claim categories.

Together, they form a managed identity which can accumulate signed claims from many parties.

While this is theoretically similar in concept design, our approach is cheaper in its 
blockchain footprint as there is only a single deployed contract involved, and the on-chain claims information
is recorded at a lower fidelity.

The approaches are not mutually exclusive - if the outcomes of the various 
compliance checks have equivalent mappings atop ERC735, an extension to the implementation could allow a token-holder to link
their entity to an ERC725/735 identity (should they have one). In turn, such an extension would export the YES claims to equivlaent 
ones consumable by ERC735. The ERC725 identity would possess the YES compliance status as any other address through possession of the
token.

*ERC-1056 Lightweight Identity*: This is a much cheaper alternative to ERC725 and is strictly identity, not (necessarily) claims. 
As with ERC725, possession of a YES token would give the identity its registered compliance status and minting powers.

*ERC-1404 Simple Restricted Token Standard, R Token*: These standards serves to define an APIs which enable restrictions on the 
movement of ERC20 tokens. They are compatible with the YES token approach, as any particular regulator implementation could include an
on-chain strategy which leverages the YES query API to make decisions on transfer restrictions.
While compatible in this way, our focus is the codification of the compliance outcomes and the ability to give that information to
interested parties. While the outcomes are the same in some cases, creating codified restrictions at the coin/token level 
addresses a different problem.

*ERC-780*: Lightweight open on-chain datastructure which allows anyone to publish their own claims. Compatible in a similar context
as ERC735, in that any YES attributions could (in-theory) be linked and/or exported to a 780 registry. However, it requires that 
real claim data is placed on the chain, so privacy is sacrificed.

<!-- ## Test Cases
We have test cases covering all the fundamental mechanics of the draft implementation [here](https://github.com/sendwyre/yes-compliance-token/blob/develop/test/yestoken.js). 
-->

## FAQ

#### Does allowing all token-holders weaken their meaning or present a security risk?

Though it is possible for someone to generate an endless supply of tokens, all tokens generated by an particular entity
is bound to the identity of that entity. It's equivalently possible to get new drivers licenses issued for yourself and sell them.
However, you are adopting the risk of what happens with those licenses after they're sold. Similarly, individuals are disincentivized
to distribute their tokens because it opens them up to liability for actions they are not themselves performing. Should legal problems
arise as a result of such a circumstance, the actual identified holder is on the hook to provide an avenue to (legal) justice.

#### Forbidding token movement when target already possesses an identity - DDoS?

In theory, an actor on the network with a valid but fradulently acquired YES identity could create a large supply of their 
own tokens sending them to addresses which are not theirs, thereby blocking all those addresses from accepting their 
actual owners' identity token.

While this is possible, the timing is constrained, and lockability (and centralized control in general) of the token system means
that a validator can revoke or disable the set of tokens in response to such an attack.

#### What are examples of this "validator to validator" information sharing in the real world today? Does it ever happen or is this a new concept?

The reason why companies struggle to find banking partners willing to service them is because banks see them as an MSB. This scares banks as it means theres risk that the company could not be upholding their policy verifying people, and the bank is then at risk for disbursing payments to people they shouldn't be. We went into depth explaining [why](https://blog.sendwyre.com/msbs-must-read-for-fintech-entrepreneurs-crypto-or-non-crypto-5ed3cf8aae40) this is the case, and what we'd learned. 

#### How would validators share information in the event of regulatory intervention?

The scope of this proposal doesn't cover circumstances beyond the entity/validator relationship. At the moment, any information sharing is done through encrypted storage/uploads to the relevant parties. 

#### What does an agreement between two validators look like? Does it protect them from liability?

It varies with language in terms of how much liability is taken on by signing an agreement. There's multiple types of agreements that cover AML/KYC policies being adhered too. Here's language from an example agreement known as a "Currency Purchase Agreement". Common use-case would be large scale OTC (Over-The-Counter) trading.

_Section 4.7 Confidentiality. Each of [TradingCo.] and Counterparty hereby agrees to not disclose, and to otherwise keep confidential, the transactions contemplated hereby, the existence or nature of any relationship between the Parties, the name of the other Party or the fact that the Parties engaged in any transaction (“Confidential Information”), provided, however, that each Party may disclose Confidential Information to its directors, officers, members, employees, agents, affiliates, and professional advisers or to financial institutions providing services to a Party in connection with any applicable anti-money laundering or compliance requirements . If either Party is required by law, rule or regulation, or advised by legal counsel to disclose such information (the “Required Party”), the Required Party will, to the extent legally permissible, provide the other Party (the “Subject Party”) with prompt written notice of such requirement so that such Subject Party may seek an appropriate protective order or waive compliance with this Section 4.7. The Subject Party shall promptly respond to such request in writing by either authorizing the disclosure or advising of its election to seek such a protective order, or, if such Subject Party fails to respond promptly, such disclosure shall be deemed approved. The confidentiality obligations set forth in this Section 4.7 shall survive the termination or expiration of this Agreement.

Essentially it is saying that we respect confidentiality amongst one another, however if any information is required based on legal intervention, or our lawyers saying so, then we'll both be cooperative.

Implementation of this proposal would mean that all these agreements and paperwork could be reduced down to simply signing a message between two validators.

## Implementation
<!--The implementations must be completed before any EIP is given status "Final", but it need not be completed before the EIP is accepted. While there is merit to the approach of reaching consensus on the specification and rationale before writing code, the principle of "rough consensus and running code" is still useful when it comes to resolving many discussions of API details.-->

A draft implementation covering the entirety of this spec is located [here](https://github.com/sendwyre/yes-compliance-token).

## Additional Reading

* Increased government parties and double handling of information - [Link](https://www.gao.gov/assets/680/675400.pdf) 
* Multiple parties for oversight - [Link](https://en.wikipedia.org/wiki/Financial_Stability_Oversight_Council)
* Financial Intelligence Unit - [Link](https://en.wikipedia.org/wiki/Financial_intelligence)
* Regulatory Oversight - [Link](https://www.investing.com/brokers/regulation)
* Securities Market Participants - [Link](https://en.wikipedia.org/wiki/Securities_market_participants_(United_States))
* FINRA example template for AML/CFT - [Link](http://www.finra.org/industry/anti-money-laundering-template-small-firms)
## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).










