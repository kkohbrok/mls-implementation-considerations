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
provide guidance and recommendations for implementers of MLS regarding
correctness, security, efficiency and robustness.

--- middle

# Introduction

Implementing the specification of a complex security protocol in a correct and
secure manner is a challenging task.

This document aims to provide guidance to implementers of the Messaging
Layer Security (MLS) protocol specification {{?I-D.ietf-mls-protocol}} in a
variety of ways.

The goal of this document is also to extend the existing guidance given in the
MLS architecture document {{?I-D.ietf-mls-architecture}} on how to use MLS
securely in a larger context.

Finally, this document is also meant to provide examples on how to fill several
of the gaps that the protocol specification and architective document have left
open, but that are likely going to be needed by a large number of MLS users.

# Conventions and Definitions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in BCP 14 {{RFC2119}} {{!RFC8174}}
when, and only when, they appear in all capitals, as shown here.

# Typed structs
In many places, MLS uses a typed pattern of structs, where we have a type `uint`,
followed by a `select` statement that decides the type of the following field based on the preceding type `uint`.
For example the Credential struct:

```
struct {
    CredentialType credential_type;
    select (Credential.credential_type) {
        case basic:
            BasicCredential;

        case x509:
            Certificate chain<1..2^32-1>;
    };
} Credential;
```

Other examples are `Content`/`ContentType`, `PSKType` in `PreSharedKey`, `Sender`/`SenderType` and `Proposal`/`ProposalType`.

We can generalise this into a typed struct as follows:

```
struct {
    T type;
    select(typed<T>.type) {
        T.variant1: 
            Variant1 
    }
} typed<T>
```

# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.



--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
