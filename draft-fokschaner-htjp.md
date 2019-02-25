---
title: Hypertext Jeopardy Protocol (HTJP/1.0)
abbrev: Hypertext Jeopardy Protocol 1.0
docname: draft-fokschaner-htjp-latest
category: info
submissiontype: independent
date: 2019-04-01

ipr: trust200902
area: art
workgroup: Independent Submission
keyword: ["HTJP", "HTTP", "Jeopardy"]

stand_alone: yes
pi:
- toc
- sortrefs
- symrefs

author:
- ins: E. Fokschaner
  name: Edmund Fokschaner
  organization: ""
  email: edfokschaner@gmail.com

normative:
  UNIX-General-Concepts:
    title: "General Concepts, Chapter 4 of The Open Group Base Specifications Issue 7, 2018 edition IEEE Std 1003.1-2017"
    date: 2018
    target: "http://pubs.opengroup.org/onlinepubs/9699919799/basedefs/V1_chap04.html"
informative:
  I-D.draft-thomson-postel-was-wrong-03:

--- abstract

The Hypertext Jeopardy Protocol (HTJP) is a pseudo-protocol that inverts the request/response semantics of
the Hypertext Transfer Protocol (HTTP). Using conventional HTTP one connects to a server,
asks a question, and expects a correct answer. Using HTJP, one connects to a server, sends an answer,
and expects a correct question. This document specifies the semantics of HTJP.

--- middle

# Introduction

The Hypertext Jeopardy Protocol (HTJP) 1.0 is a stateless application-level response/request pseudo-protocol that functions
as the semantic inverse of the Hypertext Transfer Protocol (HTTP) 1.1 .

It can roughly be specified in relation to HTTP by the following rules:

- Where an HTTP client would send an HTTP request message, an HTJP client would send an HTTP response message.
- Where an HTTP server would send an HTTP response message, an HTJP server would send an HTTP request message.
- The HTTP request sent as an HTJP response, should be an HTTP request that (if sent to the appropriate HTTP server)
would elicit the HTTP response sent in the HTJP request.

HTJP is compatible with the HTTP/1.1 specification, at least in spirit, if not in letter.

HTJP has novel applications in all the following areas:

- Generative automated testing of HTTP implementations and HTTP-based applications.
- Monitoring of HTTP-based applications in production.
- Forensic and diagnostic reconstruction of HTTP requests from HTTP response logs.
- Discovery of first-party and third-party security vulnerabilities.


# Conventions Used In This Document
{::boilerplate bcp14}


# A Pseudo-Protocol  {#pseudo-protocol}

It is a little-known fact that HTTP/1.1 already defines itself to be its own inverse mode of operation.
[RFC 7230 Section 3.1](#RFC7230) which describes the Start Line of the HTTP Message Format, states:

> In theory, a client could receive requests and a server could receive
> responses, distinguishing them by their different start-line formats,
> but, in practice, servers are implemented to only expect a request \[...\] and
> clients are implemented to only expect a response.

It is only convention that HTTP clients send HTTP requests and that HTTP servers send HTTP responses.
Therefore, HTJP is just HTTP with some alternative conventions. It is not a distinct protocol.
Furthermore, since all messages in the HTJP protocol are indistinguishable from HTTP messages,
an HTJP peer would have no way of identifying itself explicitly as using HTJP rather than HTTP.

Therefore, we describe HTJP as a "pseudo-protocol" in order to distinguish clients, servers and conversations
that are using the HTTP conventions laid out in this document from those that use conventions which are
more commonly associated with HTTP.

# Response and Request Semantics

An HTJP request MUST be an HTTP response message. An HTJP response message MUST be an HTTP request message which,
if issued to the appropriate HTTP server, would elicit the HTTP response specified by the HTJP request being replied to.

As described in [](#pseudo-protocol) HTJP is unconventional but valid HTTP, and so the entirety of the HTTP
specification as defined in {{!RFC7230}}, {{!RFC7231}}, {{!RFC7232}}, {{!RFC7233}}, {{!RFC7234}}, and {{!RFC7235}}
MUST be respected when doing so is consistent with HTJP's unique semantics.

The following example illustrates a typical message exchange for an HTJP request concerning the same hypothetical server
from [RFC 7230 Section 2.1](#RFC7230)

~~~
Client request:

  HTTP/1.1 200 OK
  Date: Mon, 27 Jul 2009 12:28:53 GMT
  Server: Apache
  Last-Modified: Wed, 22 Jul 2009 19:15:56 GMT
  ETag: "34aa387-d-1568eb00"
  Accept-Ranges: bytes
  Content-Length: 51
  Vary: Accept-Encoding
  Content-Type: text/plain

  Hello World! My payload includes a trailing CRLF.

Server response:

  GET /hello.txt HTTP/1.1
  Host: www.example.com
~~~

## Applicability of Postel's Robustness Principle  {#postel-was-right}
Implementations of HTJP SHOULD respect Postel's Robustness Principle (which is unfortunately still in draft form) {{I-D.draft-thomson-postel-was-wrong-03}}.

Applied to HTJP, Postel's Robustness Principle implies that, given the choice of multiple valid HTJP responses for an HTJP request,
one SHOULD prefer a response which is more adherent to the HTTP standard or uses less HTTP features.
For example, sometimes a User-Agent header has no bearing on the HTTP response from an HTTP server.
On seeing such a response in an HTJP request, and HTJP server could validly respond with a practically unlimited number
of permutations on the User-Agent header value. However, it SHOULD prefer to respond with an HTTP request that has no User-Agent header whatsoever,
in keeping with Postel's Robustness Principle.

The same consideration applies when encountering an HTJP request for which there are both valid and ["pseudo-valid"](#pseudo-valid) HTJP responses available.


## Identifying the Server Associated with an HTJP response
It may be of interest to a user of HTJP to try issuing an HTJP response as an HTTP request to the appropriate server.
This brings up the issue of correctly identifying the host to which the HTJP response should be sent.
Much of the time this will be able to be determined from the Host header field of the HTJP response.
The Host header is required by conformant HTTP 1.1 requests.
In the case that the Host header is not present (for example, if the HTJP response is an HTTP 1.0 request rather than HTTP 1.1),
an HTJP response MAY use the absolute URI form in the HTTP Request Line, to add clarity about the target host if it would be validly accepted by the server.
This specific example is complicated by the fact that prior to HTTP 1.1 it was not required that implementations accept the absolute URI form.
For this reason, it is also possible to phrase the HTJP response as an HTTP request to a Forward Proxy server,
which would have accepted, indeed needed, the absolute URI Request Line prior to and after HTTP 1.1.
As a last resort, it may be possible to heuristically derive the target host of an HTJP response from the message payload,
for example, it might have absolute references to other HTTP resources that seem to come from the same host.


## Temporal Considerations
When an HTJP response is issued, there is no guarantee that, by the time the response is received by an HTJP client,
the HTTP server that is associated with said response will still reply with it.
Providing any guarantee about "when" an HTTP server would reply with said response is obviously a theoretically un-solvable problem
and therefore outside the scope of this HTJP specification.
It is only required that the HTJP response be correct at some point in the range of the 32-bit Unix Timestamp,
see [Unix General Concepts Section 4.16 "Seconds Since the Epoch"](#UNIX-General-Concepts) .

HTJP servers that are responding with an HTTP request for a volatile resource, and with high confidence in the time range at which the
resource would be in the state described by the HTJP request, MAY add a Date header.
to the HTJP response, which is in conformance with the ability for HTTP requests to carry a Date header, [see RFC 7231 Section 7.1.1.2](#RFC7231) .

HTJP clients can try to demand more temporal certainty by adding a Date header to their HTTP response,
embedding criteria for the time of the HTTP response in the HTTP response itself. Of course, the client might
still only receive that exact HTTP response if it manages to deliver the HTTP request at the precise time of the
previously requested Date header, and even then it is still not guaranteed due to HTTP caching et cetera.


## Pseudo-Valid HTJP Messages  {#pseudo-valid}
In the wild, HTTP clients and servers have been known to occasionally exchange HTTP messages which are not conformant to any HTTP specification.
For this reason, we will identify a class of messages which are, on the one hand invalid HTTP messages, yet at the same time, correctly answerable HTJP requests OR correct answers to an HTJP request, as "pseudo-valid" HTJP messages.

Consider, for example, an HTTP server that erroneously reports a Content-Length header field of zero when sending a non-zero length HTTP payload.
Despite this HTTP message violating the HTTP specification, it is possible for an HTJP server to receive such a message and correctly respond to it,
satisfying the HTJP semantics in doing so.

This applies to both HTJP request and HTJP responses. There may be times when the only valid HTJP response is an invalid HTTP request.
When there are both valid and invalid HTTP requests that could satisfy the HTJP request, Postel's Robustness Principle SHOULD be applied,
as described in [](#postel-was-right)


## Un-Requestable HTTP Responses
Given that an HTJP response MUST be an HTTP request, and that HTTP requests do not have a status field (such as a status code),
there is no way for an HTJP response to signal a failure in response to an HTJP request, via a status code or otherwise.
The correct HTJP response to an HTJP request when a server cannot determine an HTTP request that elicits the HTTP response, is to not respond at all.
The HTJP responder MAY close the connection, however the HTJP requester MUST not interpret the closing of the connection as a response.
This can have issues when HTJP servers are hosted behind non-HTJP-aware HTTP proxies as the proxy may inject a 5XX HTTP response
which could be misinterpreted as an HTJP response. See more on proxies in [](#caches-and-proxies).


# Caches and proxies  {#caches-and-proxies}
Despite being valid HTTP traffic, support for caching and proxying of HTJP traffic is unfortunately not widespread.
It is estimated that currently approximately 0.0% of all HTTP-aware intermediaries gracefully handle HTJP traffic.
For this reason, it is currently recommended that HTJP should not be deployed behind an HTTP proxying or caching layer.
Support will likely increase proportionally to HTJP's popularity in production.


# IANA Considerations
In accordance with {{?BCP90=RFC3864}}, the following new HTTP header field will be registered in the "Message Headers" registry maintained at <http://www.iana.org/assignments/message-headers/>.

| Header field name | Applicable protocol | Status        | Reference            |
| Anti-HTJP-Nonce   | http                | informational | [](#anti-htjp-nonce) |


# Security Considerations

## Securing HTTP against HTJP
A complete implementation of HTJP is inadvisable from a security perspective.
Due to its semantics, the issuing of a successfully authorized HTTP response to an HTJP server,
will result in a reply containing the HTTP request that elicits said response including any credentials,
tokens, cookies, or other authorization materials that were required to elicit that response.

As an example:

~~~
Client request:

  HTTP/1.1 200 OK
  Date: Mon, 27 Jul 2009 12:28:53 GMT
  Content-Length: 61
  Content-Type: text/plain

  Some predictable information accessed using authorization.

Server response:
(line breaks in Authorization header are for RFC formatting)

  GET /information.txt HTTP/1.1
  Host: authorised-usage-service.example.com
  Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
      eyJzdWIiOiJodGpwIiwibmFtZSI6IkV2ZXJ5b25lIiwiaWF0IjowfQ.
      JOL-kIObgTI0MzFfm1yVFFkIo1xf7DZGjY_oedRBZW0
~~~

Given that we cannot prevent anyone from attempting to implement HTJP,
it is RECOMMENDED to consider how HTJP impacts security when using HTTP.

Note that it was only possible to query for the credentialed HTTP request because the
response to the authorized request was predictable. HTTP servers could mitigate this
vulnerability exposed by HTJP by only serving a response that is at least as secret as the credentials themselves
in response to an authorized request.

### Anti-HTJP-Nonce Header {#anti-htjp-nonce}
A generic solution to this problem is to use an "Anti-HTJP-Nonce" HTTP header in HTTP responses.
The value of an "Anti-HTJP-Nonce" header SHOULD be a cryptographically-secure random number in any encoding that is valid for an HTTP header value.
The length of this number SHOULD be determined by the producer of the HTTP response, accounting for their RNG method and their threat model.

## HTJPS
HTJP, being just HTTP, also has most of the same security concerns and features as HTTP itself.
For example, the use of HTJP over an encrypted connection, such as TLS 1.2 {{?RFC5246}},
similar to HTTP Secure (HTTPS), is referred to as HTJP Secure (HTJPS). It is important to
note however that, unlike with HTTPS, it is not expected that the hostname you are securely communicating
with is the same hostname as featured in the Host headers or absolute URI's of the HTJP messages themselves,
as HTJP messages are typically referring to other HTTP hosts. This excludes the case of a server that
supports both conventional HTTP and HTJP, where it is possible to make HTJP requests securely to the same host
that is also the subject of the HTJP requests being made.

--- back

# Hypertext Double Jeopardy Protocol

Also worth mentioning, in case one encounters it in the wild, is the Hypertext Double Jeopardy Protocol (HTJ2P).
The Hypertext Double Jeopardy Protocol 1.0 is a stateless application-level request/response pseudo-protocol that functions
as the inverse of the Hypertext Jeopardy Protocol (HTJP) 1.0 .

An HTJ2P response MUST be an HTTP response which would be issued for an HTTP request delivered as the HTJ2P request.
Implementations of HTJ2P have groundbreaking potential in the fields of HTTP caching, and in the implementation of HTJP.

# Acknowledgements
{: numbered="no"}
The author wishes to thank everyone who made the effort to feign amusement on his sharing of this protocol with them.
The author also wishes to thank anyone who has ever built a tool or a technology to make people ask "Why?".
