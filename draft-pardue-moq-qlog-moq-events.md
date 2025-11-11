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
 -
    fullname: Mathis Engelbart
    organization: Technical University of Munich
    email: mathis.engelbart@gmail.com
 -
    fullname: Aman Sharma
    organization: Meta
    email: amsharma@meta.com

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
| moqt:subgroup_header_created        | Core       | {{subgroupheadercreated}} |
| moqt:subgroup_header_parsed         | Core       | {{subgroupheaderparsed}} |
| moqt:subgroup_object_created        | Core       | {{subgroupobjectcreated}} |
| moqt:subgroup_object_parsed         | Core       | {{subgroupobjectparsed}} |
| moqt:fetch_header_created           | Core       | {{fetchheadercreated}} |
| moqt:fetch_header_parsed            | Core       | {{fetchheaderparsed}} |
| moqt:fetch_object_created           | Core       | {{fetchobjectcreated}} |
| moqt:fetch_object_parsed            | Core       | {{fetchobjectparsed}} |
{: #events-table title="MOQT Events"}

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
type, data, group_id, RawInfo, and time-related
fields.

Events are defined with an importance level as described in {{Section 8.3 of
QLOG-MAIN}}.

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

Some MOQT messages include a reason phrase that can provide additional
information in the format of a byte sequences. However, these sequences are not
guaranteed to be presentable as UTF-8. In order to accomodate various encodings,
where the wire image of a message includes a reason phrase, the MOQT qlog event
type, includes two option fields: `reason` (for UTF-8) and `reason_bytes` (a
hex-encoded string representing raw bytes). Implementations SHOULD log at least
one format, but MAY log both or none.

## control_message_created {#controlmessagecreated}

The `control_message_created` event is emitted when a control message is created.
It has Core importance level.

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
It has Core importance level.

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
has Base importance level.

~~~ cddl
MOQTStreamTypeSet = {
    ? owner: Owner
    stream_id: uint64
    stream_type: $MOQTStreamType

    * $$moqt-streamtypeset-extension
}

$MOQTStreamType /=  "control" /
                    "subgroup_header" /
                    "fetch_header"
~~~
{: #streamtypeset-def title="MOQTStreamTypeSet definition"}

## object_datagram_created {#objectdatagramcreated}

The `object_datagram_created` event is emitted when the OBJECT_DATAGRAM message
is created. It has Core importance level.

~~~ cddl
MOQTObjectDatagramCreated = {
    track_alias: uint64
    group_id: uint64
    ? object_id: uint64
    publisher_priority: uint8
    ? extension_headers_length: uint64
    ? extension_headers: [* MOQTExtensionHeader]
    ? object_status: uint64
    ? object_payload: RawInfo
    end_of_group: bool

    * $$moqt-objectdatagramcreated-extension
}
~~~
{: #objectdatagramcreated-def title="MOQTObjectDatagramCreated definition"}

## object_datagram_parsed {#objectdatagramparsed}

The `object_datagram_parsed` event is emitted when the OBJECT_DATAGRAM message
is parsed. It has Core importance level.

~~~ cddl
MOQTObjectDatagramParsed = {
    track_alias: uint64
    group_id: uint64
    ? object_id: uint64
    publisher_priority: uint8
    ? extension_headers_length: uint64
    ? extension_headers: [* MOQTExtensionHeader]
    ? object_status: uint64
    ? object_payload: RawInfo
    end_of_group: bool

    * $$moqt-objectdatagramparsed-extension
}
~~~
{: #objectdatagramparsed-def title="MOQTObjectDatagramParsed definition"}

## subgroup_header_created {#subgroupheadercreated}

The `subgroup_header_created` event is emitted when a stream begins and a
SUBGROUP_HEADER is created. It has Core importance level; see {{Section 9.2 of
QLOG-MAIN}}.

~~~ cddl
MOQTSubgroupHeaderCreated = {
    stream_id: uint64
    track_alias: uint64
    group_id: uint64
    ? subgroup_id: uint64
    publisher_priority: uint8

    * $$moqt-subgroupheadercreated-extension
}
~~~
{: #subgroupheadercreated-def title="MOQTSubgroupHeaderCreated definition"}

## subgroup_header_parsed {#subgroupheaderparsed}

The `subgroup_header_parsed` event is emitted when the SUBGROUP_HEADER is
parsed. It has Core importance level.

~~~ cddl
MOQTSubgroupHeaderParsed = {
    stream_id: uint64
    track_alias: uint64
    group_id: uint64
    ? subgroup_id: uint64
    publisher_priority: uint8

    * $$moqt-subgroupheaderparsed-extension
}
~~~
{: #subgroupheaderparsed-def title="MOQTSubgroupHeaderParsed definition"}

## subgroup_object_created {#subgroupobjectcreated}

The `subgroup_object_created` event is emitted when a subgroup object is
created. It has Core importance level.

~~~ cddl
MOQTSubgroupObjectCreated = {
    stream_id: uint64
    ? group_id: uint64
    ? subgroup_id: uint64
    object_id: uint64
    extension_headers_length: uint64
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
It has Core importance level.

~~~ cddl
MOQTSubgroupObjectParsed = {
    stream_id: uint64
    ? group_id: uint64
    ? subgroup_id: uint64
    object_id: uint64
    extension_headers_length: uint64
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
FETCH_HEADER is created. It has Core importance level.

~~~ cddl
MOQTFetchHeaderCreated = {
    stream_id: uint64
    request_id: uint64

    * $$moqt-fetchheadercreated-extension
}
~~~
{: #fetchheadercreated-def title="MOQTFetchHeaderCreated definition"}

## fetch_header_parsed {#fetchheaderparsed}

The `fetch_header_parsed` event is emitted when the SUBGROUP_HEADER is
parsed. It has Core importance level.

~~~ cddl
MOQTFetchHeaderParsed = {
    stream_id: uint64
    request_id: uint64

    * $$moqt-fetchheaderparsed-extension
}
~~~
{: #fetchheaderparsedd-def title="MOQTFetchHeaderParsed   definition"}

## fetch_object_created {#fetchobjectcreated}

The `fetch_object_created` event is emitted when a fetch object is created. It
has Core importance level.

~~~ cddl
MOQTFetchObjectCreated = {
    stream_id: uint64
    group_id: uint64
    subgroup_id: uint64
    object_id: uint64
    publisher_priority: uint8
    extension_headers_length: uint64
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
Core importance level.

~~~ cddl
MOQTFetchObjectParsed = {
    stream_id: uint64
    group_id: uint64
    subgroup_id: uint64
    object_id: uint64
    publisher_priority: uint8
    extension_headers_length: uint64
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

## MOQTSetupParameter

The generic $MOQTSetupParameter is defined here as a CDDL "type socket"
extension point. It can be extended to support additional MOQT Setup Parameters.

~~~ cddl
; The MOQTSetupParameter is any key-value map (e.g., JSON object)
$MOQTSetupParameter /= {
    * text => any
}
~~~
{: #moqtsetupparameter-def title="MOQTSetupParameter type socket definition"}

~~~ cddl
MOQTBaseSetupParameters /=  MOQTAuthorityParameter /
                            MOQTPathSetupParameter /
                            MOQTMaxRequestIdSetupParameter /
                            MOQTMaxAuthTokenCacheSizeParameter /
                            MOQTAuthorizationTokenParameter /
                            MOQTImplementationParameter /
                            MOQTUnknownSetupParameter

$MOQTSetupParameter /= MOQTBaseSetupParameters
~~~
{: #moqtbasesetupparameters-def title="MOQTBaseSetupParameters definition"}

### MOQTAuthorityParameter

~~~ cddl
MOQTAuthorityParameter = {
  name: "authority"
  value: text
}
~~~
{: #moqtauthorityparameter-def title="MOQTAuthorityParameter definition"}

### MOQTPathSetupParameter

~~~ cddl
MOQTPathSetupParameter = {
  name: "path"
  value: text
}
~~~
{: #moqtpathsetupparameter-def title="MOQTPathSetupParameter definition"}

### MOQTMaxRequestIdSetupParameter

~~~ cddl
MOQTMaxRequestIdSetupParameter = {
  name: "max_request_id"
  value: uint64
}
~~~
{: #moqtmaxsubscribeidsetupparameter-def title="MOQTMaxRequestIdSetupParameter definition"}

### MOQTMaxAuthTokenCacheSizeParameter

~~~ cddl
MOQTMaxAuthTokenCacheSizeParameter = {
  name: "max_auth_token_cache_size"
  value: uint64
}
~~~
{: #moqtmaxauthtokencachesizeparameter-def title="MOQTMaxAuthTokenCacheSizeParameter definition"}

### MOQTAuthorizationTokenParameter

~~~ cddl
MOQTAuthorizationTokenParameter = {
  name: "authorization_token"
  value: [* AuthorizationToken]
}

$MOQTAuthorizationToken = {
  alias_type: MOQTAliasType
  ? token_alias: uint64
  ? token_type: uint64
  ? token_value: RawInfo
}

$MOQTAliasType /=  "delete" /
                   "register" /
                   "use_alias" /
                   "use_value"
~~~
{: #moqtauthorizationtokenparameter-def title="MOQTAuthorizationTokenParameter definition"}

### MOQTImplementationParameter

~~~ cddl
MOQTImplementationParameter = {
  name: "implementation"
  value: text
}
~~~
{: #moqtimplementationparameter-def title="MOQTImplementationParameter definition"}

### MOQTUnknownSetupParameter

~~~ cddl
MOQTUnknownSetupParameter = {
  name:"unknown"
  name_bytes: uint64
  ? length: uint64
  ? value: uint64
  ? value_bytes: RawInfo
}
~~~
{: #moqtunknownsetupparameter-def title="MOQTUnknownSetupParameter definition"}

## MOQTParameter

The generic $MOQTParameter is defined here as a CDDL "type socket" extension
point. It can be extended to support additional MOQT Parameters.

~~~ cddl
; The MOQTParameter is any key-value map (e.g., JSON object)
$MOQTParameter /= {
    * text => any
}
~~~
{: #moqtparameter-def title="MOQTParameter type socket definition"}

~~~ cddl
MOQTBaseParameters /= MOQTAuthorizationTokenParameter /
                      MOQTDeliveryTimeoutParameter /
                      MOQTMaxCacheDurationParameter /
                      MOQTUnknownParameter

$MOQTParameter /= MOQTBaseParameters
~~~
{: #moqtbaseparameters-def title="MOQTBaseParameters definition"}

### MOQTAuthorizationTokenParameter

~~~ cddl
MOQTAuthorizationTokenParameter = {
  name: "authorization_token"
  alias_type: uint64
  ? token_alias: uint64
  ? token_type: uint64
  ? token_value: RawInfo
}
~~~
{: #moqtauthorizationTokenparameter-def title="MOQTAuthorizationTokenParameter definition"}

### MOQTDeliveryTimeoutParameter

~~~ cddl
MOQTDeliveryTimeoutParameter = {
  name: "delivery_timeout"
  value: uint64
}
~~~
{: #moqtdeliverytimeoutparameter-def title="MOQTDeliveryTimeoutParameter definition"}

### MOQTMaxCacheDurationParameter

~~~ cddl
MOQTMaxCacheDurationParameter = {
  name: "max_cache_duration"
  value: uint64
}
~~~
{: #moqtmaxcachedurationparameter-def title="MOQTMaxCacheDurationParameter definition"}

### MOQTUnknownParameter

~~~ cddl
MOQTUnknownParameter = {
  name:"unknown"
  name_bytes: uint64
  ? length: uint64
  ? value: uint64
  ? value_bytes: RawInfo
}
~~~
{: #moqtunknownparameter-def title="MOQTUnknownParameter definition"}

## MOQTByteString

The MOQTByteString type allows representing MOQT bytestrings, such as the value
of a Track or Track Namespace tuple field, using two different encodings. The
`value` field can be used for bytestrings that can be encoded in UTF-8. The
`value_bytes` field can be used for bytestrings of any type by using the
`hexstring` encoding.

Implementations SHOULD populate one of either the `value` or `value_bytes`
field. Populating both fields is redundant.

~~~ cddl
MOQTByteString = {
  ? value: text
  ? value_bytes: hexstring
}
~~~
{: #MOQTByteString-def title="MOQTByteString definition"}

## MOQTLocation

A Location, as defined in {{Section 1.3.1 of MOQT}}

~~~ cddl
MOQTLocation = {
  group: uint64
  object: uint64
}
~~~
{: #moqtlocation-def title="MOQTLocation definition}


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
                          MOQTMaxRequestId /
                          MOQTRequestsBlocked /
                          MOQTSubscribe /
                          MOQTSubscribeOk /
                          MOQTSubscribeError /
                          MOQTSubscribeUpdate /
                          MOQTUnsubscribe /
                          MOQTPublishDone /
                          MOQTPublish /
                          MOQTPublishOk /
                          MOQTPublishError /
                          MOQTFetch /
                          MOQTFetchOk /
                          MOQTFetchError /
                          MOQTFetchCancel /
                          MOQTTrackStatus /
                          MOQTTrackStatusOk /
                          MOQTTrackStatusError /
                          MOQTPublishNamespace /
                          MOQTPublishNamespaceOk /
                          MOQTPublishNamespaceError /
                          MOQTPublishNamespaceDone /
                          MOQTPublishNamespaceCancel /
                          MOQTSubscribeNamespace /
                          MOQTSubscribeNamespaceOk /
                          MOQTSubscribeNamespaceError /
                          MOQTUnsubscribeNamespace

$MOQTControlMessage /= MOQTBaseControlMessages
~~~
{: #moqtbasecontrolmessage-def title="MOQTBaseControlMessages definition"}


### MOQTClientSetupMessage

~~~ cddl
MOQTClientSetupMessage = {
  type: "client_setup"
  number_of_supported_versions: uint64
  supported_versions: [* uint64]
  number_of_parameters: uint64
  ? setup_parameters: [* $MOQTSetupParameter]
}
~~~
{: #clientsetup-def title="MOQTClientSetupMessage definition"}

### MOQTServerSetupMessage

~~~ cddl
MOQTServerSetupMessage = {
  type: "server_setup"
  selected_version: uint64
  number_of_parameters: uint64
  ? setup_parameters: [* $MOQTSetupParameter]
}
~~~
{: #serversetup-def title="MOQTServerSetupMessage definition"}

### MOQTGoaway

~~~ cddl
MOQTGoaway = {
  type: "goaway"
  ? length: uint64
  new_session_uri: RawInfo
}
~~~
{: #goaway-def title="MOQTGoaway definition"}

### MOQTMaxRequestId

~~~ cddl
MOQTMaxRequestId = {
  type: "max_request_id"
  request_id: uint64
}
~~~
{: #maxsubscribeid-def title="MOQTMaxRequestId definition"}

### MOQTRequestsBlocked

~~~ cddl
MOQTRequestsBlocked = {
  type: "requests_blocked"
  maximum_request_id: uint64
}
~~~
{: #subscribesblocked-def title="MOQTRequestsBlocked definition"}

### MOQTSubscribe

~~~ cddl
MOQTSubscribe = {
  type: "subscribe"
  request_id: uint64
  track_namespace: [ *MOQTByteString]
  track_name: MOQTByteString
  subscriber_priority: uint8
  group_order: uint8
  forward: uint8
  filter_type: uint64
  ? start_location: MOQTLocation
  ? end_group: uint64
  number_of_parameters: uint64
  ? parameters: [* $MOQTParameter]
}
~~~
{: #subscribe-def title="MOQTSubscribe definition"}

### MOQTSubscribeOk

~~~ cddl
MOQTSubscribeOk = {
  type: "subscribe_ok"
  request_id: uint64
  track_alias: uint64
  expires: uint64
  group_order: uint8
  content_exists: uint8
  ? largest_location: MOQTLocation
  number_of_parameters: uint64
  ? parameters: [* $MOQTParameter]
}
~~~
{: #subscribeok-def title="MOQTSubscribeOk definition"}

### MOQTSubscribeError

~~~ cddl
MOQTSubscribeError = {
  type: "subscribe_error"
  request_id: uint64
  error_code: uint64
  ? reason: text
  ? reason_bytes: hexstring
}
~~~
{: #subscribeerror-def title="MOQTSubscribeError definition"}

### MOQTSubscribeUpdate

~~~ cddl
MOQTSubscribeUpdate = {
  type: "subscribe_update"
  request_id: uint64
  start_location: MOQTLocation
  end_group: uint64
  subscriber_priority: uint8
  forward: uint8
  number_of_parameters: uint64
  ? parameters: [* $MOQTParameter]
}
~~~
{: #subscribeupdate-def title="MOQTSubscribeUpdate definition"}

### MOQTUnsubscribe

~~~ cddl
MOQTUnsubscribe = {
  type: "unsubscribe"
  request_id: uint64
}
~~~
{: #unsubscribe-def title="MOQTUnsubscribe definition"}

### MOQTPublishDone

~~~ cddl
MOQTPublishDone = {
  type: "publish_done"
  request_id: uint64
  status_code: uint64
  stream_count: uint64
  ? reason: text
  ? reason_bytes: hexstring
}
~~~
{: #publishdone-def title="MOQTPublishDone definition"}

### MOQTPublish

~~~ cddl
MOQTPublish = {
  type: "publish"
  request_id: uint64
  track_namespace: [ *MOQTByteString]
  track_name: MOQTByteString
  track_alias: uint64
  group_order: uint8
  content_exists: uint8
  ? largest: MOQTLocation
  forward: uint8
  number_of_parameters: uint64
  ? parameters: [* $MOQTParameter]
}
~~~
{: #publish-def title="MOQTPublish definition"}

### MOQTPublishOk

~~~ cddl
MOQTPublishOk = {
  type: "publish_ok"
  request_id: uint64
  forward: uint8
  subscriber_priority: uint8
  group_order: uint8
  filter_type: uint64
  ? start: MOQTLocation
  ? end_group: uint64
  number_of_parameters: uint64
  ? parameters: [* $MOQTParameter]
}
~~~
{: #publishok-def title="MOQTPublishOk definition"}

### MOQTPublishError

~~~ cddl
MOQTPublishError = {
  type: "publish_error"
  request_id: uint64
  error_code: uint64
  ? reason: text
  ? reason_bytes: hexstring
}
~~~
{: #publisherror-def title="MOQTPublishError definition"}

### MOQTFetch

~~~ cddl
MOQTFetch = {
  type: "fetch"
  request_id: uint64
  subscriber_priority: uint8
  group_order: uint8
  fetch_type: uint64

  track_namespace: [ *MOQTByteString]
  ? track_name: MOQTByteString
  ? start_location: MOQTLocation
  ? end_location: MOQTLocation

  ? joining_request_id: uint64
  ? preceding_group_offset: uint64

  number_of_parameters: uint64
  ? parameters: [* $MOQTParameter]

}
~~~
{: #fetch-def title="MOQTFetch definition"}

### MOQTFetchOk

~~~ cddl
MOQTFetchOk = {
  type: "fetch_ok"
  request_id: uint64
  group_order: uint8
  end_of_track: uint8
  end_location: MOQTLocation
  number_of_parameters: uint64
  ? parameters: [* $MOQTParameter]
}
~~~
{: #fetchok-def title="MOQTFetchOk definition"}

### MOQTFetchError

~~~ cddl
MOQTFetchError = {
  type: "fetch_error"
  request_id: uint64
  error_code: uint64
  ? reason: text
  ? reason_bytes: hexstring
}
~~~
{: #fetcherror-def title="MOQTFetchError definition"}

### MOQTFetchCancel

~~~ cddl
MOQTFetchCancel = {
  type: "fetch_cancel"
  request_id: uint64
}
~~~
{: #fetchcancel-def title="MOQTFetchCancel definition"}

### MOQTTrackStatus

~~~ cddl
MOQTTrackStatus = {
  type: "track_status"
  request_id: uint64
  track_namespace: [ *MOQTByteString]
  track_name: MOQTByteString
  subscriber_priority: uint8
  group_order: uint8
  forward: uint8
  filter_type: uint64
  ? start_location: MOQTLocation
  ? end_group: uint64
  number_of_parameters: uint64
  ? parameters: [* $MOQTParameter]
}
~~~
{: #trackstatus-def title="MOQTTrackStatus definition"}

### MOQTTrackStatusOk

~~~ cddl
MOQTTrackStatusOk = {
  type: "track_status_ok"
  request_id: uint64
  track_alias: uint64
  expires: uint64
  group_order: uint8
  content_exists: uint8
  ? largest_location: MOQTLocation
  number_of_parameters: uint64
  ? parameters: [* $MOQTParameter]
}
~~~
{: #trackstatusok-def title="MOQTTrackStatusOk definition"}

### MOQTTrackStatusError

~~~ cddl
MOQTTrackStatusError = {
  type: "track_status_error"
  request_id: uint64
  error_code: uint64
  ? reason: text
  ? reason_bytes: hexstring
}
~~~
{: #trackstatuserror-def title="MOQTTrackStatusError definition"}

### MOQTPublishNamespace

~~~ cddl
MOQTPublishNamespace = {
  type: "publish_namespace"
  request_id: uint64
  track_namespace: [ *MOQTByteString]
  number_of_parameters: uint64
  ? parameters: [* $MOQTParameter]
}
~~~
{: #publishnamespace-def title="MOQTPublishNamespace definition"}

### MOQTPublishNamespaceOk

~~~ cddl
MOQTPublishNamespaceOk = {
  type: "publish_namespace_ok"
  request_id: uint64
}
~~~
{: #publishnamespaceok-def title="MOQTPublishNamespaceOk definition"}

### MOQTPublishNamespaceError

~~~ cddl
MOQTPublishNamespaceError = {
  type: "publish_namespace_error"
  request_id: uint64
  error_code: uint64
  ? reason: text
  ? reason_bytes: hexstring
}
~~~
{: #publishnamespaceerror-def title="MOQTPublishNamespaceError definition"}

### MOQTPublishNamespaceDone

~~~ cddl
MOQTPublishNamespaceDone = {
  type: "publish_namespace_done"
  track_namespace: [ *MOQTByteString]
}
~~~
{: #publishnamespacedone-def title="MOQTPublishNamespaceDone definition"}

### MOQTPublishNamespaceCancel

~~~ cddl
MOQTPublishNamespaceCancel = {
  type: "publish_namespace_cancel"
  track_namespace: [ *MOQTByteString]
  error_code: uint64
  ? reason: text
  ? reason_bytes: hexstring
}
~~~
{: #publishnamespacecancel-def title="MOQTPublishNamespaceCancel definition"}

### MOQTSubscribeNamespace

~~~ cddl
MOQTSubscribeNamespace = {
  type: "subscribe_namespace"
  request_id: uint64
  track_namespace_prefix: [ *MOQTByteString]
  number_of_parameters: uint64
  ? parameters: [* $MOQTParameter]
}
~~~
{: #subscribenamespace-def title="MOQTSubscribeNamespace definition"}

### MOQTSubscribeNamespaceOk

~~~ cddl
MOQTSubscribeNamespaceOk = {
  type: "subscribe_namespace_ok"
  request_id: uint64
}
~~~
{: #subscribenamespaceok  -def title="MOQTSubscribeNamespaceOk definition"}

### MOQTSubscribeNamespaceError

~~~ cddl
MOQTSubscribeNamespaceError = {
  type: "subscribe_namespace_error"
  request_id: uint64
  error_code: uint64
  ? reason: text
  ? reason_bytes: hexstring
}
~~~
{: #subscribenamespaceerror-def title="MOQTSubscribeNamespaceError definition"}

### MOQTUnsubscribeNamespace

~~~ cddl
MOQTUnsubscribeNamespace = {
  type: "unsubscribe_namespace"
  track_namespace_prefix: [ *MOQTByteString]
}
~~~
{: #unsubscribenamespace-def title="MOQTUnsubscribeNamespace definition"}

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

The security and privacy considerations discussed in {{QLOG-MAIN}} apply to this
document as well.

# IANA Considerations

This document registers a new entry in the "qlog event schema URIs" registry (created in {{Section 15 of QLOG-MAIN}}).

Event schema URI:
: urn:ietf:params:qlog:events:moqt

Namespace
: moqt

Event Types
: control_message_created,
  control_message_parsed,
  stream_type_set,
  object_datagram_created,
  object_datagram_parsed,
  subgroup_header_created,
  subgroup_header_parsed,
  subgroup_object_created,
  subgroup_object_parsed,
  fetch_header_created,
  fetch_header_parsed,
  fetch_object_created,
  fetch_object_parsed

Description:
: Event definitions related to the MOQT protocol.

Reference:
: This Document

--- back

# Acknowledgments
{:numbered="false"}

Thanks to Lorenzo Miniero, Sujay Patel, and Aman Sharm for feedback and contributions to this document.
