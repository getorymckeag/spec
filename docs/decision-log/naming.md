---
title: Network Element Names and Naming Authorities
parent: Decision Records
---

## Status

* Status: `Committed`
* Issue Link: [GitHub Issue](https://github.com/trolie/spec/issues/9)

## Context

The data exchange specified by TROLIE implies that the Ratings Provider and the
Transmission Provider must both agree on identifiers for Transmission Facilities
and Network Segments.  This is required for the exchange to function.

IEC CIM standards document IEC 61970-452 defines the concept of a master
resource identifier (mRID), issued by a model authority.  In theory, mRIDs may
be used for globally consistent identification of any network model object,
which would provide a shared identifier that these parties could agree on. In
Europe, this would be feasible, as CIM data exchange has become commonplace in
response to the ENTSO-E-mandated Common Grid Model Exchange Standard
([CGMES](https://www.entsoe.eu/data/cim/cim-for-grid-models-exchange/)).

There is no such prevailing standard in North America, and CIM adoption has been
limited, so we cannot assume common mRIDs across systems. While mRIDs are of
course supported in many software products, they are not necessarily used or
shared widely in data exchanges.

Therefore, while mRIDs provide architectural simplicity to TROLIE, we do not
believe that they can be mandated in order to use TROLIE. In order to integrate
systems by the FERC 881 deadline, TROLIE implementations will have to adapt to
the names that the various parties actually know and use. This ADR specifies
rules and design philosophy on usage of names in TROLIE server and client
implementations.

## Decision

The following list is a summary of rules for TROLIE implementations. Further
discussion follows:

* The **server** defines which names will be used in a given exchange.  The
  set(s) of names to use are assumed to be pre-coordinated.
* Clients must conform, in query parameters and request bodies, to the names
  expected by the server.
* TROLIE is compatible with CIM mRIDs, and mRIDs are _encouraged_, but not
  required.
* Implementations are free to provide extensions that enhance the behavior, as
  long as they do not break TROLIE compatibility. For example, `GET` Operations
  in TROLIE might be defined that allow the server to return a complete list of
  names that it knows for a given object.


### The TROLIE Server Determines the Names Used

Since the server must process the request, TROLIE servers must generally have an
internal representation of the network model they're operating against. While
servers may adapt the names they present to clients in a variety of ways,
leaving this decision to the server allows the simplest possible server designs,
where the server only knows a single identity for each object. We believe this
is a simpler path to interop than attempting to specify a richer name/identity
negotiation as part of the specification.

### CIM MRIDs are Encouraged

Implementers of TROLIE servers are encouraged to support CIM mRIDs. The
benefits to functions of the CGMES such as cross-TSO schedule exchanges are
obvious. TROLIE and other such exchanges will benefit from consistent use of
these identifiers over time.  

### Use of CIM NameTypes

In CIM, "mRID" is a property of IdentifiedObject, the base class of most types
in the CIM model. IdentifiedObject also provides some additional fields that
provide additional identifying information, including:

* `name` is intended to be a human-readable identifier.

* `aliasName` is a single alternative name. However, it is deprecated in favor
  of

* a child collection of `Name` objects, which can map to a `NameType`. This
  construct allows for an extensible set of name types, which could map to
  arbitrary names in underlying systems owned by Transmission Provider, such as
  the EMS, planning, asset registration, or names from neighboring
  authorities.

It is possible that TROLIE server implementations will know about alternative
names for various objects. Consider a server-to-server communication where both
servers have an mRID from different naming authorities. These parties might
pre-coordinate the use of both mRIDs in the `Name` collection to avoid any
ambiguity. To take another example, consider a TO with their own unique name
for a transformer while the TP uses an mRID. Assuming the TP also has the TO's
name mapped, then the TP could accept proposals that use just the TO's
proprietary name for the transformer, while including both names in snapshots.

## Consequences

While deferring some challenges to implementors, the intent of this ADR is to
aid interop and specification conformance by assigning responsibility for names
purely to the server. The tradeoff is that it may require implementors to
create extensions and other mechanisms to handle disparate naming knowledge
between Transmission Providers and Rating Providers.

This decision also implies rules for TROLIE specification design.  Specifically:

* The server may provide alternative names, based on the CIM `Name`/`NameType`
  paradigm, but only in `GET`s.  The client must never be required to provide
  them.

* These headers are represented in an `alternative-identifiers` attribute of the
  power system resource list in the header.  

These changes are reflected in the current example of [In-use forecasts](../example-narratives/in-use-forecasts.md).  