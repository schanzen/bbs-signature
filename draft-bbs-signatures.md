%%%
title = "The BBS Signature Scheme"
abbrev = "The BBS Signature Scheme"
ipr= "none"
area = "Internet"
workgroup = "none"
submissiontype = "IETF"
keyword = [""]

[seriesInfo]
name = "Individual-Draft"
value = "draft-bbs-signatures-latest"
status = "informational"

[[author]]
initials = "M."
surname = "Lodder"
fullname = "Mike Lodder"
#role = "editor"
organization = "CryptID"
  [author.address]
  email = "redmike7gmail.com"

[[author]]
initials = "T."
surname = "Looker"
fullname = "Tobias Looker"
#role = "editor"
organization = "Mattr"
  [author.address]
  email = "tobias.looker@mattr.global"

[[author]]
initials = "A."
surname = "Whitehead"
fullname = "Andrew Whitehead"
#role = "editor"
organization = ""
  [author.address]
  email = "cywolf@gmail.com"
%%%

.# Abstract

BBS is a form of short group digital signature scheme that supports multi-message signing that produces a single output digital signature. The scheme allows a possessor of a signature to derive proofs that selectively reveal from the originally signed set of messages, whilst preserving verifiable authenticity and integrity of the messages. Derived proofs are said to be zero-knowledge in nature as they do not reveal the underlying signature, instead proof of knowledge of the signature.

{mainmatter}

# Introduction

A digital signature scheme is a fundamental cryptographic primitive that is used to provide data integrity and verifiable authenticity in various protocols. The core premise of digital signature technology is built upon asymmetric cryptography where-by the possessor of a private key is able to sign a payload (often revered to as the signer), where anyone in possession of the corresponding public key matching that of the private key is able to verify the signature.

However traditional signature schemes require the entire signature and message to be disclosed during verification. BBS allows a fast and small zero-knowledge signature proof of knowledge to be created from the signature and the public key. This allows the signature holder to selectively reveal any number of signed messages to another entity (none, all, or any number in between).

A recent emerging use case applies signature schemes in [verifiable credentials](https://www.w3.org/TR/vc-data-model/). One problem with
using simple signature schemes like ECDSA or ED25519 is that a holder must disclose the entire signed message and signature for verification. Circuit based logic can be applied to verify these in zero-knowledge like SNARKS or Bulletproofs with R1CS but tend to be complicated. BBS on the other hand adds, to verifiable credentials or any other application, the ability to do very efficient zero-knowledge proofs. A holder gains the ability to choose which claims to reveal to a relying party without the need for any additional complicated logic. (FIXME: Informative references missing)

## Notational Conventions

The keywords **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**,
**SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL**, when they appear in this
document, are to be interpreted as described in [@!RFC2119].

## Terminology

The following terminology is used throughout this document:

SK
: The secret key for the signature scheme.

PK
: The public key for the signature scheme.

L
: The total number of messages that the signature scheme can sign.

R
: The set of message indices that are retained or hidden in a signature proof of knowledge.

D
: The set of message indices that are disclosed in a signature proof of knowledge.

msg
: The input to be signed by the signature scheme.

h\[i\]
: The generator corresponding to a given msg.

h0
: A generator for the blinding value in the signature.

signature
: The digital signature output.

commitment
: A pedersen commitment composed of 1 or more messages.

nonce
: A cryptographic nonce

presentation_message (pm)
: A message generated and bound to the context of a specific spk.

spk
: Zero-Knowledge Signature Proof of Knowledge.

nizk
: A non-interactive zero-knowledge proof from fiat-shamir heuristic.

dst
: The domain separation tag.

I2OSP
: As defined by Section 4 of [@!RFC8017]

OS2IP
: As defined by Section 4 of [@!RFC8017].

a || b
: Denotes the concatenation of octet strings a and b.

Terms specific to pairing-friendly elliptic curves that are relevant to this document are restated below, originally defined in [@!I-D.irtf-cfrg-pairing-friendly-curves]

E1, E2
: elliptic curve groups defined over finite fields. This document assumes that E1 has a more compact representation than E2, i.e., because E1 is defined over a smaller field than E2.

G1, G2
: subgroups of E1 and E2 (respectively) having prime order r.

GT
: a subgroup, of prime order r, of the multiplicative group of a field extension.

e
: G1 x G2 -> GT: a non-degenerate bilinear map.

P1, P2
: points on G1 and G2 respectively. For a pairing-friendly curve, this document denotes operations in E1 and E2 in additive notation, i.e., P + Q denotes point addition and x \* P denotes scalar multiplication. Operations in GT are written in multiplicative notation, i.e., a \* b is field multiplication.

hash\_to\_curve\_g1(ostr) -> P
: The cryptographic hash function that takes as an arbitrary octet string input and returns a point in G1 as defined in [@!I-D.irtf-cfrg-hash-to-curve]. The algorithm first requires selection of the pairing friendly curve and digest
algorithm, once selected apply the isogeny simplified SWU map to compute a point in G1 using the random oracle method. The domain separation tag value is dst.

hash\_to\_curve\_g2(ostr) -> P
: The cryptographic hash function that takes as an arbitrary octet string input and returns a point in G2 as defined in [@!I-D.irtf-cfrg-hash-to-curve]. The algorithm first requires selection of the pairing friendly curve and digest algorithm, once selected the isogeny simplified SWU map to compute a point in G2 using the random oracle method. The domain separation tag value is dst.

point\_to\_octets\_min(P) -> ostr
: returns the canonical representation of the point P as an octet string in compressed form. This operation is also known as serialization. (FIXME: This either requires a normative wire format or a reference to the normative wire format)

point\_to\_octets\_norm(P) -> ostr
: returns the canonical representation of the point P as an octet string in uncompressed form. This operation is also known as serialization. (FIXME: This either requires a normative wire format or a reference to the normative wire format)

octets\_to\_point(ostr) -> P
: returns the point P corresponding to the canonical representation ostr, or INVALID if ostr is not a valid output of point\_to\_octets.  This operation is also known as deserialization. Consumes either compressed or uncompressed ostr.

subgroup\_check(P) -> VALID or INVALID
: returns VALID when the point P is an element of the subgroup of order r, and INVALID otherwise. This function can always be implemented by checking that r \* P is equal to the identity element.  In some cases, faster checks may also exist, e.g., [Bowe19].

# Overview

//TODO

## Organization of this document

This document is organized as follows:

* (#core-operations) defines core operations of the signature scheme.

* (#security-considerations) defines security considerations associated to the signature scheme.

* (#profiles) defines concrete profiles for the signature scheme.

# Core operations

This section defines core operations used by the schemes defined in Section 3. These operations MUST NOT be used except as described in that section.

## Parameters

The core operations in this section depend on several parameters:

* A pairing-friendly elliptic curve, plus associated functionality given in Section 1.4.

* H, a hash function that MUST be a secure cryptographic hash function. For security, H MUST output at least ceil(log2(r)) bits, where r is the order of the subgroups G1 and G2 defined by the pairing-friendly elliptic curve.

* PRF(n): a pseudo-random function similar to [@!RFC4868]. Returns n pseudo randomly generated bytes.

## KeyGen

The KeyGen algorithm generates a secret key SK deterministically from a secret octet string IKM.

KeyGen uses a HKDF [@!RFC5869] instantiated with the hash function H.

For security, IKM MUST be infeasible to guess, e.g., generated by a trusted source of randomness.

IKM MUST be at least 32 bytes long, but it MAY be longer.

Because KeyGen is deterministic, implementations MAY choose either to store the resulting SK or to store IKM and call KeyGen to derive SK when necessary.

KeyGen takes an optional parameter, key\_info. This parameter MAY be used to derive multiple independent keys from the same IKM.  By default, key\_info is the empty string.

```
SK = KeyGen(IKM)
```

Inputs:

- IKM, a secret octet string. See requirements above.

Outputs:

- SK, a uniformly random integer such that 0 < SK < r.

Parameters:

- key\_info, an optional octect string. if this is not supplied, it MUST default to an empty string.

Definitions:

- HKDF-Extract is as defined in [@!RFC5869], instantiated with hash H.
- HKDF-Expand is as defined in [@!RFC5869], instantiated with hash H.
- I2OSP and OS2IP are as defined in [@!RFC8017], Section 4.
- L is the integer given by ceil((3 \* ceil(log2(r))) / 16).
- "BBS-SIG-KEYGEN-SALT-" is an ASCII string comprising 20 octets.

Procedure:

1. PRK = HKDF-Extract("BBS-SIG-KEYGEN-SALT-", IKM || I2OSP(0, 1))

2. OKM = HKDF-Expand(PRK, key\_info || I2OSP(L, 2), L)

3. SK = OS2IP(OKM) mod r

4. return SK

## SkToPk

SkToPk algorithm takes a secret key SK and outputs a corresponding public key.

```
PK = SkToPk(SK)
```

Inputs:

- SK, a secret integer such that 0 < SK < r

Outputs:

- PK, a public key encoded as an octet string

Procedure:

1. w = SK \* P2

2. PK = w

3. return point\_to\_octets\_min(PK)

## KeyValidate

KeyValidate checks if the public key is valid.

As an optimization, implementations MAY cache the result of KeyValidate in order to avoid unnecessarily repeating validation for known keys.

```
result = KeyValidate(PK)
```

Inputs:

- PK, a public key in the format output by SkToPk.

Outputs:

- result, either VALID or INVALID

Procedure:

1. (w, h0, h) = octets\_to\_point(PK)

2. result = subgroup\_check(w) && subgroup\_check(h0)

3. for i in 0 to len(h): result &= subgroup\_check(h\[i\])

4. return result

## Sign

Sign computes a signature from SK, PK, over a vector of messages.

```
signature = Sign((msg[i],...,msg[L]), SK, PK)
```

Inputs:

- msg\[i\],...,msg\[L\], octet strings
- SK, a secret key output from KeyGen
- PK, a public key output from SkToPk

Outputs:

- signature, an octet string

Procedure:

1. (w, h0, h) = octets\_to\_point(PK)

2. e = H(PRF(8*ceil(log2(r)))) mod r

3. s = H(PRF(8*ceil(log2(r)))) mod r

4. b = P1 + h0 \* s + h\[i\] \* msg\[i\] + ... + h\[n\] \* msg\[L\]

5. A = b \* (1 / (SK + e))

6. signature = (point\_to\_octets\_min(A), e, s)

7. return signature

## Verify

Verify checks that a signature is valid for the octet string messages under the public key.

```
result = Verify((msg[i],...,msg[L]), signature, PK)
```

Inputs:

- msg\[i\],...,msg\[L\], octet strings.
- signature, octet string.
- PK, a public key in the format output by SkToPk.

Outputs:

- result, either VALID or INVALID.

Procedure:

1. (A, e, s) = octets\_to\_point(signature.A), e, s

2. pub\_key = octets\_to\_point(PK)

3. if subgroup\_check(A) is INVALID

4. if KeyValidate(pub\_key) is INVALID

5. b = P1 + h0 \* s + h\[i\] \* msg\[i\] + ... + h\[n\] \* msg\[L\]

6. C1 = e(A, w + P2 \* e)

7. C2 = e(b, P2)

8. return C1 == C2

## SpkGen

A signature proof of knowledge generating algorithm that creates a zero-knowledge proof of knowledge of a signature while selectively disclosing messages from a signature given a vector of messages, a vector of indices of the revealed messages, the signer's public key, and a presentation message.

```
spk = SpkGen(PK, (msg[1],...,msg[L]), RIdxs, signature, pm)
```

Inputs:

- PK, octet string in output form from SkToPk
- (msg\[1\],...,msg\[L\]), octet strings (messages in input to Sign).
- RIdxs, vector of unsigned integers (indices of revealed messages).
- signature, octet string in output form from Sign
- pm, octet string

Outputs:

- spk, octet string

Procedure:

1. (A, e, s) = signature

2. (w, h0, h\[1\],...,h\[L\]) = PK

3. (i1, i2,..., iR) = RIdxs

4. if subgroup\_check(A) is INVALID abort

5. if KeyValidate(PK) is INVALID abort

6. b = commitment + h0 \* s + h\[1\] \* msg\[1\] + ... + h\[L\] \* msg\[L\]

7. r1 = H(PRF(8\*ceil(log2(r)))) mod r

8. r2 = H(PRF(8\*ceil(log2(r)))) mod r

9. e\~ = H(PRF(8\*ceil(log2(r)))) mod r

10. r2\~ = H(PRF(8\*ceil(log2(r)))) mod r

11. r3\~ = H(PRF(8\*ceil(log2(r)))) mod r

12. s\~ = H(PRF(8\*ceil(log2(r)))) mod r

13. r3 = r1 ^ -1 mod r

14. for i in RIdxs m\~\[i\] = H(PRF(8\*ceil(log2(r)))) mod r

15. A' = A \* r1

16. Abar = A' \* -e + b \* r1

17. d = b \* r1 + h0 \* -r2

18. s' = s - r2 \* r3

19. C1 = A' \* e\~ + h0 \* r2\~

20. C2 = d \* r3\~ + h0 \* s\~ + h\[i1\] \* m\~\[i1\] + ... + h\[iR\] \* m\~\[iR\]

21. c = H(Abar || A' || h0 || C1 || d || h0 || h\[i1\] || ... || h\[iR\] || C2 || pm)

22. e^ = e\~ + c \* e

23. r2^ = r2\~ - c \* r2

24. r3^ = r3\~ + c \* r3

25. s^ = s\~ - c \* s'

26. for i in RIdxs: m^\[i\] = m\~\[i\] - c \* msg\[i\]

27. spk = ( A', Abar, d, C1, e^, r2^, C2, r3^, s^, (m^\[i1\], ..., m^\[iR\]) )

28. return spk

How a signature is to be encoded is not covered by this document. (TODO perhaps add some additional information in the appendix)
(FIXME: Encoding out of scope OK, but wire format is required for test vectors. Unless this document is not normative.)

## SpkVerify

SpkVerify checks if a signature proof of knowledge is VALID given the proof, the signer's public key, a vector of revealed messages, a vector with the indices of these revealed messages, and the presentation message used in SpkGen.

```
result = SpkVerify(spk, PK, (Rmsg[1],..., Rmsg[R]), RIdxs, pm)
```

Inputs:

- spk, octet string.
- PK, octet string in output form from SkToPk.
- (Rmsg\[1\], ..., Rmsg\[R\]), octet strings (revealed messages).
- RIdxs, vector of unsigned integers (indices of revealed messages).
- pm, octet string

Outputs:

- result, either VALID or INVALID.

Procedure:

1. if KeyValidate(PK) is INVALID

2. (i1, i2, ..., iR) = RIndxs

3. (A', Abar, d, C1, e^, r2^, C2, r3^, s^, (m^\[i1\],...,m^\[iR\])) = spk

4. (w, h0, h\[1\],...,h\[L\]) = PK

5. if A' == 1 return INVALID

6. X1 = e(A', w)

7. X2 = e(Abar, P2)

8. if X1 != X2 return INVALID

9. c = H(Abar || A' || h0 || C1 || d || h0 || h\[i1\] || ... || h\[iR\] || C2 || pm)

10. T1 = Abar - d

11. T2 = P1 + h\[i1\] \* Rmsg\[1\] + ... + h\[iR\] \* Rmsg\[R\]

12. Y1 = A' \* e^ + h0 \* r2^ + T1 \* c

13. Y2 = d \* r3^ + h0 \* s^ + h\[i1\] \* m^\[i1\] + ... + h\[iR\] \* m^\[iR\] - T2 \* c

14. if C1 != Y1 return INVALID

15. if C2 != Y2 return INVALID

16. return VALID

# Security Considerations

## Validating public keys

All algorithms in Section 2 that operate on points in public keys require first validating those keys.  For the sign, verify and proof schemes, the use of KeyValidate is REQUIRED.

## Skipping membership checks

Some existing implementations skip the subgroup\_check invocation in Verify (Section 2.8), whose purpose is ensuring that the signature is an element of a prime-order subgroup.  This check is REQUIRED of conforming implementations, for two reasons.

1.  For most pairing-friendly elliptic curves used in practice, the pairing operation e (Section 1.3) is undefined when its input points are not in the prime-order subgroups of E1 and E2. The resulting behavior is unpredictable, and may enable forgeries.

2.  Even if the pairing operation behaves properly on inputs that are outside the correct subgroups, skipping the subgroup check breaks the strong unforgeability property [ADR02].

## Side channel attacks

Implementations of the signing algorithm SHOULD protect the secret key from side-channel attacks.  One method for protecting against certain side-channel attacks is ensuring that the implementation executes exactly the same sequence of instructions and performs exactly the same memory accesses, for any value of the secret key. In other words, implementations on the underlying pairing-friendly elliptic curve SHOULD run in constant time.

## Randomness considerations

The IKM input to KeyGen MUST be infeasible to guess and MUST be kept secret. One possibility is to generate IKM from a trusted source of randomness.  Guidelines on constructing such a source are outside the scope of this document.

Secret keys MAY be generated using other methods; in this case they MUST be infeasible to guess and MUST be indistinguishable from uniformly random modulo r.

BBS signatures are nondeterministic, meaning care must be taken against attacks arising from signing with bad randomness, for example, the nonce reuse attack on ECDSA [HDWH12]. It is recommended that the nonces used in signature proof generation are from a trusted source of randomness.

BlindSign as discussed in 2.10 uses randomness from two parties so care MUST be taken that both sources of randomness are trusted. If one party uses weak randomness, it could compromise the signature.

When a trusted source of randomness is used, signatures and proofs are much harder to forge or break due to the use of multiple nonces.

## Implementing hash\_to\_point\_g1 and hash\_to\_point\_g2

The security analysis models hash\_to\_point and hash\_pubkey\_to\_point as random oracles.  It is crucial that these functions are implemented using a cryptographically secure hash function.  For this purpose, implementations MUST meet the requirements of [@!I-D.irtf-cfrg-hash-to-curve].

In addition, ciphersuites MUST specify unique domain separation tags for hash\_to\_point.  The domain separation tag used in Section 1.4 is the RECOMMENDED one.

## Use of Contexts

Contexts can be used to separate uses of the protocol between different protocols (which is very hard to reliably do otherwise) and between different uses within the same protocol. However, the following SHOULD be kept in mind:

The context SHOULD be a constant string specified by the protocol using it. It SHOULD NOT incorporate variable elements from the message itself.

Contexts SHOULD NOT be used opportunistically, as that kind of use is very error prone. If contexts are used, one SHOULD require all signature schemes available for use in that purpose support contexts.

Contexts are an extra input, which percolate out of APIs; as such, even if the signature scheme supports contexts, those may not be available for use. This problem is compounded by the fact that many times the application is not invoking the signing, verification, and proof functions directly but via some other protocol.

The ZKP protocols use nonces which MUST be different in each context.

## Choice of underlying curve

BBS signatures can be implemented on any pairing-friendly curve. However care MUST be taken when selecting one that is appropriate, this specification defines a profile for using the BLS12-381 curve in (#profiles) which as a curve currently achieves close to 128-bit security.

# Profiles

## BLS12-381

### Definition

The following defines the correspondence between the primitives in (#core-operations) and the parameters given in Section 4.2.2 of [@!I-D.irtf-cfrg-pairing-friendly-curves].

* E1, G1: the curve E and its order-r subgroup.

* E2, G2: the curve E' and its order-r subgroup.

* GT: the subgroup G\_T.

* P1: the point BP.

* P2: the point BP'.

* e: the optimal Ate pairing defined in Appendix A of [@!I-D.irtf-cfrg-pairing-friendly-curves].

* point\_to\_octets and octets\_to\_point use the compressed serialization formats for E1 and E2 defined by [ZCash].

* subgroup_check MAY use either the naive check described in Section 1.3 or the optimized check given by [Bowe19].

### Test Vectors

//TODO

# IANA Considerations

This document does not make any requests of IANA.

# Appendix

## Usecases

### Non-correlating Security Token

In the most general sense BBS signatures can be used in any application where a cryptographically secured token is required but correlation caused by usage of the token is un-desirable.

For example in protocols like OAuth2.0 the most commonly used form of the access token leverages the JWT format alongside conventional cryptographic primitives such as traditional digital signatures or HMACs. These access tokens are then used by a relying party to prove authority to a resource server during a request. However, because the access token is most commonly sent by value as it was issued by the authorization server (e.g in a bearer style scheme), the access token can act as a source of strong correlation for the relying party. Relevant prior art can be found [here](https://www.ietf.org/archive/id/draft-private-access-tokens-01.html).

BBS Signatures due to their unique properties removes this source of correlation but maintains the same set of guarantees required by a resource server to validate an access token back to its relevant authority (note that an approach to signing JSON tokens with BBS that may be of relevance is the [JWP](https://json-web-proofs.github.io/json-web-proofs/draft-jmiller-json-web-proof.html) format and serialization). In the context of a protocol like OAuth2.0 the access token issued by the authorization server would feature a BBS Signature, however instead of the relying party providing this access token as issued, in their request to a resource server, they derive a unique proof from the original access token and include that in the request instead, thus removing this vector of correlation.

### Improved Bearer Security Token

Bearer based security tokens such as JWT based access tokens used in the OAuth2.0 protocol are a highly popular format for expressing authorization grants. However their usage has several security limitations. Notably a bearer based authorization scheme often has to rely on a secure transport between the authorized party (client) and the resource server to mitigate the potential for a MITM attack or a malicious interception of the access token. The scheme also has to assume a degree of trust in the resource server it is presenting an access token to, particularly when the access token grants more than just access to the target resource server, because in a bearer based authorization scheme, anyone who possesses the access token has authority to what it grants. Bearer based access tokens also suffer from the threat of replay attacks.

Improved schemes around authorization protocols often involve adding a layer of proof of cryptographic key possession to the presentation of an access token, which mitigates the deficiencies highlighted above as well as providing a way to detect a replay attack. However, approaches that involve proof of cryptographic key possession such as DPoP (https://datatracker.ietf.org/doc/html/draft-ietf-oauth-dpop-04) suffer from an increase in protocol complexity. A party requesting authorization must pre-generate appropriate key material, share the public portion of this with the authorization server alongside proving possession of the private portion of the key material. The authorization server must also be-able to accommodate receiving this information and validating it.

BBS Signatures ofter an alternative model that solves the same problems that proof of cryptographic key possession schemes do for bearer based schemes, but in a way that doesn't introduce new up-front protocol complexity. In the context of a protocol like OAuth2.0 the access token issued by the authorization server would feature a BBS Signature, however instead of the relying party providing this access token as issued, in their request to a resource server, they derive a unique proof from the original access token and include that in the request instead. Because the access token is not shared in a request to a resource server, attacks such as MITM are mitigated. A resource server also obtains the ability to detect a replay attack by ensuring the proof presented is unique.

### Hardware Attestations

TODO

### Selectively Disclosure Enabled Identity Assertions

TODO

### Privacy preserving bound signatures

TODO

{backmatter}