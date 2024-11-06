---
v: 3
ipr: trust200902
docname: draft-ietf-lamps-rfc6712bis-latest
cat: std
obsoletes: '6712 9480'
consensus: 'true'
submissiontype: IETF
lang: en
pi:
  toc: 'true'
  tocdepth: '4'
  symrefs: 'true'
  sortrefs: 'false'
title: >
  Internet X.509 Public Key Infrastructure --
  HTTP Transfer for the Certificate Management Protocol (CMP)
abbrev: HTTP Transfer for CMP
area: sec
wg: LAMPS Working Group
keyword:
- CMP
- HTTP
- Certificate management
- PKI
github: "lamps-wg/cmp-updates"
latest: "https://lamps-wg.github.io/cmp-updates/draft-ietf-lamps-rfc6712bis.html"
date: 2024
author:
- name: Hendrik Brockhaus
  org: Siemens
  abbrev: Siemens
  street: Werner-von-Siemens-Strasse 1
  code: '80333'
  city: Munich
  country: Germany
  email: hendrik.brockhaus@siemens.com
  uri: https://www.siemens.com
- name: David | von Oheimb
  org: Siemens
  abbrev: Siemens
  street: Werner-von-Siemens-Strasse 1
  code: '80333'
  city: Munich
  country: Germany
  email: david.von.oheimb@siemens.com
  uri: https://www.siemens.com
- name: Mike Ounsworth
  org: Entrust
  abbrev: Entrust
  street: 1187 Park Place
  region: MN
  code: '55379'
  city: Minneapolis
  country: United States of America
  email: mike.ounsworth@entrust.com
  uri: https://www.entrust.com
- name: John Gray
  org: Entrust
  abbrev: Entrust
  street: 1187 Park Place
  region: MN
  code: '55379'
  city: Minneapolis
  country: United States of America
  email: john.gray@entrust.com
  uri: https://www.entrust.com
informative:
  RFC9480:
  RFC9483:
  RFC2510:
  RFC4210:
  RFC5246:
  RFC6712:
  RFC7296:
  RFC8446:
  RFC9530:
  RFC9205:
  RFC9293:
normative:
  RFC1945:
  RFC8615:
  RFC9110:
  RFC9112:
  I-D.ietf-lamps-rfc4210bis:
  ITU.X690.1994:

--- abstract


This document describes how to layer the Certificate Management Protocol
(CMP) over HTTP.

It includes the updates to RFC 6712 specified in RFC 9480 Section 3. These
updates introduce CMP URIs using a Well-known prefix. It obsoletes
RFC 6712 and together with I-D.ietf-lamps-rfc4210bis and it also obsoletes
RFC 9480.

--- middle

# Introduction {#sect-1}

[RFC Editor: please delete:

During IESG telechat the CMP Updates document was approved on condition that
LAMPS provides a RFC6712bis document.  Version -00 of this document shall
be identical to RFC 6712 and version -01 incorporates the changes specified
in CMP Updates Section 3.

A history of changes is available in Appendix A of this document.

The authors of this document wish to thank Tomi Kause and Martin Peylo, the
original authors of RFC 6712, for their work and invite them, next to further
volunteers, to join the -bis activity as co-authors.

]

The Certificate Management Protocol (CMP) {{I-D.ietf-lamps-rfc4210bis}} requires a well-defined
transfer mechanism to enable End Entities (EEs), Registration
Authorities (RAs), and Certification Authorities (CAs) to pass
PKIMessage structures between them.

The first version of the CMP specification {{RFC2510}} included a brief
description of a simple transfer protocol layer on top of TCP.  Its
features were simple transfer-level error handling and a mechanism to
poll for outstanding PKI messages.  Additionally, it was mentioned
that PKI messages could also be conveyed using file-, E-mail-, and
HTTP-based transfer, but those were not specified in detail.

Since the second version of the CMP specification {{RFC4210}} incorporated
its own polling mechanism and thus the need for a transfer protocol
providing this functionality vanished.  The remaining features CMP
requires from its transfer protocols are connection and error
handling.

CMP can benefit from utilizing a reliable transport as CMP requires connection and error handling
from the transfer protocol. All theses features are covered by HTTP.  Additionally,
delayed delivery of CMP response messages may be handled at transfer level,
regardless of the message contents.  Since {{RFC9480}} extends the polling
mechanism specified in the second version of [CMP](#RFC4210) to cover
all types of PKI management transactions, delays detected at application
level may also be handled within CMP, using pollReq and pollRep messages.

The usage of HTTP (e.g., HTTP/1.1 as specified in {{RFC9110}} and {{RFC9112}}) for transferring CMP messages exclusively uses the POST
method for requests, effectively tunneling CMP over HTTP. While this is
generally considered bad practice (see [BCP 56](#RFC9205) for best current
practice on building protocols with HTTP) and should not be emulated, there
are good reasons to do so for transferring CMP.  HTTP is used
as it is generally easy-to-implement and it is able to traverse
network borders utilizing ubiquitous proxies.  Most importantly, HTTP
is already commonly used in existing CMP implementations.  Other HTTP
request methods, such as GET, are not used because PKI management
operations can only be triggered using CMP's PKI messages, which need
to be transferred using a POST request.

With its status codes, HTTP provides needed error reporting
capabilities.  General problems on the server side, as well as those
directly caused by the respective request, can be reported to the
client.

As CMP implements a transaction identification (transactionID), identifying transactions spanning
over more than just a single request/response pair, the statelessness
of HTTP is not blocking its usage as the transfer protocol for CMP
messages.

## Changes Made by RFC 9480
{: id="sect-1.1"}

[CMP Updates](#RFC9480) updated {{Section 3.6 of RFC6712}}, supporting the PKI
management operations specified in the [Lightweight CMP Profile](#RFC9483), in the following areas:


* Introduce the HTTP URI path prefix '/.well-known/cmp'.

* Add options for extending the URI structure with further segments and
  define a new protocol registry group to that aim.


## Changes Made by This Document
{: id="sect-1.2"}

This document obsoletes {{RFC6712}}.
It includes the changes specified in {{Section 3 of RFC9480}} as
described in {{sect-1.1}} of this document, removed the requirement to support HTTP/1.0 {{RFC1945}} in accordance with {{Section 4.1 of RFC9205}} and removed {{Section 3.8 of RFC6712}} as it contains information redundant with current HTTP specification.



# Conventions Used in This Document {#sect-2}

{::boilerplate bcp14-tagged}

# HTTP-Based Protocol {#sect-3}

For direct interaction between two entities, where a reliable
transport protocol like [TCP](#RFC9293) is available, HTTP SHOULD be utilized for
conveying CMP messages.

## HTTP Versions
{: id="sect-3.1"}

This draft requires uses of the POST method ({{sect-3.3}}) and the "Content-Type" header field ({{sect-3.4}}) which are available since HTTP/1.0 {{RFC1945}}. This specification also specifies use of persistent connections ({{sect-3.2}}). This document refers to HTTP/1.1 as specified in {{RFC9110}} and {{RFC9112}} for further details.

<!-- Implementations MUST support at least HTTP/1.0 {{RFC1945}}. This is because
the POST method and the "Content-Type" and "Connection: keep-alive" header fields are available since
version 1.0.

Implementations SHOULD support HTTP/1.1 as specified in {{RFC9110}} and {{RFC9112}}. This is because the
persistent connection was improved with HTTP/1.1 which helps
transferring messages in transactions with more than one request/response
pair more efficiently, see {{Section 9.3 of RFC9112}} for persistent connections and {{Appendix C.2.2 of RFC9112}} for interoperability with the Keep-Alive feature in HTTP/1.0. -->


## Persistent Connections
{: id="sect-3.2"}

HTTP persistent connections ({{Section 9.3 of RFC9112}}) allow multiple interactions to
take place on the same HTTP connection.  However, neither HTTP nor
the protocol specified in this document are designed to correlate
messages on the same connection in any meaningful way; persistent
connections are only a performance optimization.  In particular,
intermediaries can do things like mix connections from different
clients into one upstream connection, terminate persistent
connections, and forward requests as non-persistent requests, etc.
As such, implementations MUST NOT infer that requests on the same
connection come from the same client (e.g., for correlating PKI
messages with ongoing transactions); every message is to be evaluated
in isolation.


## General Form
{: id="sect-3.3"}

A DER-encoded {{ITU.X690.1994}} PKIMessage ({{Section 5.1 of I-D.ietf-lamps-rfc4210bis}}) MUST be sent as the
content of an HTTP POST request.  If this HTTP request is
successful, the server returns the CMP response in the content of the
HTTP response.  The HTTP response status code in this case MUST be
200 (OK) status code; other Successful 2xx status codes MUST NOT be used for this purpose.
HTTP responses to pushed CMP announcement messages described in {{sect-3.7}}
utilize the status codes 201 and 202 to identify whether the received
information was processed.

While Redirection 3xx status codes MAY be supported by
implementations, clients should only be enabled to automatically
follow them after careful consideration of possible security
implications.  As described in {{sect-5}}, 301 (Moved Permanently) status code
could be misused for permanent denial of service.

<!-- Implementations SHOULD use CMP Status Codes and Failure Information according to {{Section 5.2.3 of I-D.draft-ietf-lamps-rfc4210bis}} for error handling. -->
All applicable Client Error 4xx or Server Error 5xx status codes
MAY be used to inform the client about errors.
<!-- Any content contained in such response message SHOULD be provided by the client to the CMP application. -->


## Header Fields
{: id="sect-3.4"}

The Internet Media Type "application/pkixcmp" MUST be set in the HTTP
"Content-Type" header field when conveying a PKIMessage.

<!-- Note that the PKIMessage type is used also when sending an announcement
message.

In line with {{Section 8.6 of RFC9110}}, the "Content-Length" header
field should be provided whenever a PKIMessage is sent, giving the
length of the ASN.1 DER-encoded PKIMessage. -->


## Communication Workflow
{: id="sect-3.5"}

In CMP, most communication is initiated by the EEs where every CMP
request triggers a CMP response message from the CA or RA.

The CMP announcement messages described in {{sect-3.7}} are an
exception.  Their creation may be triggered by certain events or done
on a regular basis by a CA.  The recipient of the announcement only
replies with an HTTP status code acknowledging the receipt or
indicating an error, but not with a CMP response.

If the receipt of an HTTP request is not confirmed by receiving an
HTTP response, it MUST be assumed that the transferred CMP message
was not successfully delivered to its destination.


## HTTP Request-URI
{: id="sect-3.6"}

Each CMP server on a PKI management entity supporting HTTP or HTTPS transfer
MUST support the use of the path prefix '/.well-known/' as defined in
{{RFC8615}} and the registered name 'cmp' to ease interworking
in a multi-vendor environment.

CMP clients have to be configured with sufficient information to form
the CMP server URI.  This is at least the authority portion of the URI, e.g.,
'www.example.com:80', or the full operation path segment of the PKI management
entity. Additionally, path segments MAY be added after the registered
application name as part of the full operation path to provide further distinction.
The path segment 'p' followed by an arbitraryLabel \<name> could, for example,
support the differentiation of specific CAs or certificate profiles. Further
path segments, e.g., as specified in the Lightweight CMP Profile {{RFC9483}},
could indicate PKI management operations using an operationLabel \<operation>.
The following list examples of valid full CMP URIs:




> http://www.example.com/.well-known/cmp

> http://www.example.com/.well-known/cmp/\<operation>

> http://www.example.com/.well-known/cmp/p/\<name>

> http://www.example.com/.well-known/cmp/p/\<name>/\<operation>


Note that https can also be used instead of http, see [item 5 in the Security Considerations](#sect-5).


## Pushing of Announcements
{: id="sect-3.7"}

A CMP server may create event-triggered announcements or generate
them on a regular basis.  It MAY utilize HTTP transfer to convey them
to a suitable recipient.  In this use case, the CMP server acts as an
HTTP client, and the recipient needs to utilize an HTTP server.  As
no request messages are specified for those announcements, they can
only be pushed to the recipient.

If an EE wants to poll for a potential CA Key Update Announcement or
the current CRL, a PKI Information Request using a General Message as
described in {{Appendix D.5 of I-D.ietf-lamps-rfc4210bis}} can be used.

When pushing announcement messages, PKIMessage structures MUST be sent as
the content of an HTTP POST request.

Suitable recipients for CMP announcements might, for example, be
repositories storing the announced information, such as directory
services.  Those services listen for incoming messages, utilizing the
same HTTP Request-URI scheme as defined in {{sect-3.6}}.

The following types of PKIMessage are announcements that may be pushed by a
CA.  The prefixed numbers reflect ASN.1 tags of the PKIBody structure ({{Section 5.1.2 of I-D.ietf-lamps-rfc4210bis}}).

~~~~
   [15] CA Key Update Announcement
   [16] Certificate Announcement
   [17] Revocation Announcement
   [18] CRL Announcement
~~~~

CMP announcement messages do not require any CMP response.  However,
the recipient MUST acknowledge receipt with an HTTP response having
an appropriate status code and an empty content.  When not receiving
such a response, it MUST be assumed that the delivery was not
successful.  If applicable, the sending side MAY try sending the
announcement again after waiting for an appropriate time span.

If the announced issue was successfully stored in a database or was
already present, the answer MUST be an HTTP response with a 201 (Created)
status code and an empty content.

In case the announced information was only accepted for further
processing, the status code of the returned HTTP response MAY also be
202 (Accepted).  After an appropriate delay, the sender may then try
to send the announcement again and may repeat this until it receives
a confirmation that it has been successfully processed.  The
appropriate duration of the delay and the option to increase it
between consecutive attempts should be carefully considered.

A receiver MUST answer with a suitable 4xx or 5xx error code
when a problem occurs.


<!-- ## HTTP Considerations
{: id="sect-3.8"}

While all defined features of the HTTP are available to
implementations, they should keep the protocol utilization as simple
as possible.  For example, there is no benefit in using chunked
Transfer-Encoding, as the length of an ASN.1 sequence is known when
starting to send it.

There is no need for the clients to send an "Expect" request header
field with the "100-continue" expectation and wait for a 100 (Continue) status code
as described in {{Section 10.1.1 of RFC9112}}.  The CMP
content sent by a client is relatively small, so having extra
messages exchanged is inefficient, as the server will only seldom
reject a message without evaluating the content. -->



# Implementation Considerations {#sect-4}

Implementers should be aware that other implementations might exist that
use a different approach for transferring CMP over HTTP.
Further, implementations based on earlier I-Ds that led to
{{RFC6712}} might use an unregistered "application/pkixcmp-poll" Media Type.
Conforming implementations MAY handle this type like "application/pkixcmp".

# Security Considerations {#sect-5}

The following aspects need to be considered by implementers and
users:


1. There is the risk for denial-of-service attacks through resource
  consumption by opening many connections to an HTTP server.
  Therefore, idle connections should be terminated after an
  appropriate timeout; this may also depend on the available free
  resources.  After sending a CMP error message with PKIStatus other than "waiting", the server should
  close the connection, even if the CMP transaction is not yet
  fully completed.

1. Without being encapsulated in effective security protocols, such
  as Transport Layer Security (TLS) {{RFC5246}} or {{RFC8446}}, or
  without using HTTP digest {{RFC9530}} there is no
  integrity protection at the HTTP level.  Therefore,
  information from the HTTP should not be used to change
  state of the transaction, regardless of whether any mechanism was
  used to ensure the authenticity or integrity of HTTP messages
  (e.g., TLS or HTTP digests).

1. Client users should be aware that storing the target location of
  an HTTP response with the 301 (Moved Permanently) status code
  could be exploited by a meddler-in-the-middle attacker trying to
  block them permanently from contacting the correct server.

1. If no measures to authenticate and protect the HTTP responses to
  pushed announcement messages are in place, their information
  regarding the announcement's processing state may not be trusted.
  In that case, the overall design of the PKI system must not
  depend on the announcements being reliably received and processed
  by their destination.

1. CMP provides inbuilt integrity protection and authentication.
  The information communicated unencrypted in CMP messages does not
  contain sensitive information endangering the security of the PKI
  when intercepted.  However, it might be possible for an
  eavesdropper to utilize the available information to gather
  confidential personal, technical, or business critical information.
  The protection of the confidentiality of CMP messages together with
  an initial authentication of the RA/CA before the first CMP message
  is transmitted ensures the privacy of the EE requesting
  certificates. Therefore, users of the HTTP transfer for CMP messages
  should consider using HTTP over TLS according to {{RFC9110}} or using virtual
  private networks created, for example, by utilizing Internet
  Protocol Security according to {{RFC7296}}.


# IANA Considerations {#sect-6}

The reference to {{RFC2510}} at https://www.iana.org/assignments/media-types/media-types.xhtml should be replaced with a reference to this document.

The reference to {{RFC4210}} at https://www.iana.org/assignments/core-parameters/core-parameters.xhtml should be replaced with a reference to this document.

The reference to {{RFC9480}} at https://www.iana.org/assignments/well-known-uris/well-known-uris.xhtml and https://www.iana.org/assignments/cmp/cmp.xhtmlshould should be replaced with a reference to this document.

No further action by the IANA is necessary for this document or any anticipated
updates.


# Acknowledgments {#sect-7}

The authors wish to thank Tomi Kause and Martin Peylo, the
original authors of {{RFC6712}}, for their work.

We also thank all reviewers for their valuable feedback.


--- back

# History of Changes {#History}

Note: This appendix will be deleted in the final version of the document.

From version 07 -> 08:


* Addressed HTTPDIR, SECDIR, OPSDIR and ARTART review comments

* Aligned the terminology with https://httpwg.org/admin/editors/style-guide

* Implemented editorial changes proposed by OPSDIR reviewer

* Removed requirement to support HTTP/1.0

* Added normative language in Sections 3.3 and 3.7 for clarity

* Removed the paragraph on the "Content-Length" header field and Section 3.8 to reduce redundancy with current versions HTTP/1.1


From version 06 -> 07:


* Updated the the page header to 'HTTP Transfer for CMP'

* Removed one instruction to RFC Editors

* Deprecated PKIMessages as plural of PKIMessage to prevent confusion with ASN.1 type PKIMessages

* Fixed some nits in Section 1

* Aligned Section 3.6 and Section 5 with RFC 9483 and draft-ietf-anima-brski-ae

From version 05 -> 06:


* Updates IANA considerations addressing IANA early review (see thread "[IANA #1368693] Early review: draft-ietf-lamps-rfc4210bis-12 (IETF 120)").

From version 04 -> 05:


* Added IANA considerations addressing IANA early review.

From version 03 -> 04:


* Aligned with released RFC 9480 - RFC 9483.

From version 02 -> 03:


* Fixing one formatting nit.

From version 01 -> 02:


* Updated Section 3.4 including the requirement to add the content-length filed
  into the HTTP header.

* Added a reference to TLS 1.3.

* Addressed idnits feedback, specifically changing the following RFC references:
  RFC2616 -> RFC9112; RFC2818 -> RFC9110, and RFC5246 -> RFC8446

From version 00 -> 01:


* Performed all updates specified in CMP Updates Section 3.

Version 00:

This version consists of the text of RFC6712 with the following changes:


* Introduced the authors of this document and thanked the authors of RFC6712
  for their work.

* Added a paragraph to the introduction explaining the background of this document.

* Added the change history to this appendix.

