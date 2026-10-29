---

title: "Clearing the DO bit for DNSSEC"
abbrev: "No DO for you"
category: info

docname: draft-wagope-dnsop-do-bit-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: AREA
workgroup: WG Working Group
keyword:
 - DNSSEC
 - PQC
 - large signatures
venue:
  group: "DNSOP"
  type: Working Group
  #mail: WG@example.com
  #arch: https://example.com/WG
  github: wkumari/draft-wagope-dnsop-do-bit
  latest: https://wkumari.github.io/draft-wagope-dnsop-do-bit/draft-wagope-dnsop-do-bit.html

author:
  -
    fullname: Warren Kumari
    organization: Google, Inc.
    email: warren@kumari.net
  -
    ins: G. Huston
    fullname: Geoff Huston
    organization: APNIC
    email: gih@apnic.net
  -
    fullname: Peter Thomassen
    organization: deSEC, SSE
    email: peter@desec.io


normative:
  RFC4034:

informative:

...

--- abstract

The DNSSEC OK (DO) bit is an EDNS0 flag used in DNS queries to signal that the
requestor wishes to receive DNSSEC resource records (such as RRSIG, DNSKEY, DS,
and NSEC) in the response.

This bit is named DNSEC OK (DO) as an indication that the resolver
implementation will not die if it receives DNSSEC records, not that it is
actually performing DNSSEC validation.

It is important to note that clearing the DO bit means that the authoritative
server will not include DNSSEC records in its response, even if they exist.

One of the concerns around Post Quantum Cryptography (PQC) is that the
resulting DNSSEC signatures may be significantly larger than current
signatures. This could exacerbate issues related to DNS message size and
fragmentation, and cause many connections to fail back to TCP.Clearing the DO
bit can help mitigate these issues by preventing the inclusion of potentially
large DNSSEC records in responses, but it also means that resolvers will not
have access to the DNSSEC proofs necessary for validation.

--- middle

# Introduction

One of the primary concerns around deploying Post Quantum Cryptography (PQC)
algorithms in DNSSEC is that the resulting signatures will almost certainly
exceed the size of UDP responses.

This document proposes that non-DNSEC validating resolvers clear the DO bit as
they do not (necessarily) need the DNSSEC records.

This approach may help mitigate the number of TCP connections that authoritative
servers need to handle, but at the cost of additional complexity and number of
queries.

# Detail and implications

If a resolver is not performing DNSSEC validation, and the incoming query has
the DO bit clear, the resolver SHOULD clear the DO bit in its outgoing queries.
This signals to authoritative servers that DNSSEC records are not required in
the response, reducing the size of the response and avoiding TCP fallback.

This does, however, mean that if the recursive resolver subsequently receives a
query from a DNSSEC validating client (e.g a DNSSEC validating stub, or a
forwarded query from a DNSSEC validating resolver), it will not be able to
provide the necessary DNSSEC records for validation, which would cause
validation to fail.

For this reason, resolvers following this behavior MUST keep track in their
cache which responses were obtained without DNSSEC records. If a query for such
a cached entry is received with the DO bit set, the resolver must then re-query
the authoritative server with the DO bit set to obtain the necessary DNSSEC
records for validation.

# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments
{:numbered="false"}

The authors would like to thank the following individuals for their feedback
and suggestions: Gautam Akiwate, Russ Housley, Barry Leiba, John Levine, Rod
Rasmussen, Laurin Weissinger, Suzanne Woolf.

Unfortunately, at least one of the authors has a terrible memory, and has lost
track of all those who have contributed to this topic over the years, and will
be more than happy to acknowledge their input if reminded of this :-)

In addition, Warren would like to thank Infected Mushroom for once again
providing a good soundtrack during the writing of this document.