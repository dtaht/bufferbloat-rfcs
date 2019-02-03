%%%
title = "Explicit Load Regulation"
abbrev = "ELR"
updates = [3168]
ipr = "trust200902"
area = "Internet"
docname = "draft-morton-taht-ELR"
workgroup = "Network Working Group"
submissiontype = "IETF"
keyword = [""]
#date = 2019-01-30T00:00:00Z

[seriesInfo]
name = "Internet-Draft"
value = "draft-morton-taht-ELR"
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

