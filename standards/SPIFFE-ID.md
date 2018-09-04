# The SPIFFE Identity and Verifiable Identity Document

## Status of this Memo
This document specifies an experimental identity and identity issuance standard for the internet community, and requests discussion and suggestions for improvements. It is a work in progress. Distribution of this document is unlimited.

## Abstract
The SPIFFE standard provides a specification for a framework capable of bootstrapping and issuing identity to services across heterogeneous environments and organizational boundaries. It encompasses a variety of specifications, each defining the operation of a specific subset of the SPIFFE functionality.

This document, in particular, serves as the core specification for the SPIFFE standard. While there are other specifications falling under the SPIFFE umbrella, conformance with this document is sufficient for achieving SPIFFE compliance, and gaining the interoperability benefit of the SPIFFE standard itself.

For more general information about SPIFFE, please see the [Identity Framework for Everyone (SPIFFE)](SPIFFE.md) standard.

## Table of Contents
1\. [Introduction](#1-introduction)  
2\. [SPIFFE Identity](#2-spiffe-identity)  
2.1. [Trust Domain](#21-trust-domain)  
2.2. [Path](#22-path)  
3\. [SPIFFE Verifiable Identity Document](#3-spiffe-verifiable-identity-document)  
3.1. [SVID Trust](#31-svid-trust)  
3.2. [SVID Components](#32-svid-components)  
3.3. [SVID Format](#33-svid-format)  
4\. [Conclusion](#4-conclusion)  

## 1. Introduction
This document sets forth the official SPIFFE specification. It defines the two most fundamental components of the SPIFFE standard: the SPIFFE Identity and the SPIFFE Verifiable Identity document.

Section 2 outlines the SPIFFE Identity (SPIFFE ID) and its namespace. The SPIFFE ID is a structured string used to identify a resource or caller, and is the cornerstone of the SPIFFE standard. All other SPIFFE components focus on the issuance and verification of the SPIFFE IDs themselves.

Section 3 describes the SPIFFE Verifiable Identity Document (or SVID). An SVID is a mechanism through which a compute endpoint can present its SPIFFE ID in a way that can be cryptographically verified and deemed trustworthy. SVIDs, which reference an associated asymmetric key pair, can additionally be used to form a secure communication channel.

Conformance with this document is sufficient for the purposes of SPIFFE compliance.

## 2. SPIFFE Identity
In order to communicate an identity, we must first define an identity namespace. A SPIFFE Identity (or SPIFFE ID) is defined as an [RFC 3986](https://tools.ietf.org/html/rfc3986) compliant URI comprising a “trust domain” and an associated path. The trust domain stands as the authority component of the URI, and serves to identify the system in which a given identity is issued. The following example demonstrates how a SPIFFE ID is constructed:

```spiffe://trust-domain/path```

Valid SPIFFE IDs MUST use the `spiffe` scheme, include a non-zero trust domain, and MUST NOT include a query or fragment component. In other words, a SPIFFE ID is defined in its entirety by the `spiffe` scheme and a site-specific `hier-part` which includes an authority component and an optional path.

### 2.1. Trust Domain
The trust domain corresponds to the trust root of a system. A trust domain could represent an individual, organization, environment or department running their own independent SPIFFE infrastructure.

Trust domains are nominally self-registered, unlike public DNS there is no delegating authority that acts to assert and register a base domain to an actual legal real-world entity, or assert that legal entity has fair and due rights to any particular trust domain.

The trust domain is defined as the authority component of the URI - specifically, the `host` part of the authority. The `userinfo` and `port` parts of the authority component MUST NOT be set, and the `:` delimiter MUST NOT be present. Please see section 3.2 of [RFC 3986](https://tools.ietf.org/html/rfc3986) for more information.

### 2.2. Path
The path component of a SPIFFE ID allows for the unique identification of a given workload. The meaning behind the path is left open ended and the responsibility of the administrator to define.

Paths MAY be hierarchical - similar to filesystem paths. The specific meaning of paths is reserved as an exercise to the implementer and are outside the SVID specification. However some examples and conventions are expressed below.

* Identifying services directly

  Often it is valuable to identify services directly. For example, an administrator may decree that any process running on a particular set of nodes should be able to present itself as a particular identity. For example:

  ```spiffe://staging.example.com/payments/mysql```
  or
  ```spiffe://staging.example.com/payments/web-fe```

  The two SPIFFE IDs above refer to two different components - the mysql database service and a web front-end - of a payments service running in a staging environment. The meaning of ‘staging’ as an environment, ‘payments’ as a high level service collection is defined by the implementer.

* Identifying service owners

  Often higher level orchestrators and platforms may have their own identity concepts built in (such as Kubernetes service accounts, or AWS/GCP service accounts) and it is helpful to be able to directly map SPIFFE identities to those identities. For example:

  ```spiffe://k8s-west.example.com/ns/staging/sa/default```

  In this example, the administrator of example.com is running a Kubernetes cluster k8s-west.example.com, which has a ‘staging’ namespace, and within this a service account (sa) called ‘default’. These are conventions defined by the SPIFFE administrator, not assertions guaranteed by this specification.


* Opaque SPIFFE identity

  The above examples are illustrative and, in the most general case, the SPIFFE path may be left opaque, carrying no visible hierarchical information. Metadata, such as geographic location, logical system partitioning and/or service name, may be provided by a secondary system, where identities and their attributes are registered. That can be queried to retrieve any metadata associated with the SPIFFE identifier. For example:

  ```spiffe://example.com/9eebccd2-12bf-40a6-b262-65fe0487d453```

## 3. SPIFFE Verifiable Identity Document
A SPIFFE Verifiable Identity Document (SVID) is the mechanism through which a workload communicates its identity to a resource or caller. An SVID is considered valid if it has been signed by an authority within the SPIFFE ID's trust domain, and the presenter can prove ownership of the associated private key.

### 3.1. SVID Trust
As covered in Section 2.1, SPIFFE trust is rooted in a given ID's trust domain. A signing authority MUST exist in each trust domain, and this signing authority MUST carry an SVID of its own. The SPIFFE ID of the signing authority SHOULD reside in the trust domain in which it is authoritative, and SHOULD NOT have a path component. The SVID of the signing authority then forms the basis of trust for a given trust domain.

Chaining of trust, if desired, can be achieved by signing the authority’s SVID with the private key of a foreign trust domain’s authority. In the event that trust is not being chained, then the authority’s SVID is self-signed.

### 3.2. SVID Components
An SVID is a fairly simple construct, and comprises three basic components:

* A SPIFFE ID
* A public key
* A valid signature

The SPIFFE ID and the public key MUST be included in a portion of the payload which is signed. The corresponding private key is retained by the entity to which the SVID has been issued, and is used to prove ownership of the SVID itself.

An SVID MAY include information beyond what is described here. It is assumed, however, that the SPIFFE signing authority has validated all information contained within the SVID prior to issuing it.

### 3.3. SVID Format
An SVID is not itself a document type. Many document formats exist already which fulfil the needs of a SPIFFE SVID, and we do not wish to re-invent those formats. Instead, we define a set of format-specific specifications which standardize the encoding of SVID information.

In order for an SVID to be considered valid, it MUST leverage a document type for which a corresponding specification has been defined. At the time of this writing, the only supported document type is X.509. Note that format-specific SVID specifications may upgrade the requirements set forth in this document.

Please see the [X.509 SPIFFE Verifiable Identity Document](X509-SVID.md) specification for more information.

## 4. Conclusion
The specifications contained within this document cover what it means to be SPIFFE compliant. While other specifications will need to be referenced in order to build a complete implementation, conformance to this document is sufficient for compliance purposes.

By conforming to the SPIFFE standard, we can begin to address modern identity and authentication challenges. Namely, the issuance and consumption of identity in dynamic, heterogeneous environments. Furthermore, by defining a standardized way to encode identity into a provable document, we can bridge the identity gap between organizations and systems with disparate authentication mechanisms.
