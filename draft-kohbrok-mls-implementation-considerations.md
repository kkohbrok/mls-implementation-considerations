---
title: "The Messaging Layer Security (MLS) Implementation Considerations"
abbrev: "MLS Implementations"
docname: draft-kohbrok-mls-implementation-considerations-latest
category: info

ipr: trust200902
area: Security
keyword: Internet-Draft

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
 -
    ins: K. Kohbrok
    name: Konrad Kohbrok
    organization: Wire
    email: konrad@wire.com

normative:
  RFC2119:

informative:



--- abstract

The MLS protocol is a large and complex security protocol. In this document, we
provide guidance and recommendations for implementers of MLS regarding security,
efficiency and robustness.

--- middle

# Introduction

TODO Introduction


# Conventions and Definitions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in BCP 14 {{RFC2119}} {{!RFC8174}}
when, and only when, they appear in all capitals, as shown here.


# Security Considerations

## Improving Resiliance to Poor Random Number Generators

The security of a group hinges on every member having access to a good source of
entropy. To make the protocol more robust in environments with only weak,
intermitent (or even no) external sources of entropy, each MLS client SHOULD
maintain a persistant `entropy_pool` value which is initialized to a uniform
random octet string (e.g. using the operating system's Random Number Generator
(RNG)) of length `KDF.Nh`, where `KDF` is the KDF supported by the client's
ciphersuites with the highest value of `Nh`. The pool serves as a cache which
continuously accumulates entropy over the lifetime of the client.

The entropy pool functions in a similar way as the Key Schedule in that clients
can repeatedly perform two types of operations with their entropy pool:

1. "inject" fresh inputs into their pool to accumulate any entropy the
input contains and
2. "extract" a random value from the pool.

### Entropy Pool Security

Whenever enough entropy has accumulated in a pool, all subsequent values
extracted from it will look as if they were sampled from a truly uniform random
and independent entropy source. This holds even if an adversary can control all
inputs mixed into the pool after is has accumulated enough entropy. Pools can
accumulate sufficient entropy either by being initialized using a good source of
randomness or by having sufficient entropy mixed in (possibly accumulated over a
number of inputs).

In the same way as the Key Schedule, Forward Secrecy is achieved by "ratcheting
the `entropy_pool` forward" as part of the process of extracting a value.
Conversely, to ensure that Post-Compromise Security (PCS) can be achieved again
after `entropy_pool` was leaked to an adversary, fresh entropy can mixed in to
the pool.

### Entropy Pool Operations

Clients obtain inputs to mix in to their pool both from sources external to the
MLS protocol (such as the OS's RNG) and sources internal to the MLS protocol
(i.e. exported from key schedules of ongoing MLS groups). Mixing in internal
sources can allow clients with no local source of entropy to piggyback off of
the entropy sources of other members of one or more groups they are in. A pool
SHOULD be shared between all MLS groups on the client as this can lead to more
diverse sources of internal input improving the likelyhood that, at any given
moment, the pool contains sufficient entropy.

The operations of the entropy pool are extract-expand cycles, where an extract
operation is performed based on the current `entropy_pool` value, plus an
optional source of entropy that is injected followed by an expand operation to
derive a new `entropy_pool` value, as well as optionally a additional fresh
random value.

To extract a fresh value (denoted `fresh_secret`) for use in an MLS group,
clients MUST first mix in a new `external_entropy` sampled from an external RNG.

~~~~~
             entropy_pool_[n-1]
                        |
                        V
external_entropy -> KDF.Extract = intermediate_secret
                        |
                        +--> DeriveSecret(., "mls_secret")
                        |    = fresh_secret
                        |
                        V
                  DeriveSecret(., "entropy_pool")
                  = entropy_pool_[n]
~~~~~

Besides inputs from external sources, clients also mix in entropy from internal
sources. That is, when processing an incoming welcome or commit packet sent by
another group member clients derive a fresh `internal_entropy` value from the
new epochs key schedule and mix it in to their `entropy_pool`. To ensure that
every group member uses a distinct value to inject into their entropy pool, the
MLS client's `leaf_index` is used as context when deriving the
`internal_entropy`.

~~~~~
internal_entropy =
    ExpandWithLabel(`epoch_secret`, "internal_update", leaf_index, KDF.Nh)
~~~~~

Where `KDF.Nh` is referring to the `KDF` of the ciphersuite of the group that
the randomness is extracted from.

The resulting `internal_entropy` is mixed in to the entropy pool as follows.

~~~~~
             entropy_pool_[n-1]
                        |
                        V
internal_entropy -> KDF.Extract = intermediate_secret
                        |
                        V
                  DeriveSecret(., "entropy_pool")
                  = entropy_pool_[n]
~~~~~

In particular, when an `internal_entropy` input derived from a secure epoch is
mixed into the pool in this way, the resulting `entropy_pool_[n]` has sufficient
entropy regardless of the entropy in the preceding `entropy_pool_[n-1]`.
Meanwhile, if `entropy_pool_[n-1]` already had sufficient entropy than so does
`entropy_pool_[n]`, even when the adversary knows the value of
`internal_entropy`. As a consequence, this operation strictly improves the
quality of the gathered entropy.

MLS clients SHOULD inject `internal_entropy` whenever processing an update from
another party in any of their groups.

### External Entropy Sources

The OS' RNG is the primary source for fresh entropy for injection into the
entropy pool and MUST be used whenever a `fresh_value` is retrieved. However,
the entropy pool can benefit from other external sources as well. For example,
{{!RFC8937}} proposes the injection of the function of a party's long term key
into the entropy pool. Similarly, exported keys from other cryptographic
security protocols such as TLS can be leveraged to improve the quality of the
overall entropy in the pool.

# IANA Considerations

This document has no IANA actions.

# Contributors

* Joel Alwen \\
  Wickr \\
  joel.alwen@wickr.com

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
