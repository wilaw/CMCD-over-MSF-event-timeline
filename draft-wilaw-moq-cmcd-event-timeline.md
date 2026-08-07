---
title: "CMCD transmission over MSF Event Timeline"
abbrev: "CMCD over MSF"
category: info

docname: draft-wilaw-moq-cmcd-event-timeline-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: "Web and Internet Transport"
workgroup: "Media Over QUIC"
keyword:
 - MSF
 - Eventtimeline
 - SCTE35
 - MOQT
venue:
  group: "Media Over QUIC"
  type: "Working Group"
  mail: "moq@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/moq/"
  github: "wilaw/CMCD-over-MSF-event-timeline"
  latest: "https://wilaw.github.io/CMCD-over-MSF-event-timeline/draft-wilaw-moq-cmcd-event-timeline.html"

author:
  - fullname: Will Law
    organization: Akamai
    email: "wilaw@akamai.com"

normative:
  MSF: I-D.draft-ietf-moq-msf-01
  JSON: RFC8259
  CMCD:
    title: "Common Media Client Data (CTA-5004-B)"
    date: 2026
    target: https://cta-wave.github.io/Resources/common-media-client-data--cta-5004-b.html

informative:

...

--- abstract

Defines the transmission of CMCD data over MSF Event Timeline tracks.


--- middle

# Introduction

MOQT Streaming Format {{MSF}} defines Event Timeline tracks as a generic mechanism for
transmitting ad hoc data associated with MSF media tracks. CMCD {{CMCD}} defines outlines a
means by which a media player can communicate structured data and have it processed consistently
by delivery networks and third-party data collection services. This draft specifies how an MSF
catalog can instruct a publisher to publish configurable CMCD data at a target destination using
MSF Event Timeline tracks.

This specification

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# CMCD mode and key restrictions

CMCD supports two reporting modes - Request Mode and Event Mode. Since Request mode is tied to
HTTP requests, which are not present in MOQT, this specification constrains CMCD usage to those
options offered in Event Mode. Request mode MUST NOT be used.

Furthermore, there are certain keys under Event Mode which reference request semantics and therefore
are not allowed in this specification. The keys which MUST NOT be used are: 'ab', 'cmsdd', 'cmsds',
'd', 'dl', 'h', 'lab', 'nor', 'nr', 'ot', 'rc', 'rtp', 'sf', 'smrt', 'su','tab', 'ttfb', 'ttfbb',
'ttlb' and 'url'.

The following keys MAY be used in Event Mode: 'bl', 'bg', 'br', 'bs', 'bsa', 'bsd', 'bsda', 'cen',
'cid', 'cs', 'dfa', 'e', 'ec', 'lb', 'ltc', 'msd', 'mtp', 'pb', 'pr', 'pt', 'sid', 'sn', 'st',
'sta', 'tb', 'tbl', 'tpb' and 'ts'.

The Event key 'e' MUST not use the 'h' and 'rr' tokens.

When the CMCD spec refers to a 'manifest' in the description of key, interpret that as the 'catalog'
in the context of this specification.

The version key 'v' MUST be present in each report and MUST carry a value of 2.

# Catalog requirements

## Publish tracks
A broadcaster triggers a catalog recipient to send CMCD data by including a 'publishTracks'
(see {{MSF}} Sect 5.1.5) entry. That entry MUST point at a CMCD track {{cmcd-track}}. Multiple CMCD
tracks MAY be included in the publishtracks array in order to target data at different destinations.


## CMCD track {#cmcd-track}
A CMCD track defines a track which is to be published by the receiver of the catalog. It defines the
configuration of the CMCD data to be sent within the payload of that track, as well as the namespace
and name under which the track will be published.

An MSF track carrying {{CMCD}} data MUST

* declare a packaging value of "eventtimeline".
* declare an eventType value of "urn:cta:cmcd:2026".
* be referenced in the 'publishTracks' array
* carry a 'cmcd-config' {{cmcd-config}} custom track field.

## Configuration {#cmcd-config}

This specification defines a new, custom MOQT track field, intended solely for use within a
CMCD track. The name of the field is "cmcd-config" and the value is a JSON object. The purpose of
this field is to instruct the publisher on which CMCD keys to send in the track, how to send them and
where to send them.

The JSON Object contains the following fields:

* 'events': an array of

## Batching



# Examples

To illustrate the implementation of SCTE 35 within the MSF Event Timeline track, the following
examples demonstrate the mapping of various timing sources — using MSF's native t, l, and m index
reference fields — and the two supported encoding formats (Binary and XML).

## Example 1: Binary Encoding with PTS Timing

This example shows the standard frame-accurate splice using a pts_time. Per Section 3.2 case 1,
the record uses m, the calculated millisecond offset, and the payload is the Base64-encoded binary
splice_info_section().

~~~ json
TBD
~~~




# Security Considerations

The carriage of CMCD signals within the MSF Event Timeline track inherits the security
considerations of both the underlying MSF transport and the CMCD standard itself.




# IANA Considerations

This document adds one entry to the "MSF Event Timeline Types" registry.

| Event Type            | Description            | Specification    |
|:======================|:=======================|:=================|
| urn:cta:cmcd:2026     | CMCD data transmission | This document    |



# Acknowledgments
The IETF moq workgroup and the CTA WAVE CMCD workgroup.
