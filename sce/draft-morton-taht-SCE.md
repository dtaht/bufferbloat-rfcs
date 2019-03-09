%%%
title = "The Some Congestion Experienced Bit"
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
  phone = "+1"
  [author.address.postal]
  street = "PO Box "
  city = "San Francisco"
  region = "Ca"
  code = "94117"
  country = "USA"
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

This memo reclassifies ECT(0) to be an early notification of
congestion on ECT(1) marked packets, which can be used by AQM
algorithms and transports as an earlier signal of congestion than
CE. It is transparent, backward compatible upgrade to existing and
future IETF-approved AQM algorithms.

{mainmatter}

# Terminology

The keywords **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL**, when they appear in this document, are to be interpreted as described in [@!RFC2119].

# Introduction

[@RFC3168] defines the lower two bits of the (former) TOS byte in the IPv4/6 header as the ECN field.  This may take four values: Not-ECT, ECT(0), ECT(1) or CE.  To quote:

   IPv4 TOS Byte and IPv6 Traffic Class Octet

   Description:  The registrations are identical for IPv4 and IPv6.

   Bits 0-5:  see Differentiated Services Field Codepoints Registry
           (http://www.iana.org/assignments/dscp-registry)

   Bits 6-7, ECN Field:

   Binary  Keyword                                  References
   ------  -------                                  ----------
     00     Not-ECT (Not ECN-Capable Transport)     [RFC 3168]
     01     ECT(1) (ECN-Capable Transport(1))       [RFC 3168]
     10     ECT(0) (ECN-Capable Transport(0))       [RFC 3168]
     11     CE (Congestion Experienced)             [RFC 3168]

Presently however, the ECT(1) codepoint goes essentially unused, with
the Nonce Sum extension to ECN having not been implemented in practice
(and now considered unnecessary and obsolete).  RFC-compliant senders
do not emit ECT(1), and RFC-compliant middleboxes do not alter the
field to ECT(1), while RFC-compliant receivers interpret ECT(1)
identically to ECT(0).  These are useful properties which represent an
opportunity for improvement.

Meanwhile, experience gained with AQM in the field suggests that it is
difficult to maintain the desired 100% link utilisation, whilst
simultaneously strictly minimising induced delay due to excess queue
depth - irrespective of whether ECN is in use.  This leads to a
reluctance amongst hardware vendors to implement the most effective
AQM schemes because their headline benchmarks are throughput-based.

The underlying cause is the very sharp "multiplicative decrease"
reaction required of transport protocols to congestion signalling
(whether that be packet loss or CE marks), which tends to leave the
congestion window significantly smaller than the ideal BDP when
triggered at only slightly above the ideal value.  The availability of
this sharp response is required to assure network stability (AIMD
principle), but there is presently no standardised and
backwards-compatible means of providing a less drastic signal.

Given the confluence of these circumstances, we propose as a solution
the redefinition of ECT(1) as SCE, meaning "Some Congestion
Experienced".  The above ECN-field codepoint table would then become:

   Binary  Keyword                                  References
   ------  -------                                  ----------
     00     Not-ECT (Not ECN-Capable Transport)     [@RFC3168]
     01     SCE (Some Congestion Experienced)       [This Internet-draft]
     10     ECT (ECN-Capable Transport)             [@RFC3168]
     11     CE (Congestion Experienced)             [@RFC3168]

This permits middleboxes implementing AQM to signal incipient
congestion, below the threshold required to justify setting CE, by
converting some proportion of ECT codepoints to SCE ("SCE marking").
Existing receivers transparently ignore this new signal, and
existing middleboxes will still be able to convert SCE to CE as they
presently do for ECT, thus ensuring backwards compatibility.
Additionally, SCE-aware middleboxes will still produce CE markings as
at present, retaining current behaviour with Classic ECN endpoints.

Permitted ECN codepoint packet transitions by middleboxes:

   	Not-ECT ->   Not-ECT or DROP
   	ECT     ->   ECT or SCE or CE or DROP
   	SCE     ->   SCE or CE or DROP
   	CE      ->   CE or DROP

In other words, for ECN-aware flows, the ECN marking of an individual
packet MAY be increased by a middlebox to signal congestion, but MUST
NOT be decreased, and packets MUST NOT be altered to appear to be
ECN-aware if they were not originally, nor vice versa.  Note however
that SCE is numerically less than ECT, but semantically greater, and
the latter definition applies for this rule.

SCE-aware receivers and transport protocols must continue to apply the
[@RFC3168] interpretation of the CE codepoint, that is, to signal the
sender to back off send rate to the same extent as if a packet loss
were detected.  This maintains compatibility with existing
middleboxes.

SCE-aware receivers and transport protocols should interpret the SCE
codepoint as an indication of mild congestion, with the relative
incidence of ECT and SCE codepoints received indicating the relative
severity of such congestion, and respond accordingly by applying send
rates intermediate between those resulting from a continuous sequence
of ECT codepoints, and those resulting from a CE codepoint.

As a concrete example, consider a TCP transport implementing CUBIC
congestion control.  This presently exhibits exponential cwnd growth
during slow-start, polynomial cwnd growth in steady-state, and
multiplicative decrease (by 40%) upon detecting a single CE marking or
packet loss in one RTT cycle.  With SCE awareness, it might exit
slow-start upon detecting a single SCE marking, switch from polynomial
to Reno-linear cwnd growth when the SCE:ECT ratio exceeds 1:2, halt
cwnd growth entirely when it exceeds 1:1, and implement a Reno-linear
decline when it exceeds 2:1, in addition to retaining the sharp 40%
decrease on detecting CE.  In ideal circumstances, this would result
in the cwnd stabilising at a level which produces between 50% and 66%
SCE marking at some bottleneck on the path.

Details of how to implement SCE awareness at the transport layer will
be left to a separate proposal in the ELR series.

To maximise the benefit of SCE, middleboxes should be capable of
producing SCE markings considerably earlier than they presently
produce CE markings.  In particular, the ideal moment to signal the
exit of TCP slow-start (which typically doubles the cwnd per RTT) is
when the throughput of the flow reaches 50% of the bandwidth available
for it.  At this point, however, traditional AQM implementations see
only an empty queue and take no signalling action.

This is also a matter deserving more detailed attention in a separate
proposal in the ELR series.

This document therefore limits its scope to proposing the redefinition
of the ECT(1) codepoint as SCE, with some brief illustrations of how
it may be used.

---


Since the deprecation of the ECN Nonce

# History

# SCE flow diagram

```
if ECT(0) or ECT(1) is set
ECT(0) -> ECT(1) -> ECT(1) | ECT(0) (CE) -> DROP
```

# Implementation status

## SCE\_THRESHOLD

The existing CE\_THRESHOLD portion of FQ\_CODEL can be repurposed for this.

# Effects

Reciever - side has a couple advantages - we alredy do it -android clamps rwind.
That can decide or not, to send the ECT bit, based on it's local viewpoint of the congestion problem.

ECE bit

# Implementation guidelines

# Related Work

[@RFC8087] [@RFC7567] [@RFC7928] [@RFC8290] [@RFC8289]

# IANA Considerations

There are no IANA considerations.

# Security Considerations

There are no security considerations.

# Acknowledgements

{backmatter}

<reference anchor='IPv4CLEANUP' target='https://github.com/dtaht/ipv4-cleanup'>
<front>
<title>IPv4 cleanup project</title>
<author initials='D.' surname='Taht' fullname='Dave Taht'>
<address>
<email>dave@taht.net</email>
</address>
</author>
<date year='2019' />
</front>
</reference>


