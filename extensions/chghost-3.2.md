---
title: IRCv3.2 `chghost` Extension
layout: spec
copyrights:
  -
    name: "Christine Dodrill"
    period: "2013"
    email: "xena@yolo-swag.com"
  -
    name: "Ryan"
    period: "2016"
    email: "ryan@hashbang.sh"
---

The chghost client capability allows servers to directly inform clients about a
host or user change without having to send fake quits or joins. This capability
MUST be referred to as `chghost` at capability negotiation time.

When enabled, clients will get the `CHGHOST` message to designate the host of a
user changing for clients on common channels with them.

The `CHGHOST` message is as follows:

    :nick!user@host CHGHOST new-user new.host.goes.here

The field represented by `new-user` represents the user's "username" or "ident"
which may or may not have changed in the CHGHOST process.

The field represented by `new.host.goes.here` represents the new hostname for
the user which may or may not have changed in the CHGHOST process.

## Examples

In this example, `tim!~toolshed@backyard` gets their username changed to `b` and
their hostname changed to `ckyard`:

    :tim!~toolshed@backyard CHGHOST b ckyard

In this example, `tim!b@ckyard` gets their username changed to `~toolshed` and
their hostname changed to `backyard`:

    :tim!b@ckyard CHGHOST ~toolshed backyard

## Errata

* Previous versions of this specification did not include any examples, which made
it unclear as to whether the de-facto `~` prefix should be included on CHGHOST
messages. The new examples make clear that it should be included.
* Previous versions of this specification used confusing descriptions and have
since been rewritten to include a simpler description and example.
