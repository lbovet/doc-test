# AT-Gateway - API Specifications

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [AT-Gateway - API Specifications](#at-gateway---api-specifications)
- [1. Overview](#1-overview)
- [2. Public Access](#2-public-access)
  - [2.1. Github](#21-github)
    - [2.1.1. Compare](#211-compare)
  - [2.2. nuget.org](#22-nugetorg)
- [3. ATG-API Standard PubSub Specification](#3-atg-api-standard-pubsub-specification)
  - [3.1. Introduction](#31-introduction)
  - [3.2. References](#32-references)
  - [3.3. Principles](#33-principles)
  - [3.4. Protocol](#34-protocol)
  - [3.5. Topics](#35-topics)
    - [3.5.1. Pattern](#351-pattern)
    - [3.5.2. Semantics](#352-semantics)
    - [3.5.3. Resources, Operations and Events](#353-resources-operations-and-events)
    - [3.5.4. Status Message](#354-status-message)
  - [3.6. Interaction Sequence Patterns](#36-interaction-sequence-patterns)
    - [3.6.1. get-request (Pull)](#361-get-request-pull)
    - [3.6.2. set-request](#362-set-request)
    - [3.6.3. update (Push)](#363-update-push)
  - [3.7. Client Behaviour](#37-client-behaviour)
    - [3.7.1. Connection](#371-connection)
    - [3.7.2. Last Will](#372-last-will)
    - [3.7.3. Topic and Broker Configuration](#373-topic-and-broker-configuration)
    - [3.7.4. Retained Messages](#374-retained-messages)
    - [3.7.5. Empty Messages](#375-empty-messages)
  - [3.8. Message Format](#38-message-format)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# 1. Overview

This documentation describes all APIs the AT Gateway supports. These
pages also contain the links to the API definition resources and
artifacts.

For a general overview of the concepts that apply to all APIs, see the
chapter [ATG-API Standard PubSub
Specification](https://wikit.post.ch/display/TEC/ATG-API+Standard+PubSub+Specification).

# 2. Public Access

## 2.1. Github

The source code of the APIs as well as the generated artifacts and
documentation is publicly exposed in Github. For links, see the
corresponding sub page.

-   Releases are tagged with a **release/{version}** tag,
    e.g. <https://github.com/swisspush/cen-mc-rc/releases/tag/release%2F1.0.0-rc.9>
-   Additionally, there is a **dist/{version}** tag that also contains
    the generated artifacts (at model and doc),
    e.g. <https://github.com/swisspush/cen-mc-rc/releases/tag/dist%2F1.0.0-rc.9>

### 2.1.1. Compare

We can use Github functionality to compare to release versions, e.g.
differences between version **1.0.0-rc.9** and **1.0.0-rc.10** of
**cen-mc-rc**: <https://github.com/swisspush/cen-mc-rc/compare/dist/1.0.0-rc.9...dist/1.0.0-rc.10>

## 2.2. [nuget.org](http://nuget.org)

The NuGet packages of the APIs containing the generated C\# classes that
map from and to the APIs JSON structures are published to the
[https://www.nuget.org](https://www.nuget.org/) repository.
Corresponding links are available on the sub pages.

# 3. ATG-API Standard PubSub Specification

 

-   [Introduction](#AT-Gateway-APISpecifications-Introduction)
-   [References](#AT-Gateway-APISpecifications-References)
-   [Principles](#AT-Gateway-APISpecifications-Principles)
-   [Protocol](#AT-Gateway-APISpecifications-Protocol)
-   [Topics](#AT-Gateway-APISpecifications-Topics)
    -   [Pattern](#AT-Gateway-APISpecifications-Pattern)
    -   [Semantics](#AT-Gateway-APISpecifications-Semantics)
    -   [Resources, Operations and
        Events](#AT-Gateway-APISpecifications-Resources,OperationsandEvents)
    -   [Status Message](#AT-Gateway-APISpecifications-StatusMessage)
-   [Interaction Sequence
    Patterns](#AT-Gateway-APISpecifications-InteractionSequencePatterns)
    -   [get-request
        (Pull)](#AT-Gateway-APISpecifications-get-request(Pull))
    -   [set-request](#AT-Gateway-APISpecifications-set-request)
    -   [update (Push)](#AT-Gateway-APISpecifications-update(Push))
-   [Client Behaviour](#AT-Gateway-APISpecifications-ClientBehaviour)
    -   [Connection](#AT-Gateway-APISpecifications-Connection)
    -   [Last Will](#AT-Gateway-APISpecifications-LastWill)
    -   [Topic and Broker
        Configuration](#AT-Gateway-APISpecifications-TopicandBrokerConfiguration)
    -   [Retained
        Messages](#AT-Gateway-APISpecifications-RetainedMessages)
    -   [Empty Messages](#AT-Gateway-APISpecifications-EmptyMessages)
-   [Message Format](#AT-Gateway-APISpecifications-MessageFormat)

## 3.1. Introduction

This chapter exposes the standard way for integrating systems together
via the AT-Gateway. This applies to all PubSub interfaces (a.k.a
asynchronous APIs) exposed by the AT-Gateway and covers the common
aspects. API-specific aspects are covered in dedicated API
specifications.

## 3.2. References

1.  <https://en.wikipedia.org/wiki/Publish%E2%80%93subscribe_pattern>
2.  <https://en.wikipedia.org/wiki/MQTT>
3.  <https://semver.org/>
4.  <https://www.json.org/>
5.  <http://json-schema.org/>

## 3.3. Principles

1.  The communication uses a topic-based
    [publish-subscribe](https://en.wikipedia.org/wiki/Publish%E2%80%93subscribe_pattern) pattern
    \[1\]. 
2.  Systems are spatially decoupled, they must only know the AT-Gateway.
3.  Systems are timely coupled, there is no delayed delivery in case of
    system down, except for the last message of a topic if specified so
    (retained messages).
4.  There is no message queue persistence. The AT-Gateway basically
    plays the role of an in-memory event bus.
5.  Formal API definitions are provided in swagger format for
    documentation but they are *not *REST APIs, please
    mentally substitute POST with PUBLISH.

## 3.4. Protocol

1.  The publish-subscribe protocol
    is [MQTT](https://en.wikipedia.org/wiki/MQTT) 3.1.1 \[2\].
2.  Quality of service is 1, at least once delivery (acknowledged
    delivery).
3.  No persistent session. `cleanSession` always set to true.

## 3.5. Topics

-   Related topics are grouped in versioned asynchronous APIs.
-   Topic names are kebab-case (all lowercase and words are separated
    with dashes). Except system names that are uppercase.

### 3.5.1. Pattern

Topics follow the
pattern `API_NAME/VERSION/ROLE/SUBJECT/SRC_SYSTEM/SRC_INSTANCE/[DEST_SYSTEM/DEST_INSTANCE/][CONVERSATION_ID].`

Example: `cen-mc-rc/v1/machine/submit-mailpiece/TRC/21/`

-   `API_NAME` correspond to the title of the API specification.
-   `VERSION` is the major version number of the API, following
    [semver](https://semver.org/)\[3\] compatibility rules.
-   `ROLE` is a logical identification if the system publishing on this
    topic.
-   `SUBJECT` corresponds to the message type posted in this topic.
-   `SRC_SYSTEM` is the standardized uppercase short name given to the
    posting system.
-   `SRC_INSTANCE` is the three-digit number of the system instance.
-   `CONVERSATION_ID`: When issuing a request that will trigger a
    response, a conversationId must be submitted in order to be able to
    correlate the response to the request. The conversationId must also
    be submitted by the responding system (also see
    **[00.StandardPubSubSpecification-Resources,OperationsandEvents](#AT-Gateway-APISpecifications-00.StandardPubSubSpecification-Resources,OperationsandEvents))**.
    Example:
    -   Request: my\_api/v1/role/subject:get/SYS\_A/**01**/12345
    -   Response:
        my\_api/v1/role/subject:get:status/SYS\_B/02/**SYS\_A**/**01**/**12345**

### 3.5.2. Semantics

-   Each API specification specifies the mandatory segments of each
    topic.
-   Clients basically subscribe to `API_NAME/VERSION/ROLE/#` 
    -   ![(info)](images/icons/emoticons/information.svg "(info)"){.emoticon
        .emoticon-information} For performance reasons, subscriptions
        may be split up into more specific sub-subscriptions. But
        conceptually, the client should be able to deal with any message
        topics sent to him. If it is not supposed to handle a certain
        topic, it should just ignore this message. For details,
        see [00.StandardPubSubSpecification-TopicFiltering](#AT-Gateway-APISpecifications-00.StandardPubSubSpecification-TopicFiltering)
-   For point-to-point or multicast messaging, clients filter the
    message with the topic
    pattern API\_NAME/VERSION/ROLE/+/+/DEST\_SYSTEM/DEST\_INSTANCE to
    handle only messages addressed to them. In API definitions,
    such behavior is specified in the formal topic pattern.
-   When responding to a point-to-point request,` DEST_SYSTEM`,
    `DEST_INSTANCE` must be filled with the `SRC_SYSTEM` and
    `SRC_INSTANCE` used in the request. In API definitions,
    such behavior is specified in the formal topic pattern.
-   When responding to a point-to-point request containing
    a `CONVERSATION_ID`, this latter must be used for the response
    topic. In API definitions, such behavior is specified in the formal
    topic pattern.
-   When a conversation id is required but the destination is not known,
    `DEST_SYSTEM` and `DEST_INSTANCE`  are replaced with question marks
    (e.g. cen-mc-rc/v1/control/query-parameters/MDS/21/?/?/444883).

### 3.5.3. Resources, Operations and Events

The subject can denote resources, operations or events. When these are
related, they appear in the subject name.

`RESOURCE:OPERATION`, example: `parameter-list:get`

-   A topic with such a subject will be use to request the current
    parameter list resource. As a result, the resource will be provided
    in the corresponding topic with subject `parameter-list`.
-   The set of operations (get, set, ...) is not normatively defined.

`RESOURCE:EVENT`, example: `parameter-list:status`

-   Such a topic informs about something that happened on the resource
    (e.g. that it has been deleted).

The set of events is not normatively defined but the mandatory fields of
the status event are (see below).   
  
`RESOURCE:OPERATION:EVENT`, example: `parameter-list:get:status`

-   Such a topic informs about something that happened on an <span
    class="underline">operation</span> related to the resource. In such
    an interaction, the conversation id is mandatory.

The set of events is not normatively defined but the mandatory fields of
the status event are (see below). 

### 3.5.4. Status Message

The content of these events are normed. They contain a `state` field
having values taken from a subset of:

<table>
<thead>
<tr class="header">
<th>Value</th>
<th>Resource</th>
<th>Operation</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>OK</code></td>
<td>The resource is in a consistent state</td>
<td>The operation was performed successfully</td>
</tr>
<tr class="even">
<td><code>ERROR</code></td>
<td>The resource is in an error state</td>
<td><p>The operation failed.</p>
<p>An optional field <em>failedReason</em> (string) may contain additional information on why the operation failed</p></td>
</tr>
<tr class="odd">
<td><code>PROCESSING</code></td>
<td>The resource is being under change and possibly inconsistent</td>
<td>The operation is being processed and not yet finished</td>
</tr>
</tbody>
</table>

## 3.6. Interaction Sequence Patterns

Different use cases require different interaction sequences between the
publishes and the subscriber(s) of a certain topics. The used sequence
patterns are as follows:

### 3.6.1. get-request (Pull)

A get-request is issued to trigger an operation on a resource. The
request must include a conversationId. The processing system will
include that ID in its *get:status* message (either successful or
unsuccessful), along with the destination system (which equals the
source system of the get request). In case an operation should return
data, an additional message must be published on the resource topic
without the conversationId (and without the destination system).

  
![](download/temp/plantuml4318010015012751885.png)

### 3.6.2. set-request

A set-request is simular to a get-request, but it contains a data
payload describing on how we want to change a resource. Requests also
include a conversationId which will be included in the status response.
In the successful case, the processing system always publishes a message
containing the updated data.

  
![](download/temp/plantuml5526309624376031072.png)

### 3.6.3. update (Push)

An update request is published by systems to communicate udpates on a
resource.

![(info)](images/icons/emoticons/information.svg "(info)"){.emoticon
.emoticon-information} For a subscribing system, it does not make any
difference if a resource update is published as a push message or as a
result of a get- or set-request.

  
![](download/temp/plantuml6658697714480586126.png)

## 3.7. Client Behaviour

### 3.7.1. Connection

Clients connect using a clientId composed of the system name and
instance (e.g. ASL01).

Upon successful connection, clients must update their state by sending a
client status message:

<table>
<tbody>
<tr class="odd">
<td>Topic</td>
<td><code>vsi-process-control/v1/client/status/{SRC_SYSTEM}/{SRC_INSTANCE}</code></td>
</tr>
<tr class="even">
<td>Retain</td>
<td>true</td>
</tr>
<tr class="odd">
<td>QoS</td>
<td>1</td>
</tr>
<tr class="even">
<td>Payload</td>
<td><div class="content-wrapper">
<div class="code panel pdl" style="border-width: 1px;">
<div class="codeContent panelContent pdl">
<div class="sourceCode" id="cb1" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence" style="brush: java; gutter: false; theme: Confluence"><pre class="sourceCode java"><code class="sourceCode java"><a class="sourceLine" id="cb1-1" data-line-number="1">{</a>
<a class="sourceLine" id="cb1-2" data-line-number="2">   <span class="st">&quot;state&quot;</span>: <span class="st">&quot;CONNECTED&quot;</span>,</a>
<a class="sourceLine" id="cb1-3" data-line-number="3">   <span class="st">&quot;timestamp&quot;</span>: <span class="st">&quot;2018-08-31T08:10:07+00:00&quot;</span></a>
<a class="sourceLine" id="cb1-4" data-line-number="4">}</a></code></pre></div>
</div>
</div>
</div></td>
</tr>
</tbody>
</table>

Before cleanly disconnecting, it must send:

<table>
<tbody>
<tr class="odd">
<td>Topic</td>
<td><code>vsi-process-control/v1/client/status/{SRC_SYSTEM}/{SRC_INSTANCE}</code></td>
</tr>
<tr class="even">
<td>Retain</td>
<td>true</td>
</tr>
<tr class="odd">
<td>QoS</td>
<td>1</td>
</tr>
<tr class="even">
<td>Payload</td>
<td><div class="content-wrapper">
<div class="code panel pdl" style="border-width: 1px;">
<div class="codeContent panelContent pdl">
<div class="sourceCode" id="cb1" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence" style="brush: java; gutter: false; theme: Confluence"><pre class="sourceCode java"><code class="sourceCode java"><a class="sourceLine" id="cb1-1" data-line-number="1">{</a>
<a class="sourceLine" id="cb1-2" data-line-number="2">   <span class="st">&quot;state&quot;</span>: <span class="st">&quot;DISCONNECTED&quot;</span>,</a>
<a class="sourceLine" id="cb1-3" data-line-number="3">   <span class="st">&quot;timestamp&quot;</span>: <span class="st">&quot;2018-08-31T08:10:07+00:00&quot;</span></a>
<a class="sourceLine" id="cb1-4" data-line-number="4">}</a></code></pre></div>
</div>
</div>
</div></td>
</tr>
</tbody>
</table>

### 3.7.2. Last Will

In order to monitor the liveliness of clients, it is required that
clients specify a MQTT Last Will message:

<table>
<tbody>
<tr class="odd">
<td>Topic</td>
<td><code>vsi-process-control/v1/client/status/{SRC_SYSTEM}/{SRC_INSTANCE}</code></td>
</tr>
<tr class="even">
<td>Retain</td>
<td>true</td>
</tr>
<tr class="odd">
<td>QoS</td>
<td>1</td>
</tr>
<tr class="even">
<td>Payload</td>
<td><div class="content-wrapper">
<div class="code panel pdl" style="border-width: 1px;">
<div class="codeContent panelContent pdl">
<div class="sourceCode" id="cb1" data-syntaxhighlighter-params="brush: java; gutter: false; theme: Confluence" data-theme="Confluence" style="brush: java; gutter: false; theme: Confluence"><pre class="sourceCode java"><code class="sourceCode java"><a class="sourceLine" id="cb1-1" data-line-number="1">{</a>
<a class="sourceLine" id="cb1-2" data-line-number="2">   <span class="st">&quot;state&quot;</span>: <span class="st">&quot;BROKEN&quot;</span></a>
<a class="sourceLine" id="cb1-3" data-line-number="3">}</a></code></pre></div>
</div>
</div>
</div></td>
</tr>
</tbody>
</table>

### 3.7.3. Topic and Broker Configuration

Clients receive the topic filters they have to subscribe to via
configuration. They subscribe "blindly" to all of them in the given
order.

Clients must be able to filter, route or discard messages for complete
APIs. Indeed, the API subscription can be a wildcard matching all
subjects of a role. (Example: `cen-mc-rc/v1/coding/#`). In this case the
client implementation must discard all messages that are not of
interest, according to the subject, destination system or conversation
id.

In particular, they must filter out messages that are not addressed to
themselves, i.e. when `DEST_SYSTEM` and `DEST_INSTANCE` are not matching
the client. Note that `DEST_SYSTEM` and `DEST_INSTANCE` set to question
mark (`?`) match all clients.

The configuration contains a list of brokers with their node addresses,
the topics filters for publication and subscription.

| Field         | Description                                                                                                                                                                                                                                                                                                                             |
|---------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| mqtt          | Main configuration container.                                                                                                                                                                                                                                                                                                           |
| brokers       | List of configured brokers. At least one broker is configured.                                                                                                                                                                                                                                                                          |
| clientId      | MQTT Client ID to use when connecting to this broker.                                                                                                                                                                                                                                                                                   |
| nodes         | List of DNS names or IP addresses for this broker and TCP port. Clients must use one of them and fallback to the others when connection is not possible. At least one node is provided.                                                                                                                                                 |
| publications  | The topic prefixes that this broker handles for publications from this client. This informs the client which broker must be used for publishing on topics. These are not regexp nor MQTT topic filters, just a prefix strings. This can be absent or empty in the case the configured client is not intended to publish on this broker. |
| subscriptions | The topic filters the client must subscribe to. These are MQTT topic filters. Clients subscribe to all of them and dispatch the messages internally. Can be absent or empty.                                                                                                                                                            |

  

**Example**

``` js
"mqtt": {
    "brokers": [
        {
            "clientId": "ASL01",
            "nodes": [ 
                 { 
                    "host": "10.34.44.22", 
                    "port": 3333
                 },
                 { 
                    "host": "vp034k", 
                    "port": 3334
                 },
             ],
            "publications": [ "vsi-process-control/v1", "cen-sortplan/v1" ],
            "subscriptions": [ "vsi-process-control/v1/control/#", "cen-sortplan/v1/control/#" ]
        },
        {   
            "clientId": "ASL01",
            "nodes": [ 
                 { 
                    "host": "10.34.44.22", 
                    "port": 3333
                 }
            ],
            "publications": [ "cen-mc-rc/v1" ],
            "subscriptions": [ "cen-mc-rc/v1/coding/transmit-mailpiece-attributes/+/+", 
                               "cen-mc-rc/v1/coding/transmit-mailpiece-attributes/+/+/ASL/+/+" ]
        }
    }
}
```

  

### 3.7.4. Retained Messages

Certain topics contain messages describing a current state that is
updated by it's corresponding source system using push requests. Often,
we'll want these messages to be sent with the MQTT retained flat set to
true so the broker can cache the latest state and send them to clients
on re-subscribe.

All topics that are supposed to receive retained messages are flagged
with a prefix **🔂\[RETAINED\]** in the API definition. It is the
responsibility of the corresponding source system sending the message to
set the retained flag accordingly.

The broker will be set up to persist retained messages and reload them
on possible broker restarts.

Note that retained message topics will usually still contain the source
system and instance,
e.g. *vsi-process-control/v1/client/status/{SRC\_SYSTEM}/{SRC\_INSTANCE}*.
Subscribing clients should therefore subscribe on the topic using
wildcards to avoid coupling to the source system.

It is the responsibility of the broker to implement a adequate handling
for source system renames or the like on a retained topic.

### 3.7.5. Empty Messages

Because of the mechanism used to delete retained messages, clients must
accept and silently discard messages that have no payload.

## 3.8. Message Format

1.  Messages are formatted in [JSON](https://www.json.org/) \[4\].
    Character encoding of the messages is **UTF-8**.
2.  API definitions describe message types using [JSON
    Schema](http://json-schema.org/) v4 \[5\].
3.  All messages in a given topic use the same message type.
4.  Topics with same API, VERSION and SUBJECT contain messages of the
    same type.
