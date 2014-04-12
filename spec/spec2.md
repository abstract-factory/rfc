# Immutable Versioning Pattern

This document describes a polylithic method of tracking change.

* Name: http://rfc.abstractfactory.io/spec/2
* Editor: Marcus Ottosson <marcus@abstractfactory.io>
* Tags: versioning
* Related: RFC3, RFC4
* State: raw

Copyright and Language can be found in RFC1

# Change Process

This document is governed by the [Consensus-Oriented Specification System](http://www.digistan.org/spec:1/COSS) (COSS).

# Goals

The opposite of versioned is monolithic and may thus be referred to as polylithic. In a monolithic process, there are no record of events or change and all that exists at any given point in time is here and now.

The monolithic approach has a few problems; mainly that it discourages change as change would imply a new permanent state.

In a versioned process, change is "layered"; meaning that no change affects previous state. This encourages change as it completely eliminates the cost of making mistakes.

# Definition

* Refering to past, present and future states MUST be explicit.

# Reference Implementation

Versioning may take the form of increasing numerical suffixes to files and folders in the form of:

`name:separator:version`

Where `name` is a short identifier for the product in development, `separator` being a unique character visually separating head from tail and `version` an increasing integer; higher numbers meaning later versions.

`myAsset_v1`

It can sometimes be helpful to maintain a fixed-length on the number of integers in versioning so as to simplify parsing.

>>> product = 'myAsset_v001'  # A Python string
>>> version = int(product[:-3])

Always assuring that the last n number of characters represents the version makes it possible to perform simple string-manipulation techniques to derive a version from any given product.

[Consensus-Oriented Specification System (COSS)]: http://www.digistan.org/spec:1/COSS
[RFC 2119]: http://tools.ietf.org/html/rfc2119
[versioning]: http://en.wikipedia.org/wiki/Software_versioning