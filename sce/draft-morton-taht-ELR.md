%%%
title = "Explicit Load Regulation"
abbrev = "ELR"
updates = [3168]
ipr = "trust200902"
area = "Internet"
docname = "draft-morton-taht-ELR-00"
workgroup = "Network Working Group"
submissiontype = "IETF"
keyword = [""]
#date = 2019-01-30T00:00:00Z

[seriesInfo]
name = "Internet-Draft"
value = "draft-morton-taht-ELR-00"
stream = "IETF"
status = "standard"

[[author]]
initials = "J."
surname = "Morton"
fullname = "Jonathon Morton"
#role = "editor"
organization = "Bufferbloat.net"
  [author.address]
  email = "chromi@gmail.com"
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
role = "editor"
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

This memo outlines how the some congestion experienced (SGE) bit can be
used as an earlier indicator of congestion than the ECN CE marking in
several common AQM algoritms and reciever side transports.

It is transparent, backward
compatible upgrade to existing AQM algorithms such as
RED/PIE/FQ_PIE/CODEL/FQ_CODEL, and newer protocols such as QUIC.

{mainmatter}

# Terminology

The keywords **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL**, when they appear in this document, are to be interpreted as described in [@!RFC2119].

# Introduction
at we don't *only* have 2 bits to work with, but a whole sequence of packets carrying these 2-bit codepoints.  We can convey fine-grained information by setting codepoints stochastically or in a pattern, rather than by merely choosing one of the three available (ignoring Not-ECT).  The receiver can then observe the density of codepoints and report that to the sender.

Which is more-or-less the premise of DCTCP.  However, DCTCP changes the meaning of CE, instead of making use of ECT(1), which I think is the big mistake that makes it undeployable.

So, from the middlebox perspective, very little changes.  ECN-capable packets still carry ECT(0) or ECT(1).  You still set CE on ECT packets, or drop Non-ECT packets, to signal when a serious level of persistent queue has developed, so that the sender needs to back off a lot.  But if a less serious congestion condition exists, you can now signal *that* by changing some proportion of ECT(0) codepoints to ECT(1), with the intention that senders either reduce their cwnd growth rate, halt growth entirely, or enter a gradual decline.  Those are three things that ECN cannot currently signal.

This change is invisible to existing, RFC-compliant, deployed middleboxes and endpoints, so should be completely backwards-compatible and incrementally deployable in the network.  (The only thing it breaks is the optional ECN integrity RFC that, according to fairly recent measurements, literally nobody bothered implementing.)

Through TCP Timestamps, both sender and receiver can know fairly precisely when a round-trip has occurred.  The receiver can use this information to calculate the ratio of ECT(0) and ECT(1) codepoints received in the most recent RTT.  A new TCP Option could replace TCP Timestamps and the two bytes of padding that usually go with it, allowing reporting of this ratio without actually increasing the size of the TCP header.  Large cwnds can be accommodated at the receiver by shifting both counters right until they both fit in a byte each; it is the ratio between them that is significant.

It is then incumbent on the sender to do something useful with that information.  A reasonable idea would be to aim for a 1:1 ratio via an integrating control loop.  Receipt of even one ECT(1) signal might be considered grounds for exiting slow-start, while exceeding 1:2 ratio should limit growth rate to "Reno linear" semantics (significant for CUBIC), and exceeding 2:1 ratio should trigger a "Reno linear" *decrease* of cwnd.  Through all this, a single CE mark (reported in the usual way via ECE and CWR) still has the usual effect of a multiplicative decrease.

That's my proposal.

# SGE flow diagram

```
ECT(0) -> ECT(1) -> ECT(1) | ECT(0) (CE) -> DROP
```

# Altering AQM algorithms

The existing CE_THRESHOLD portion of FQ_CODEL can be repurposed for this.

# Improving sender side TCP algorithms

# Improving receiver side TCP handling

# Adding multi-bit congestion control to QUIC

# RTP changes

# ELR
A brief note on ELR:

The name "Explicit Load Regulation" comes from the action of the Load Regulator in older types of diesel-electric locomotive.  This senses the torque load on the engine (through the amount of fuel injection needed to maintain constant speed) and adjusts the excitation of the main generator to hold engine torque constant.  A typical model of the 1960s has "fast up" and "fast down" segments to handle rapid changes in load, "slow up" and "slow down" segments for fine adjustments, and a "hold" segment for constant-speed cruising.

Until now, there has been no standardised equivalent to "hold" or "slow down" in TCP congestion control.  In Reno terms, the exponential growth of slow-start is "fast up", the linear growth of steady-state is "slow up", and the multiplicative-decrease in response to packet loss or a CE mark is "fast down".  Other loss-based TCPs, such as CUBIC, differ in quantitive detail but are qualitatively similar.

The result has been a system which inherently oscillates around some steady state instead of settling on it.  The send rate is at times too high and induces latency through building queues, and at other times too low and wastes path capacity.  ELR is an attempt to adjust this system into one which does settle into an ideal steady state.

# Implementation guidelines

# Related Work

# IANA Considerations

There are no IANA considerations.

# Security Considerations

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

<reference anchor='I.D.WILSON10' target='https://tools.ietf.org/id/draft-wilson-class-e-02'>
<front>
<title>Redesignation of 240/4 from "Future Use" to "Private Use"</title>
<author initials='G.' surname='Huston' fullname='Geoff Huston'>
<address>
<email>gih@apnic.net</email>
</address>
</author>
<author initials='G.' surname='Michaelson' fullname='George Michaelson'>
<email>ggm@apnic.net</email>
</author>
<author initials='P.' surname='Wilson' fullname='Paul Wilson'>
<email>pwilson@apnic.net</email>
</author>
<date year='2010' />
</front>
</reference>

<reference anchor='I.D.FULLER08' target='https://tools.ietf.org/id/draft-fuller-240space-02.txt'>
<front>
<title>240 address space</title>
<author initials='V.' surname='Fuller' fullname='Vince Fuller'>
<address>
<email>vince.fuller@gmail.com </email>
</address>
</author>

<author initials='E.' surname='Lear' fullname='Elliot Lear'></author>
<author initials='D.' surname='Meyer' fullname='David Meyer'></author>
<date year='2008' />
</front>
</reference>

