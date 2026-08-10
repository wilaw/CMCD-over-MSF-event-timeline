---
title: "CMCD transmission over MSF Event Timeline"
abbrev: "CMCD over MSF"
category: std

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

...

--- abstract

Defines the transmission of CMCD data over MSF Event Timeline tracks.


--- middle

# Introduction

MOQT Streaming Format {{MSF}} defines Event Timeline tracks as a generic mechanism for
transmitting ad hoc data associated with MSF media tracks. CMCD {{CMCD}} defines a
means by which a media player can communicate structured data and have it processed consistently
by delivery networks and third-party data collection services. This draft specifies how an MSF
catalog can instruct a publisher to publish configurable CMCD data at a target destination using
MSF Event Timeline tracks.

This specification defines transmission for version 2 of CMCD.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# CMCD mode and key restrictions {#restrictions}

CMCD supports two reporting modes - Request Mode and Event Mode. Since Request mode is tied to
HTTP requests, which are not present in MOQT, this specification constrains CMCD usage to those
options offered in Event Mode. Request mode MUST NOT be used.

Furthermore, there are certain keys under Event Mode which reference request semantics and therefore
are not allowed in this specification. The keys which MUST NOT be used are: 'ab', 'cmsdd', 'cmsds',
'd', 'dl', 'h', 'lab', 'nor', 'nr', 'ot', 'rc', 'rtp', 'sf', 'smrt', 'su', 'tab', 'ttfb', 'ttfbb',
'ttlb' and 'url'.

The following keys MAY be used in Event Mode: 'bl', 'bg', 'br', 'bs', 'bsa', 'bsd', 'bsda', 'cen',
'cid', 'cs', 'dfa', 'e', 'ec', 'lb', 'ltc', 'msd', 'mtp', 'pb', 'pr', 'pt', 'sid', 'sn', 'st',
'sta', 'tb', 'tbl' and 'tpb'.

The Event key 'e' MUST NOT use the 'h' and 'rr' tokens. While the 'ts' data is mandatory for every
report, it is transmitted as the index value of each event timeline record.

When the CMCD spec refers to a 'manifest' in the description of a key, interpret that as the 'catalog'
in the context of this specification.

The version key 'v' MUST be present in each report and MUST carry a value of 2.

# Catalog requirements

## Publish tracks
A broadcaster triggers a catalog recipient to send CMCD data by including a 'publishTracks'
(see {{MSF}} Section 5.1.5) entry. That entry MUST point at a CMCD track {{cmcd-track}}. Multiple CMCD
tracks MAY be included in the publishTracks array in order to target data at different destinations.


## CMCD track {#cmcd-track}
A CMCD track defines a track which is to be published by the receiver of the catalog. It defines the
configuration of the CMCD data to be sent within the payload of that track, as well as the namespace
and name under which the track will be published. A CMCD track MUST only be included in a 'publishTracks'
array and MUST NOT be present in a 'tracks' array.

An MSF track carrying {{CMCD}} data MUST

* declare a packaging value of "eventtimeline".
* declare an eventType value of "urn:cta:cmcd:2026".
* be referenced in the 'publishTracks' array
* carry a 'cmcdConfig' {{cmcd-config}} custom track field.

A CMCD track is an Event Timeline track (See {{MSF}} Section 8). The "T" wall-clock time index MUST be used.
The 'data' field of each timeline entry is defined as a CMCD_DATA object {{cmcd-data}}.

Each CMCD track record MUST be published as a new Group unless batching is in effect, in which case multiple
records MUST be accumulated within each Group. The Group ID SHOULD begin at 0 and then increment by one for
the duration of the session.

Each CMCD track record MUST be published as soon as possible after the event which triggered it, unless batching
is in effect, in which case the batching constraints (see {{cmcd-config}}) apply.

### CMCD_DATA object {#cmcd-data}
The CMCD_DATA Object is a sequence of key-value pairs. The key is a string which matches one of the allowed
CMCD keys specified in {{restrictions}}. The value is always a string and it holds the CMCD-defined value
for that key. The value is NOT URLEncoded.

The 'ts' key MUST NOT appear in the CMCD_DATA Object. The value of the 'ts' key MUST be used as the "T" value
of the record index.



Example CMCD track record

~~~ json
{
        "T": 1756885678361,
        "data": {
            "sid":"6e2fb550-c457-11e9-bb97-0800200c9a66",
            "bl": 4068,
            "br": "(5000;v 320;a)",
            "bsd": 321,
            "e":"t",
            "sta": "p",
            "v": 2
        }
}
~~~

## Configuration {#cmcd-config}

This specification defines a new, custom MOQT track field, intended solely for use within a
CMCD track. The name of the field is "cmcdConfig" and the value is a JSON {{JSON}} object. The purpose of
this field is to instruct the publisher on which CMCD keys to send in the track, how to send them and
where to send them.

The JSON Object contains the following fields:

* "events": an array containing one or more triggering events. Each event MUST be in the set ('abs','abe','ae',
  'as','b','bc','c','ce','e','m','pc','pe','pr','ps','sk','t','um'). This array MUST NOT be empty.
* "interval": optional, the time interval in milliseconds between 't' events. This field MUST only be present if the
  't' event is included in the 'events' array. Intervals below 5000 SHOULD NOT be used.
* "enabledKeys": an array of the allowed CMCD keys to be included in the report. By default, all keys are excluded
  so each key MUST be explicitly enabled, with the exception of the 'v' key. The 'v' key is mandatory and MUST be
  included in each record even if it is not specified in the 'enabledKeys' array.
* "sid": optional, a session ID to use as the value of the 'sid' key. This field MUST only be present if the 'sid'
  key is included in the 'enabledKeys' array. If not provided and 'sid' is specified in the 'enabledKeys' array,
  then the publisher MUST synthesize its own session ID using the guidelines specified in {{CMCD}} Section 4.3.
* "batchCount": optional, the number of records to accrue before publishing a new Group.
* "batchInterval": optional, the minimum interval in milliseconds before publishing a new Group.

If both batchCount and batchInterval are specified, then the publisher MUST publish a new Group when either constraint
is first satisfied.

An example cmcdConfig track entry, instructing the publisher, every 30 seconds and at every playstate change, to
publish a CMCD record containing session ID (which has been provided), buffer length, bitrate, play state and
absolute buffer starvation CMCD data.

~~~ json
"cmcdConfig" : {
    "events": ["t","ps"],
    "interval": 30000,
    "enabledKeys": ["sid","bl","br","sta","bsa"],
    "sid":"6e2fb550-c457-11e9-bb97-0800200c9a66"
}
~~~

# Example catalog entry

This catalog excerpt shows the catalog instructing the publisher to send CMCD data to 3 different targets.
Each target receives a different set of CMCD keys:

 - The first target is a player monitoring system, which receives general health information every 30s.
 - The second target is a play state monitoring system. These reports are batched and then sent once a minute.
 - The third target is an error collection system, which receives only player errors.

Note the use of the MSF variable substitution scheme to pass down a custom track name. This is a useful mechanism
for giving each end-receiver a cacheable catalog while still allowing individualized reports.

~~~ json

"publishTracks": [
    {
      "namespace": "example.com/collection/cmcd/health",
      "name": "%player-id%",
      "packaging": "eventtimeline",
      "eventType": "urn:cta:cmcd:2026",
      "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
      "cmcdConfig" : {
         "events": ["t"],
         "interval": 30000,
         "enabledKeys": ["sid","bl","br","bs","ltc","msd","mtp"]
      }
    },
    {
      "namespace": "example.com/collection/cmcd/playstate",
      "name": "%player-id%",
      "packaging": "eventtimeline",
      "eventType": "urn:cta:cmcd:2026",
      "token": "kkjh68n3DVun23csFGk418kHNU5VDrwb...",
      "cmcdConfig" : {
         "events": ["ps"],
         "batchInterval": 60000,
         "enabledKeys": ["sid","ps"],
         "sid":"6e2fb550-c457-11e9-bb97-0800200c9a66"
      }
    },
    {
      "namespace": "example.com/errors",
      "name": "%player-id%",
      "packaging": "eventtimeline",
      "eventType": "urn:cta:cmcd:2026",
      "cmcdConfig" : {
         "events": ["e"],
         "enabledKeys": ["sid","ec"]
      }
    }
  ]

~~~


# Security Considerations

The carriage of CMCD signals within the MSF Event Timeline track inherits the security
considerations of both the underlying MSF transport and the CMCD standard itself.
CMCD data is sent by an untrusted player and may therefore be falsified.

The ability of a publishTrack to direct output at a third-party site could be used to DoS
both the sender and receiver of the data if the CMCD payload were high and the interval low.
Setting a 't' interval to 1, for example, would cause a very high report rate. Client implementers
SHOULD protect themselves from high report rates by coercing low values to a higher, sustainable
level.

# IANA Considerations

This document adds one entry to the "MSF Event Timeline Types" registry.

| Event Type            | Description            | Specification    |
|:----------------------|:-----------------------|:-----------------|
| urn:cta:cmcd:2026     | CMCD data transmission | This document    |



# Acknowledgments
Thanks to the IETF MOQ working group and the CTA WAVE CMCD working group for their review and input.
