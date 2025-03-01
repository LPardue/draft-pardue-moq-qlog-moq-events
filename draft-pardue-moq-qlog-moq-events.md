---
title: "MoQ qlog event definitions"
category: info

docname: draft-pardue-moq-qlog-moq-events-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: ""
workgroup: "Media Over QUIC"
keyword:
 - hi fidelity
venue:
  group: "Media Over QUIC"
  type: ""
  mail: "moq@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/moq/"
  github: "LPardue/draft-pardue-moq-qlog-moq-events"
  latest: "https://LPardue.github.io/draft-pardue-moq-qlog-moq-events/draft-pardue-moq-qlog-moq-events.html"

author:
 -
    fullname: Lucas Pardue
    organization: Cloudflare
    email: lucas@lucaspardue.com

normative:

  QLOG-MAIN:
    I-D.ietf-quic-qlog-main-schema

  QLOG-QUIC:
    I-D.ietf-quic-qlog-quic-events

informative:


--- abstract

This document defines a qlog event schema containing concrete events for MoQ.


--- middle

# Introduction
This document defines a qlog event schema ({{Section 8 of QLOG-MAIN}})
containing concrete events for Media over QUIC Transport {{!MOQT=I-D.ietf-moq-transport}}.

The event namespace with identifier `moqt` is defined; see {{schema-def}}. In
this namespace multiple events derive from the qlog abstract Event class
({{Section 7 of QLOG-MAIN}}), each extending the "data" field and defining
their "name" field values and semantics.

{{events-table}} summarizes the name value of each event type that is defined in
this specification. Some event data fields use complex data types. These are
represented as enums or re-usable definitions, which are grouped together on the
bottom of this document for clarity.

| Name value                          | Importance |  Definition |
|:------------------------------------|:-----------|:------------|
| moqt:control_message_created        | Core       | {{controlmessagecreated}} |
| moqt:control_message_parsed         | Core       | {{controlmessageparsed}} |
| moqt:stream_type_set                | Core       | {{streamtypeset}} |
| moqt:object_datagram_created        | Core       | {{objectdatagramcreated}} |
| moqt:object_datagram_parsed         | Core       | {{objectdatagramparsed}} |
| moqt:object_datagram_status_created | Core       | {{objectdatagramstatuscreated}} |
| moqt:object_datagram_status_parsed  | Core       | {{objectdatagramstatusparsed}} |
| moqt:subgroup_header_created        | Core       | {{subgroupheadercreated}} |
| moqt:subgroup_header_parsed         | Core       | {{subgroupheaderparsed}} |
| moqt:subgroup_object_created        | Core       | {{subgroupobjectcreated}} |
| moqt:subgroup_object_parsed         | Core       | {{subgroupobjectparsed}} |
| moqt:fetch_header_created           | Core       | {{fetchheadercreated}} |
| moqt:fetch_header_parsed            | Core       | {{fetchheaderparsed}} |
| moqt:fetch_object_created           | Core       | {{fetchobjectcreated}} |
| moqt:fetch_object_parsed            | Core       | {{fetchobjectparsed}} |
{: #events-table title="MOQT Events"}

When any event from this document is included in a qlog trace, the
"protocol_types" qlog array field MUST contain an entry with the value "MOQT":

~~~ cddl
$ProtocolType /= "MOQT"
~~~
{: #protocoltype-extension-moqt title="ProtocolType extension for MOQT"}

## Usage with QUIC

The events described in this document can be used with or without logging the
related QUIC events defined in {{QLOG-QUIC}}. If used with QUIC events, the QUIC
document takes precedence in terms of recommended filenames and trace separation
setups.

If used without QUIC events, it is recommended that the implementation assign a
globally unique identifier to each MOQT session. This ID can then be used as the
value of the qlog "group_id" field, as well as the qlog filename or file
identifier, potentially suffixed by the vantagepoint type (For example,
abcd1234_server.qlog would contain the server-side trace of the connection with
GUID abcd1234).

# Conventions and Definitions

{::boilerplate bcp14-tagged}

The event and data structure definitions in ths document are expressed
in the Concise Data Definition Language {{!CDDL=RFC8610}} and its
extensions described in {{QLOG-MAIN}}.

The following fields from {{QLOG-MAIN}} are imported and used: name, namespace,
type, data, group_id, protocol_types, importance, RawInfo, and time-related
fields.

As is the case for {{QLOG-MAIN}}, the qlog schema definitions in this document
are intentionally agnostic to serialization formats. The choice of format is an
implementation decision.

# Event Schema Definition {#schema-def}

This document describes how the MOQT protocol is expressed in qlog with an event
schema. Per the requirements in {{Section 8 of QLOG-MAIN}}, this document
registers the `moqt` namespace. The event schema URI is
`urn:ietf:params:qlog:events:moqt`.

## Draft Event Schema Identification
{:removeinrfc="true"}

Only implementations of the final, published RFC can use the events belonging to
the event schema with the URI `urn:ietf:params:qlog:events:moqt`. Until such an
RFC exists, implementations MUST NOT identify themselves using this URI.

Implementations of draft versions of the event schema MUST append the string
"-" and the corresponding draft number to the URI. For example, draft 99 of this
document is identified using the URI `urn:ietf:params:qlog:events:moqt-99`.

The namespace identifier itself is not affected by this requirement.

# MOQT Events

MOQT events extend the `$ProtocolEventData` extension point defined in
{{QLOG-MAIN}}. Additionally, they allow for direct extensibility by their use of
per-event extension points via the `$$` CDDL "group socket" syntax, as also
described in {{QLOG-MAIN}}.

~~~ cddl
MOQTEventData = MOQTControlMessageCreated /
                MOQTControlMessageParsed /
                MOQTStreamTypeSet /
                MOQTObjectDatagramCreated /
                MOQTObjectDatagramParsed /
                MOQTObjectDatagramStatusCreated /
                MOQTObjectDatagramStatusParsed /
                MOQTSubgroupHeaderCreated /
                MOQTSubgroupHeaderParsed /
                MOQTSubgroupObjectCreated /
                MOQTSubgroupObjectParsed /
                MOQTFetchHeaderCreated /
                MOQTFetchHeaderParsed /
                MOQTFetchObjectCreated /
                MOQTFetchObjectParsed

$ProtocolEventData /= MOQTEventData
~~~
{: #moqt-events-def title="MOQTEventData definition and ProtocolEventData
extension"}

MOQT events are logged when a certain condition happens at the application
layer, and there isn't always a one to one mapping between HTTP and QUIC events.
The exchange of data between the HTTP and QUIC layer is logged via the
"stream_data_moved" and "datagram_data_moved" events in {{QLOG-QUIC}}.

The concrete MOQT event types are further defined below, their type identifier
is the heading name.

## control_message_created {#controlmessagecreated}

The `control_message_created` event is emitted when a control message is created.
It has Core importance level; see {{Section 9.2 of QLOG-MAIN}}.

The definition of control message content is in {{moqtcontrolmessage}}.

~~~ cddl
MOQTControlMessageCreated = {
    stream_id: uint64
    ? length: uint64
    message: $MOQTControlMessage
    ? raw: RawInfo

    * $$moqt-controlmessagecreated-extension
}
~~~
{: #moqt-controlmessagecreated-def title="MOQTControlMessageCreated definition"}

## control_message_parsed {#controlmessageparsed}

The `control_message_parsed` event is emitted when a control message is parsed.
It has Core importance level; see {{Section 9.2 of QLOG-MAIN}}.

The definition of control message content is in {{moqtcontrolmessage}}.

~~~ cddl
MOQTControlMessageParsed = {
    stream_id: uint64
    ? length: uint64
    message: $MOQTControlMessage
    ? raw: RawInfo

    * $$moqt-controlmessageparsed-extension
}
~~~
{: #controlmessageparsed-def title="MOQTControlMessageParsed definition"}

## stream_type_set {#streamtypeset}

The `stream_type_set` event conveys when a MOQT stream type becomes known. It
has Base importance level; see {{Section 9.2 of QLOG-MAIN}}.

~~~ cddl
MOQTStreamTypeSet = {
    ? owner: Owner
    stream_id: uint64
    stream_type: $MOQTStreamType

    * $$moqt-streamtypeset-extension
}

$MOQTStreamType /=   "subgroup_header" /
                     "fetch_header"
~~~
{: #streamtypeset-def title="MOQTStreamTypeSet definition"}

## object_datagram_created {#objectdatagramcreated}

The `object_datagram_created` event is emitted when the OBJECT_DATAGRAM message
is created. It has Core importance level; see {{Section 9.2 of QLOG-MAIN}}.

~~~ cddl
MOQTObjectDatagramCreated = {
    track_alias: uint64
    group_id: uint64
    object_id: uint64
    publisher_priority: uint8
    extension_count: uint64
    ? extension_headers: [* MOQTExtensionHeader]
    object_payload_length: uint64
    ? object_status: uint64
    object_payload: RawInfo

    * $$moqt-objectdatagramcreated-extension
}
~~~
{: #objectdatagramcreated-def title="MOQTObjectDatagramCreated definition"}

## object_datagram_parsed {#objectdatagramparsed}

The `object_datagram_parsed` event is emitted when the OBJECT_DATAGRAM message
is parsed. It has Core importance level; see {{Section 9.2 of QLOG-MAIN}}.

~~~ cddl
MOQTObjectDatagramParsed = {
    track_alias: uint64
    group_id: uint64
    object_id: uint64
    publisher_priority: uint8
    extension_count: uint64
    ? extension_headers: [* MOQTExtensionHeader]
    object_payload_length: uint64
    ? object_status: uint64
    ? object_payload: RawInfo

    * $$moqt-objectdatagramparsed-extension
}
~~~
{: #objectdatagramparsed-def title="MOQTObjectDatagramParsed definition"}

## object_datagram_status_created {#objectdatagramstatuscreated}

The `object_datagram_status_created` event is emitted when the
OBJECT_DATAGRAM_STATUS message is created. It has Core importance level; see
{{Section 9.2 of QLOG-MAIN}}.

~~~ cddl
MOQTObjectDatagramStatusCreated = {
    track_alias: uint64
    group_id: uint64
    object_id: uint64
    publisher_priority: uint8
    object_status: uint64

    * $$moqt-objectdatagramstatuscreated-extension
}
~~~
{: #objectdatagramstatuscreated-def title="MOQTObjectDatagramStatusCreated definition"}

## object_datagram_status_parsed {#objectdatagramstatusparsed}

The `object_datagram_status_parsed` event is emitted when the
OBJECT_DATAGRAM_STATUS message is parsed. It has Core importance level; see
{{Section 9.2 of QLOG-MAIN}}.

~~~ cddl
MOQTObjectDatagramStatusParsed = {
    track_alias: uint64
    group_id: uint64
    object_id: uint64
    publisher_priority: uint8
    object_status: uint64

    * $$moqt-objectdatagramstatusparsed-extension
}
~~~
{: #objectdatagramstatusparsed-def title="MOQTObjectDatagramStatusParsed definition"}

## subgroup_header_created {#subgroupheadercreated}

The `subgroup_header_created` event is emitted when a stream begins and a
SUBGROUP_HEADER is created. It has Core importance level; see {{Section 9.2 of
QLOG-MAIN}}.

~~~ cddl
MOQTSubgroupHeaderCreated = {
    stream_id: uint64
    track_alias: uint64
    group_id: uint64
    subgroup_id: uint64
    publisher_priority: uint8

    * $$moqt-subgroupheadercreated-extension
}
~~~
{: #subgroupheadercreated-def title="MOQTSubgroupHeaderCreated definition"}

## subgroup_header_parsed {#subgroupheaderparsed}

The `subgroup_header_parsed` event is emitted when the SUBGROUP_HEADER is
parsed. It has Core importance level; see {{Section 9.2 of QLOG-MAIN}}.

~~~ cddl
MOQTSubgroupHeaderParsed = {
    stream_id: uint64
    track_alias: uint64
    group_id: uint64
    subgroup_id: uint64
    publisher_priority: uint8

    * $$moqt-subgroupheaderparsed-extension
}
~~~
{: #subgroupheaderparsed-def title="MOQTSubgroupHeaderParsed definition"}

## subgroup_object_created {#subgroupobjectcreated}

The `subgroup_object_created` event is emitted when a subgroup object is
created. It has Core importance level; see {{Section 9.2 of QLOG-MAIN}}.

~~~ cddl
MOQTSubgroupObjectCreated = {
    stream_id: uint64
    object_id: uint64
    extension_count: uint64
    ? extension_headers: [* MOQTExtensionHeader]
    object_payload_length: uint64
    ? object_status: uint64
    ? object_payload: RawInfo

    * $$moqt-subgroupobjectcreated-extension
}
~~~
{: #subgroupobjectcreated-def title="MOQTSubgroupObjectCreated definition"}

## subgroup_object_parsed {#subgroupobjectparsed}

The `subgroup_object_parsed` event is emitted when a subgroup object is parsed.
It has Core importance level; see {{Section 9.2 of QLOG-MAIN}}.

~~~ cddl
MOQTSubgroupObjectParsed = {
    stream_id: uint64
    object_id: uint64
    extension_count: uint64
    ? extension_headers: [* MOQTExtensionHeader]
    object_payload_length: uint64
    ? object_status: uint64
    ? object_payload: RawInfo

    * $$moqt-subgroupobjectparsed-extension
}
~~~
{: #subgroupobjectparsed-def title="MOQTSubgroupObjectParsed definition"}

## fetch_header_created {#fetchheadercreated}

The `fetch_header_created` event is emitted when a stream begins and a
FETCH_HEADER is created. It has Core importance level; see {{Section 9.2 of
QLOG-MAIN}}.

~~~ cddl
MOQTFetchHeaderCreated = {
    stream_id: uint64
    subscribe_id: uint64

    * $$moqt-fetchheadercreated-extension
}
~~~
{: #fetchheadercreated-def title="MOQTFetchHeaderCreated definition"}

## fetch_header_parsed {#fetchheaderparsed}

The `fetch_header_parsed` event is emitted when the SUBGROUP_HEADER is
parsed. It has Core importance level; see {{Section 9.2 of QLOG-MAIN}}.

~~~ cddl
MOQTFetchHeaderParsed = {
    stream_id: uint64
    subscribe_id: uint64

    * $$moqt-fetchheaderparsed-extension
}
~~~
{: #fetchheaderparsedd-def title="MOQTFetchHeaderParsed   definition"}

## fetch_object_created {#fetchobjectcreated}

The `fetch_object_created` event is emitted when a fetch object is created. It
has Core importance level; see {{Section 9.2 of QLOG-MAIN}}.

~~~ cddl
MOQTFetchObjectCreated = {
    stream_id: uint64
    group_id: uint64
    subgroup_id: uint64
    object_id: uint64
    publisher_priority: uint8
    extension_count: uint64
    ? extension_headers: [* MOQTExtensionHeader]
    object_payload_length: uint64
    ? object_status: uint64
    ? object_payload: RawInfo

    * $$moqt-fetchobjectcreated-extension
}
~~~
{: #fetchobjectcreated-def title="MOQTFetchObjectCreated definition"}

## fetch_object_parsed {#fetchobjectparsed}

The `fetch_object_parsed` event is emitted when a fetch object is parsed. It has
Core importance level; see {{Section 9.2 of QLOG-MAIN}}.

~~~ cddl
MOQTFetchObjectParsed = {
    stream_id: uint64
    group_id: uint64
    subgroup_id: uint64
    object_id: uint64
    publisher_priority: uint8
    extension_count: uint64
    ? extension_headers: [* MOQTExtensionHeader]
    object_payload_length: uint64
    ? object_status: uint64
    ? object_payload: RawInfo

    * $$moqt-fetchobjectparsed-extension
}
~~~
{: #fetchobjectparsed-def title="MOQTFetchObjectParsed definition"}

# MOQT Data Type Definitions

The following data type definitions can be used in MOQT events.

## Owner

~~~ cddl
Owner = "local" /
        "remote"
~~~
{: #owner-def title="Owner definition"}

## MOQTParameter

~~~ cddl
MOQTParameter = {
  name: $MOQTParameterName
  length: uint64
  ? value: RawInfo
}

$MOQTParameterName /= "authorization_info" /
                    "delivery_timeout" /
                    "max_cache_duration" /
                    "path" /
                    "max_subscribe_id"

~~~
{: #moqtparameter-def title="MOQTParameter definition"}


## MOQTControlMessage

The generic `$MOQTControlMessage` is defined here as a CDDL "type socket" extension point.
It can be extended to support additional MOQT control message types.

~~~
; The MOQTControlMessage is any key-value map (e.g., JSON object)
$MOQTControlMessage /= {
    * text => any
}
~~~
{: #control-message-def title="MOQTControlMessage type socket definition"}

The MOQT control message types defined in this document are as follows:

~~~ cddl
MOQTBaseControlMessages = MOQTClientSetupMessage /
                          MOQTServerSetupMessage /
                          MOQTGoaway /
                          MOQTSubscribe /
                          MOQTSubscribeUpdate /
                          MOQTUnsubscribe /
                          MOQTFetch /
                          MOQTFetchCancel /
                          MOQTAnnounceOk /
                          MOQTAnnounceError /
                          MOQTAnnounceCancel /
                          MOQTTrackStatusRequest /
                          MOQTSubscribeAnnounces /
                          MOQTUnsubscribeAnnounces /
                          MOQTSubscribeOk /
                          MOQTSubscribeError /
                          MOQTFetchOk /
                          MOQTFetchError /
                          MOQTSubscribeDone /
                          MOQTMaxSubscribeId /
                          MOQTSubscribesBlocked /
                          MOQTAnnounce /
                          MOQTUnannounce /
                          MOQTTrackStatus /
                          MOQTSubscribeAnnouncesOk /
                          MOQTSubscribeAnnouncesError

$MOQTControlMessage /= MOQTBaseControlMessages
~~~
{: #moqtbasecontrolmessage-def title="MOQTBaseControlMessages definition"}

### MOQTClientSetupMessage

~~~ cddl
MOQTClientSetupMessage = {
  type: "client_setup"
  ? length: uint64
  number_of_support_versions: uint64
  supported_versions: [* uint64]
  number_of_parameters: uint64
  setup_parameters: [* MOQTParameter]
}
~~~
{: #clientsetup-def title="MOQTClientSetupMessage definition"}

### MOQTServerSetupMessage

~~~ cddl
MOQTClientSetupMessage = {
  type: "server_setup"
  ? length: uint64
  selected_version: uint64
  number_of_parameters: uint64
  setup_parameters: [* MOQTParameter]
}
~~~
{: #serversetup-def title="MOQTServerSetupMessage definition"}

### MOQTGoaway

~~~ cddl
MOQTGoaway = {
  type: "goaway"
  ? length: uint64
  ? new_session_uri_length: RawInfo
  ? new_session_uri: RawInfo
}
~~~
{: #goaway-def title="MOQTGoaway definition"}

### MOQTSubscribe

~~~ cddl
MOQTSubscribe = {
  type: "subscribe"
  ? length: uint64
  subscribe_id: uint64
  track_alias: uint64
  track_namespace: TODO pending tuple decision
  track_name: RawInfo
  subscriber_priority: uint8
  group_order: uint8
  filter_type: uint64
  ? start_group: uint64
  ? start_object: uint64
  ? end_group: uint64
  number_of_parameters: uint64
  subscribe_parameters: [* MOQTParameter]
}
~~~
{: #subscribe-def title="MOQTSubscribe definition"}

### MOQTSubscribeUpdate

~~~ cddl
MOQTSubscribeUpdate = {
  type: "subscribe_update"
  ? length: uint64
  subscribe_id: uint64
  start_group: uint64
  start_object: uint64
  end_group: uint64
  subscriber_priority: uint8
  number_of_parameters: uint64
  subscribe_parameters: [* MOQTParameter]
}
~~~
{: #subscribeupdate-def title="MOQTSubscribeUpdate definition"}

### MOQTUnsubscribe

~~~ cddl
MOQTUnsubscribe = {
  type: "unsubscribe"
  ? length: uint64
  subscribe_id: uint64
}
~~~
{: #unsubscribe-def title="MOQTUnsubscribe definition"}

### MOQTFetch

~~~ cddl
MOQTFetch = {
  type: "fetch"
  ? length: uint64
  subscribe_id: uint64
  subscriber_priority: uint8
  group_order: uint8
  fetch_type: uint64

  ? track_namespace: TODO pending tuple decision
  ? track_name: RawInfo
  ? start_group: uint64
  ? start_object: uint64
  ? end_group: uint64
  ? end_object: uint64

  ? joining_subscribe_id: uint64
  ? preceding_group_offset: uint64

  number_of_parameters: uint64
  parameters: [* MOQTParameter]

}
~~~
{: #fetch-def title="MOQTFetch definition"}

### MOQTFetchCancel

~~~ cddl
MOQTFetchCancel = {
  type: "fetch_cancel"
  ? length: uint64
  subscribe_id: uint64
}
~~~
{: #fetchcancel-def title="MOQTFetchCancel definition"}

### MOQTAnnounceOk

~~~ cddl
MOQTAnnounceOk = {
  type: "announce_ok"
  ? length: uint64
  track_namespace: TODO pending tuple decision
}
~~~
{: #announceok-def title="MOQTAnnounceOk definition"}

### MOQTAnnounceError

~~~ cddl
MOQTAnnounceError = {
  type: "announce_error"
  ? length: uint64
  track_namespace: TODO pending tuple decision
  error_code: unit64
  reason_phrase: RawInfo
}
~~~
{: #announceerror-def title="MOQTAnnounceError definition"}

### MOQTAnnounceCancel

~~~ cddl
MOQTAnnounceCancel = {
  type: "announce_cancel"
  ? length: uint64
  track_namespace: TODO pending tuple decision
  error_code: unit64
  reason_phrase: RawInfo
}
~~~
{: #announcecancel-def title="MOQTAnnounceCancel definition"}

### MOQTTrackStatusRequest

~~~ cddl
MOQTTrackStatusRequest = {
  type: "track_status_request"
  ? length: uint64
  track_namespace: TODO pending tuple decision
  track_name: RawInfo
}
~~~
{: #trackstatusrequest-def title="MOQTTrackStatusRequest definition"}

### MOQTSubscribeAnnounces

~~~ cddl
MOQTSubscribeAnnounces = {
  type: "subscribe_announces"
  ? length: uint64
  track_namespace: TODO pending tuple decision
  number_of_parameters: uint64
  subscribe_parameters: [* MOQTParameter]
}
~~~
{: #subscribeannounces-def title="MOQTSubscribeAnnounces definition"}

### MOQTUnsubscribeAnnounces

~~~ cddl
MOQTUnsubscribeAnnounces = {
  type: "subscribe_announces"
  ? length: uint64
  track_namespace: TODO pending tuple decision
}
~~~
{: #unsubscribeannounces-def title="MOQTUnsubscribeAnnounces definition"}

### MOQTSubscribeOk

~~~ cddl
MOQTSubscribeOk = {
  type: "subscribe_ok"
  ? length: uint64
  subscribe_id: uint64
  expires: uint64
  group_order: uint8
  content_exists: uint8
  ? largest_group_id: uint64
  ? largest_object_id: uint64
  number_of_parameters: uint64
  subscribe_parameters: [* MOQTParameter]
}
~~~
{: #subscribeok-def title="MOQTSubscribeOk definition"}

### MOQTSubscribeError

~~~ cddl
MOQTSubscribeError = {
  type: "subscribe_error"
  ? length: uint64
  subscribe_id: uint64
  error_code: unit64
  reason_phrase: RawInfo
  track_alias: uint64
}
~~~
{: #subscribeerror-def title="MOQTSubscribeError definition"}

### MOQTFetchOk

~~~ cddl
MOQTFetchOk = {
  type: "fetch_ok"
  ? length: uint64
  subscribe_id: uint64
  expires: uint64
  end_of_track: uint8
  largest_group_id: uint64
  largest_object_id: uint64
  number_of_parameters: uint64
  subscribe_parameters: [* MOQTParameter]
}
~~~
{: #fetchok-def title="MOQTFetchOk definition"}

### MOQTFetchError

~~~ cddl
MOQTFetchError = {
  type: "fetch_error"
  ? length: uint64
  subscribe_id: uint64
  error_code: unit64
  reason_phrase: RawInfo
}
~~~
{: #fetcherror-def title="MOQTFetchError definition"}

### MOQTSubscribeDone

~~~ cddl
MOQTSubscribeDone = {
  type: "subscribe_done"
  ? length: uint64
  subscribe_id: uint64
  status_code: unit64
  stream_count: unit64
  reason_phrase: RawInfo
}
~~~
{: #subscribedone-def title="MOQTSubscribeDone definition"}

### MOQTMaxSubscribeId

~~~ cddl
MOQTMaxSubscribeId = {
  type: "max_subscribe_id"
  ? length: uint64
  subscribe_id: uint64
}
~~~
{: #maxsubscribeid-def title="MOQTMaxSubscribeId definition"}

### MOQTSubscribesBlocked

~~~ cddl
MOQTSubscribesBlocked = {
  type: "subscribes_blocked"
  ? length: uint64
  maximum_subscribe_id: uint64
}
~~~
{: #subscribesblocked-def title="MOQTSubscribesBlocked definition"}

### MOQTAnnounce

~~~ cddl
MOQTAnnounce = {
  type: "announce"
  ? length: uint64
  track_namespace: TODO pending tuple decision
  number_of_parameters: uint64
  subscribe_parameters: [* MOQTParameter]
}
~~~
{: #announce-def title="MOQTAnnounce definition"}

### MOQTUnannounce

~~~ cddl
MOQTUnannounce = {
  type: "unannounce"
  ? length: uint64
  track_namespace: TODO pending tuple decision
}
~~~
{: #unannounce-def title="MOQTAnnounce definition"}

### MOQTTrackStatus

~~~ cddl
MOQTTrackStatus = {
  type: "track_status"
  ? length: uint64
  track_namespace: TODO pending tuple decision
  track_name: RawInfo
  status_code: uint64
  last_group_id: uint64
  last_object_id: uint64
}
~~~
{: #trackstatus-def title="MOQTTrackStatus definition"}


### MOQTSubscribeAnnouncesOk

~~~ cddl
MOQTSubscribeAnnouncesOk = {
  type: "subscribe_announces_ok"
  ? length: uint64
  track_namespace: TODO pending tuple decision
}
~~~
{: #subscribeannouncesok  -def title="MOQTSubscribeAnnouncesOk definition"}

### MOQTSubscribeAnnouncesError

~~~ cddl
MOQTSubscribeAnnouncesError = {
  type: "subscribe_announces_error"
  ? length: uint64
  track_namespace: TODO pending tuple decision
  error_code: uint64
  reason_phrase: RawInfo
}
~~~
{: #subscribeannounceserror-def title="MOQTSubscribeAnnouncesError definition"}

## MOQTExtensionHeader

~~~ cddl
MOQTExtensionHeader = {
  header_type: uint64
  ? header_value: uint64
  ? header_length: uint64
  ? payload: RawInfo
}
~~~
{: #extensionheader-def title="Extension Header definition"}

# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
