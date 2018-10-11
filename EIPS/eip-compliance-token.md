---
eip: TBD
title: YES Compliance Token 
author: Tyson Malchow (@tysonmalchow)
discussions-to: https://github.com/sendwyre/yes-compliance-token
status: Draft
type: ERC
category: ERC
created: 2018-09-08
requires: ERC721
---

<!--You can leave these HTML comments in your merged EIP and delete the visible duplicate text guides, they will not appear and may be helpful to refer to if you edit it again. This is the suggested template for new EIPs. Note that an EIP number will be assigned by an editor. When opening a pull request to submit your EIP, please use an abbreviated title in the filename, `eip-draft_title_abbrev.md`. The title should be 44 characters or less.-->

## Simple Summary
<!--"If you can't explain it simply, you don't understand it well enough." Provide a simplified and layman-accessible explanation of the EIP.-->
We propose a flexible, lightweight on-chain compliance ecosystem. It provides mechanisms for _end-users_ to acquire 
proof-of-compliance tokens, for _validators_ to attest to this proof, and for _integrators_ to query the state of the 
proof. We define these interactions in pursuit of a safe, legally compliant blockchain environment.

Think "Twitter Verified Blue Checkmark," but for on-chain compliance.

## Abstract
<!--A short (~200 word) description of the technical issue being addressed.-->
Compliance attestations are distributed atop an ERC721-compliant non-fungible token. This interface allows end-users to 
independently and securely associate (or de-associate) their compliance status with one or many Ethereum addresses.

These tokens are issued by _validators_. These are organizations which provide specific
verifications for end-users; ultimately, the ecosystem owner (Wyre) backs the attestations distributed on the network. 
The specific attestations allowed are defined authoritatively by this document and may be country-specific.

When an end-user attempts to interact with some 3rd-party financial service or application which supports this protocol 
(an _integrator_), the application can quickly query the compliance status of the end-user on the blockchain. A  
simple query API is defined so that the partner may contextualize their needs and 
acquire a direct answer on the compliance status of a particular address.

One deployed contract of this token encompasses a single ecosystem of recognized validators in the space. This
ensures that any partner attempting to query compliance status need not ask many validators separately, but rather
query them all through a single token. The _ecosystem owner_ (Wyre, in our case) 
ultimately controls the list of authorized validators.

## Motivation
<!--The motivation is critical for EIPs that want to change the Ethereum protocol. It should clearly explain why the existing protocol specification is inadequate to address the problem that the EIP solves. EIP submissions without sufficient motivation may be rejected outright.-->

In drafting this design, our primary goal has been to materialize a direct, simple, clean implementation which solves
the most glaring problems facing us and our partners in the space. We need a way to attach simple attestations
that had predefined definitions in the context of compliance, per-country.

TODO moar

## Specification
<!--The technical specification should describe the syntax and semantics of any new feature. The specification should be detailed enough to allow competing, interoperable implementations for any of the current Ethereum platforms (go-ethereum, parity, cpp-ethereum, ethereumj, ethereumjs, and [others](https://github.com/ethereum/wiki/wiki/Clients)).-->

See the YES interface definition [here](https://github.com/sendwyre/yes-compliance-token/blob/master/contracts/yes/YesComplianceTokenV1.sol).

#### Token Operation

The YES system distinguishes between _tokens_ and _entities_. An entity is a business or individual which maintains
some verified compliance status. An entity may have many tokens. One token will always have a single entity. All
tokens are linked to an identified entity and can be used interchangeably as proof-of-compliance. This proof references
a formally identified 
entity - business or individual - which has met specific compliance requirements, as attested to by one or more validators.

A validator will interact with the end-user in order to validate them - request documentation, etc. After this process
is complete, the validator will assign the corresponding on-chain YES attestation marks. These marks attest to the outcome 
of specific predefined degrees of compliance, and are assigned to their entity. They are always boolean; present, or not,
without degree.

The validator may mint and send a new token to the end-user, which is linked to the entity that has their YES 
attestation marks. Once they possess a token, they're then free to create more tokens 
and distribute them to any other addresses they would like to identify with (via the increasingly well-known ERC721).

Attempting to move a token to an address that already possesses a token linking it to a different address is forbidden;
one address can be associated with at most a single entity (but one address could own many tokens linked to that single
entity).

Integrators query the token to discover the compliance status of any particular user. 
(via `isYes` 
or `requireYes`) without specifying an explicit set of allowable validators implies any validator is considered.

***Validators*** are entities with the `129. Ecosystem Validator` YES mark. They are the parties who are
interacting with the end-user to process their proof of identity, and they have reporting (and liability?)
agreements established with the ecosystem owner. This gives that entity the ability to create new YES attestations 
for any entity. (Though, only the ecosystem owner has the ability to mark entities with with `129`) 
For auditability, all YES attestations marked carry the set of validator ID(s) which validated them. 

***Locking:*** An entity may be locked. This may be invoked by any validator against any entity and suspends that entity from 
receiving any compliance approval. This is intended to function in response to any type of compliance flag that throws
the overall validity of the identity into question - as in the case of possible identity theft or fraud. However, in 
the happy case where the alert is cleared, we save a lot of fees and token-holder effort by flipping one bit instead
of needing to re-issue possibly many coins to many addresses. 

***Limited tokens:*** For security, we also provide a _limited_ token. This is a token which cannot be used to mint others. In most
cases, a token permits its holder to mint new tokens; a limited token cannot. This
is useful as a division of privilege to more strongly safeguard the production new tokens. Systems which do not support
such a distinction can simply ignore limited tokens.

***Token finalization:*** Permissible to any token holder, this will prevent the token from ever being moved. It could only
get burned. In systems which have no use to move the tokens around once they reach their target, this adds a small 
degree of safety for tokens to be erroneously (or maliciously) moved.

***Proxying:*** _TODO/Future/Maybe_: Token 'proxying' to enable the ecosystem owner to delegate token recognition to 
a specific whitelist of other YES-compatible tokens so that other top-level validators could maintain their own networks 
of partners, yet remain queryable through a single token interface.

#### Regulatory Mechanics

The usage of these tokens ***does not*** relinquish the need for businesses themselves to always remain compliant. However,
any business may establish an agreement with Wyre (or another ecosystem) which offloads the KYC/AML requirements to the
ecosystem owner. In the case of applications which manage user-controlled funds, they are not money services businesses,
as they do not custodialize their customers' funds. Therefore, they are not required to maintain these licenses. 

However, this passes the buck of remaining compliant to the end-user. If someone wants to trade money with a friend, with an 
exchange, with a business, anywhere - they want to know they are properly handing over the liability of 
what happens with those funds. This ensures there are proper channels for legal recourse when required. 

This proof is provided as a chain of liable, well-identified parties. However, privacy should be maintained as best 
as possible. I should not need to know the details of the person with whom I'm interacting; merely, I should have 
trustworthy evidence that they are capable of bearing responsibility for their actions. This comes as a guarantee
from a liable party, the ecosystem owner.
 
All ongoing reporting and fraud prevention is then the responsibility of the ecosystem owner, which may have 
delegated it further (through agreements) with other parties (validators). An integrator needn't treat
tokens differently which have been validated by differing parties (unless they want to).

In the cases of businesses which custodialize their customers' funds, they must (at the least, in the US) be licensed 
money service businesses. They must follow all relevant reporting requirements as dictated by their business practices
in their own locality, and this protocol does nothing to alleviate those requirements. However, this protocol does
offer an avenue to offload this relatively rigorous process to a set of the validators in the 
ecosystem. By forming an agreement with a validator, enabling an end-user UX flow for such processing, they can gain
access to a much larger body of verified customers through the on-chain query API.

#### YES Marks: Compliance Attestation Codes

A YES mark (8-bit unsigned integer) is a code which, by convention, maps to a specific compliance attestation as given 
below. The tiers of permissions required for the outlined mechanics are also encapsulated within this structure via 
codes 128 and 129. All codes are granted in country-specific contexts, except for the owner code 128 (which has a country
code of 0).

Many codes have a generalized meaning along with a country-specific definition, which is contextualized by the local
agencies and regulations of a specific country. Regional distinctions are not supported; attestions must be country-wide, 
though it is the responsibility of any particular _integrator_ to ensure their customers are not accessing a service from
a more restrictive or unsupported region within a country.

	Special - country code 0:
	128. (special) Ecosystem Owner - only one. can attest all marks except `128`
    129. (special) Ecosystem Validator - can attest all marks except `128`, `129`

    General Definitions:

    	These define the context and minimum requirements of each attestation mark without concern
    	for differences between countries.

	    Individual: (most significant bit is 0)
	    1. Tier 1 Individual
	       - Identified
	       - Evidence for Possession of Valid Government ID
	       - AML Compliance
	    2. Tier 2 Individual - Tier 1 + "Dynamic Proof" (verified video authorization)
	    10. Accredited Investor

	    Business/Organization: (most significant bit is 1)
	    130. Tier 1 Corporation
	      - Public Company Registration Information
	      - Board Member Verification (to Tier 1 ???)
	      - WHAT ELSE?
	    131. ??? GDPR [*] - Generalized validation/recognition of following GDPR practices
	    200. Money Services Business
	    201. ??? (generalized name for ATS license holder?)

    Per-Country Specifications:
	    - USA (840):
	      - 1. Tier 1 Compliance:
	        - Identification: Name, DOB, SSN
	        - ID Evidence: Scan of drivers license OR passport
	        - AML: Copy of a bank statement
	      - 2. Tier 2 Compliance - 5 second video with specific verbal authorization, (including verifier organization's name)
	      - 10. Accredited Investor: Follows SEC-defined regulations - Rule 501 of Regulation D
	      - 130. US S or C Corp. 
	        - Articles of Incorporation
	        - Cap table
	        - ???
	      - 200. FinCEN Registered MSB - https://www.fincen.gov/msb-registrant-search
	      - 201. SEC Registered ATS License Holder

## Rationale
<!--The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made. It should describe alternate designs that were considered and related work, e.g. how the feature is supported in other languages. The rationale may also provide evidence of consensus within the community, and should discuss important objections or concerns raised during discussion.-->

todo

- 721 - ease/standardization/wallet integrations
- outcome-signatures instead of collections of proof items
- limited/finalization for security
- single-token-many-verifier

<!-- ## Test Cases
We have test cases covering all the fundamental mechanics of the draft implementation [here](https://github.com/sendwyre/yes-compliance-token/blob/develop/test/yestoken.js). 
-->

## Implementation
<!--The implementations must be completed before any EIP is given status "Final", but it need not be completed before the EIP is accepted. While there is merit to the approach of reaching consensus on the specification and rationale before writing code, the principle of "rough consensus and running code" is still useful when it comes to resolving many discussions of API details.-->
A draft implementation covering the entirety of this spec is located [here](https://github.com/sendwyre/yes-compliance-token).

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).










