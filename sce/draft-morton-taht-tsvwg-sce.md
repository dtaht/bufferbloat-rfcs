%%%
title = "The Some Congestion Experienced ECN Codepoint"
abbrev = "sceb"
updates = [3168, 8311]
obsoletes = [ ]
ipr = "trust200902"
area = "Internet"
docname = "draft-morton-taht-tsvwg-sce-00"
workgroup = "Transport Working Group"
submissiontype = "IETF"
keyword = [""]
#date = 2019-01-30T00:00:00Z

[seriesInfo]
name = "Internet-Draft"
value = "draft-morton-taht-tsvwg-sce-00"
stream = "IETF"
status = "standard"

[[author]]
initials = "J."
surname = "Morton"
fullname = "Jonathan Morton"
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
initials = "D.P."
surname = "Reed"
fullname = "David Reed"
#role = "editor"
organization = "DeepPlum Research"
  [author.address]
  email = "dpreed.ietf@teklibre.net"
  phone = "+18312059740"
  [author.address.postal]
  street = "20600 Aldercroft Heights Rd"
  city = "Los Gatos"
  region = "Ca"
  code = "95033"
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

This memo reclassifies ECT(1) to be an early notification of
congestion on ECT(0) marked packets, which can be used by AQM
algorithms and transports as an earlier signal of congestion than
CE. It is a simple, transparent, and backward compatible upgrade to
existing IETF-approved AQMs, RFC3168, and nearly all congestion
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
codepoint as SCE, "Some Congestion Experienced", with a few brief
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
practice and thus subsequently obsoleted by [@!RFC8311] (section
3). Additionally, known [@!RFC3168] compliant senders do not emit
ECT(1), and compliant middleboxes do not alter the field to ECT(1),
while compliant receivers all interpret ECT(1) identically to ECT(0).
These are useful properties which represent an opportunity for
improvement.

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
signal with respect to congestion control, and both existing and SCE-aware
middleboxes MAY convert SCE to CE in the same circumstances as for ECT, thus
ensuring backwards compatibility with [@RFC3168] ECN endpoints.

Permitted ECN codepoint packet transitions by middleboxes are:

```
   	Not-ECT ->   Not-ECT or DROP
   	ECT     ->   ECT or SCE or CE or DROP
   	SCE     ->   SCE or CE or DROP
   	CE      ->   CE or DROP
```

In other words, for ECN-aware flows, the ECN marking of an individual
packet MAY be increased by a middlebox to signal congestion, but MUST
NOT be decreased, and packets SHALL NOT be altered to appear to be
ECN-aware if they were not originally, nor vice versa.  Note however
that SCE is numerically less than ECT, but semantically greater, and
the latter definition applies for this rule.

New SCE-aware receivers and transport protocols SHALL continue to apply
the [@RFC3168] interpretation of the CE codepoint, that is, to signal
the sender to back off send rate to the same extent as if a packet
loss were detected.  This maintains compatibility with existing
middleboxes, senders and receivers.

New SCE-aware receivers and transport protocols SHOULD interpret the SCE
codepoint as an indication of mild congestion, and respond accordingly by
applying send rates intermediate between those resulting from a continuous
sequence of ECT codepoints, and those resulting from a CE codepoint.  The
ratio of ECT and SCE codepoints received indicates the relative severity
of such congestion, such that 100% SCE is very close to the threshold of
CE marking, 100% ECT indicates that the bottleneck link may not be fully
utilised, and some mixture of ECT and SCE codepoints indicates that the
present send rate is a good match to the bottleneck link.

Details of how to implement SCE awareness at the transport layer will
be left to additional Internet Drafts yet to be submitted.  To ensure
RTT-fair convergence with single-queue SCE AQMs, transports SHOULD stabilise
at higher SCE ratios for higher BDPs, and MAY reduce their response to CE
marks IFF they are responding to SCE signals received at around the same time
(eg. in adjacent packets, or at least during the same RTT) in the same flow.

To maximise the benefit of SCE, middleboxes SHOULD produce SCE markings
sooner than they produce CE markings, when the level of congestion increases.

# Examples of use

## Codel-type AQMs

A simple and natural way to implement SCE in a Codel-type AQM is to mark all
ECT packets as SCE if they are over half the Codel target sojourn time, and not
marked CE by Codel itself.  This threshold function does not necessarily
produce the best performance, but is very easy to implement and provides useful
information to SCE-aware flows, often sufficient to avoid receiving CE marks
whilst still efficiently using available capacity.

For a more sophisticated approach avoiding even small-scale oscillation, a
stochastic ramp function may be implemented with 100% marking at the Codel
target, falling to 0% marking at or above zero sojourn time.  The lower point
of the ramp should be chosen so that SCE is not accidentally signalled due to
CPU scheduling latencies or serialisation delays of single packets.  Absent
rigorous analysis of these factors, setting the lower limit at half the Codel
target should be safe in many cases.

The default configuration of Codel is 100ms interval, 5ms target.  A typical
ramp function for these parameters might cease marking below 2.5ms sojourn
time, increase marking probability linearly to 100% at 5ms, and mark at 100%
for sojourn times above 5ms (in which CE marking is also possible).

Flow-isolating AQMs should avoid signalling SCE to flows classified as "sparse"
in order to encourage the fastest possible convergence to the fair share.

For the avoidance of doubt, a decision to mark CE or to drop a packet always
takes precedence over SCE marking.

## RED-type AQMs (including PIE)

There are several reasonable methods of producing SCE signals in a RED-type AQM.

The simplest would be a threshold function, giving a hard boundary in queue depth
between 0% and 100% SCE marking.  This could be a sensible option for limited
hardware implementations.  The threshold should be set below the point at which
a growing queue might trigger CE marking or packet drops.

Another option would be to implement a second marking probability function,
occupying a queue-depth space just below that occupied by the main marking
probability function.  This should be arranged so that high marking rates
(ideally 100%) are achieved at or before the point at which CE marking or
packet drops begin.

For PIE specifically, a second marking probability function could be added with
the same parameters as the main marking probability function, except for a lower
QDELAY_REF value.  This would result in the SCE marking probability remaining
strictly higher than the CE marking probability for ECT flows.

## TCP

Some mechanism should be defined to feed back SCE signals to the sender
explicitly.  Details of this are left to future I-Ds covering TCP in detail;
use could be made of the redundant NS bit in the TCP header, which was formerly
associated with ECT(1) in the Nonce Sum specification.

Alternatively, SCE can potentially be handled entirely by the receiver, and
thus be entirely independent of any of the dozens of [@RFC3168] compliant
congestion control algorithms on the sender side.  This would be done
by adjusting the Receive Window, which has been a standard part of TCP
from its inception.  This alternative therefore requires the minimum amount
of protocol changes on the wire.

## Other

New transports under development, such as QUIC, may implement a fine-grained
signal back to the sender based on SCE.
QUIC itself appears to have this sort of feedback already (counting ECT(0), ECT(1)
and CE packets received), and the data should be made available for congestion control.

# Related Work

[@RFC8087] [@RFC7567] [@RFC7928] [@RFC8290] [@RFC8289] [@RFC8033] [@RFC8034]

# IANA Considerations

There are no IANA considerations.

# Security Considerations

An adversary could inappropriately set SCE marks at middleboxes he controls to slow
down SCE-aware flows, eventually reaching a minimum congestion window.  However,
the same threat already exists with respect to inappropriately setting CE marks on
normal ECN flows, and this would have a greater impact per mark.  Therefore no new
threat is exposed by SCE in practice.

An adversary could also simply ignore SCE marks at the receiver, or ignore SCE
information fed back from the receiver to the sender, in an attempt to gain some
advantage in throughput.  Again, the same could be said about ignoring CE marks, so
no truly new threat is exposed.  Additionally, correctly implemented SCE detection
may actually improve long-term goodput compared to ignoring SCE.

An adversary could erase congestion information by converting SCE marks to ECT or
Not-ECT codepoints, thus hiding it from the receiver.  This has equivalent effects
to ignoring SCE signals at the receiver.  An identical threat already exists for
erasing congestion information from CE marked packets, and may be mitigated by AQMs
switching to dropping packets from flows observed to be non-responsive to CE.

# Acknowledgements

Many thanks to John Gilmore, the members of the ecn-sane project and the cake@lists.bufferbloat.net mailing list, and the former IETF AQM working group.

{backmatter}
