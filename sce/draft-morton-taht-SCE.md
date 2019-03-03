%%%
title = "The Some Congestion Experienced Bit"
abbrev = "sceb"
updates = [3168, 8289, 8290, 8033, 8034]
ipr = "trust200902"
area = "Internet"
docname = "draft-morton-taht-SCE"
workgroup = "Transport Working Group"
submissiontype = "IETF"
keyword = [""]
#date = 2019-01-30T00:00:00Z

[seriesInfo]
name = "Internet-Draft"
value = "draft-morton-taht-SCE"
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

This memo reclassifies ECT(0) to be an early notification of congestion
on ECT(1) marked packets, which can be used by AQM algorithms and transports as an earlier signal of congestion than CE.. It is transparent, backward
compatible upgrade to the existing AQM algorithms such as
RED[@RFC2309], PIE [@RFC8033], [@RFC8034], FQ_PIE, CODEL [@RFC8289], and FQ_CODEL[@RFC8290].

{mainmatter}

# Introduction

# Terminology

The keywords **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL**, when they appear in this document, are to be interpreted as described in [@!RFC2119].

# Introduction

Since the deprecation of the ECN Nonce

# SCE flow diagram

```
if ECT(0) or ECT(1) is set
ECT(0) -> ECT(1) -> ECT(1) | ECT(0) (CE) -> DROP
```

# Implementation status

The existing CE_THRESHOLD portion of FQ_CODEL can be repurposed for this.

# Efficts

#


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


