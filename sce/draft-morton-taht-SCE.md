%%%
title = "The Some Congestion Experienced ECN Codepoint"
abbrev = "sceb"
updates = [3168, 8311]
obsoletes = [ ]
ipr = "trust200902"
area = "Internet"
docname = "draft-morton-taht-SCE-00"
workgroup = "Transport Working Group"
submissiontype = "IETF"
keyword = [""]
#date = 2019-01-30T00:00:00Z

[seriesInfo]
name = "Internet-Draft"
value = "draft-morton-taht-SCE-00"
stream = "IETF"
status = "standard"

[[author]]
initials = "J."
surname = "Morton"
fullname = "Jonathon Morton"
#role = "editor"
organization = "Bufferbloat.net"
  [author.address]
  email = "chromatix99@gmail.com"
  phone = "+358 44 927 2377"
  [author.address.postal]
  street = "Kökkönranta 21"
  city = "PITKÄJÄRVI"
#  region = ""
  code = "31520"
  country = "FINLAND"
[[author]]
initials = "D."
surname = "Täht"
fullname = "David M. Täht"
#role = "editor"
organization = "TekLibre"
  [author.address]
  email = "dave@taht.net"
  phone = "+18312059740"
  [author.address.postal]
  street = "20600 Aldercroft Heights Rd"
  city = "Los Gatos"
  region = "Ca"
  code = "95033"
  country = "USA"
%%%

.# Abstract

This memo reclassifies ECT(1) to be an early notification of
congestion on ECT(0) marked packets, which can be used by AQM
algorithms and transports as an earlier signal of congestion than
CE. It is a simple, transparent, and backward compatible upgrade to
existing IETF-approved AQMs, [@!RFC3138], and nearly all congestion
control algorithms.

{mainmatter}

# Terminology

The keywords **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL**, when they appear in this document, are to be interpreted as described in [@!RFC2119].

# Introduction

This memo reclassifies ECT(1) to be an early notification of
congestion on ECT(0) marked packets, which can be used by AQM
algorithms and transports as an earlier signal of congestion than
CE ("Congestion Experienced").

This memo limits its scope to the redefinition of the ECT(1)
codepoint as SCE, "Some Congestion Experienced", with few brief
illustrations of how it may be used.

# Background

[@!RFC3168] defines the lower two bits of the (former) TOS byte in the IPv4/6 header as the ECN field.  This may take four values: Not-ECT, ECT(0), ECT(1) or CE.

   Binary  Keyword                                  References
   ------  -------                                  ----------
     00     Not-ECT (Not ECN-Capable Transport)     [RFC 3168]
     01     ECT(1) (ECN-Capable Transport(1))       [RFC 3168]
     10     ECT(0) (ECN-Capable Transport(0))       [RFC 3168]
     11     CE (Congestion Experienced)             [RFC 3168]

Research has shown that the ECT(1) codepoint goes essentially unused,
with the "Nonce Sum" extension to ECN having not been implemented in
practice and subsequently obsoleted by [@!RFC8311]. Additionally, known
[@!RFC3168]-compliant senders do not emit ECT(1), and compliant
middleboxes do not alter the field to ECT(1), while compliant
receivers all interpret ECT(1) identically to ECT(0).  These are
useful properties which represent an opportunity for improvement.

Experience gained with 7 years of [@RFC8290] deployment in the field
suggests that it remains difficult to maintain the desired 100% link
utilisation, whilst simultaneously strictly minimising induced delay
due to excess queue depth - irrespective of whether ECN is in use.
This leads to a reluctance amongst hardware vendors to implement the
most effective AQM schemes because their headline benchmarks are
throughput-based.

The underlying cause is the very sharp "multiplicative decrease"
reaction required of transport protocols to congestion signalling
(whether that be packet loss or CE marks), which tends to leave the
congestion window significantly smaller than the ideal BDP when
triggered at only slightly above the ideal value.  The availability of
this sharp response is required to assure network stability (AIMD
principle), but there is presently no standardised and
backwards-compatible means of providing a less drastic signal.

# Some Congestion Experienced

As consensus has arisen that some form of ECN signaling should be an
earlier signal than drop, this Internet Draft changes the meaning of
ECT(1) to be SCE, meaning "Some Congestion Experienced".  The above
ECN-field codepoint table then becomes:

   Binary  Keyword                                  References
   ------  -------                                  ----------
     00     Not-ECT (Not ECN-Capable Transport)     [@RFC3168]
     01     SCE (Some Congestion Experienced)       [This Internet-draft]
     10     ECT (ECN-Capable Transport)             [@RFC3168]
     11     CE (Congestion Experienced)             [@RFC3168]

This permits middleboxes implementing AQM to signal incipient
congestion, below the threshold required to justify setting CE, by
converting some proportion of ECT codepoints to SCE ("SCE marking").
Existing [@RFC3168] compliant receivers MUST transparently ignore this new
signal, and existing middleboxes MUST still be able to convert SCE to
CE as they presently do for ECT, thus ensuring backwards
compatibility. 

Additionally, SCE-aware middleboxes MUST still produce CE markings as
at present, retaining current behavior with [@RFC3168] ECN endpoints.

Permitted ECN codepoint packet transitions by middleboxes are:

```
   	Not-ECT ->   Not-ECT or DROP
   	ECT     ->   ECT or SCE or CE or DROP
   	SCE     ->   SCE or CE or DROP
   	CE      ->   CE or DROP
```

In other words, for ECN-aware flows, the ECN marking of an individual
packet MAY be increased by a middlebox to signal congestion, but MUST
NOT be decreased, and packets MUST NOT be altered to appear to be
ECN-aware if they were not originally, nor vice versa.  Note however
that SCE is numerically less than ECT, but semantically greater, and
the latter definition applies for this rule.

New SCE-aware receivers and transport protocols must continue to apply
the [@RFC3168] interpretation of the CE codepoint, that is, to signal
the sender to back off send rate to the same extent as if a packet
loss were detected.  This maintains compatibility with existing
middleboxes, senders and receivers.

New SCE-aware receivers and transport protocols should interpret the SCE
codepoint as an indication of mild congestion, with the relative
incidence of ECT and SCE codepoints received indicating the relative
severity of such congestion, and respond accordingly by applying send
rates intermediate between those resulting from a continuous sequence
of ECT codepoints, and those resulting from a CE codepoint.

Details of how to implement SCE awareness at the transport layer will
be left to additional Internet Drafts yet to be submitted.

To maximise the benefit of SCE, middleboxes MUST be capable of
producing SCE markings earlier than they presently produce CE
markings.

# Examples of use

## Cubic

Consider a TCP transport implementing CUBIC congestion control.  This
presently exhibits exponential cwnd growth during slow-start,
polynomial cwnd growth in steady-state, and multiplicative decrease
upon detecting a single CE marking or packet loss in one RTT cycle.

With SCE awareness, it might exit slow-start upon detecting a single
SCE marking, switch from polynomial to Reno-linear cwnd growth when
the SCE:ECT ratio exceeds 1:2, halt cwnd growth entirely when it
exceeds 1:1, and implement a Reno-linear decline when it exceeds 2:1,
in addition to retaining the sharp 40% decrease on detecting CE.  In
ideal circumstances, this would result in the cwnd stabilising at a
level which produces between 50% and 66% SCE marking at some
bottleneck on the path.

## TCP receiver side handling

SCE can potentially be handled entirely by the receiver and be
entirely independent of any of the dozens of [@RFC3168] compliant
congestion control algorithms.

A SCE TCP aware receiver MAY (based on a setsockopt option) choose to
interpret SCE markings as CE and send ECE notifications back to the
sender based on the ratio of markings, and/or variations in TCP
timestamps.

A TCP receiver desiring low latency SHOULD respond with ECE signals
earlier, one desiring higher bandwidth, later.

Alternatively, a SCE aware reciever can attempt to infer what
congestion control is being used on the sender side of the connection.

## Other 

New transports under development such as QUIC SHOULD implement a
multi-bit and finer grained signal back to the sender based on SCE.

# Related Work

[@RFC8087] [@RFC7567] [@RFC7928] [@RFC8290] [@RFC8289] [@RFC8033] [@RFC8034]

# IANA Considerations

There are no IANA considerations.

# Security Considerations

There are no security considerations.

# Acknowledgements

Much thanks to the members of the ecn-sane project, the "cake" bufferbloat.net mailing list, the ietf AQM mailing list, and tsvwg.

{backmatter}
