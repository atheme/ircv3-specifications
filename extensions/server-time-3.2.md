---
title: IRCv3.2 `server-time` Extension
layout: spec
copyrights:
  -
    name: "Stéphan Kochen"
    period: "2012"
    email: "stephan@kochen.nl"
  -
    name: "Alexey Sokolov"
    period: "2012"
    email: "alexey-irc@asokolov.org"
  -
    name: "Kyle Fuller"
    period: "2012"
    email: "inbox@kylefuller.co.uk"
---
Clients indicate support for the extension by requesting a capability server-time as per the [IRC Client Capabilities Extension][cap].

	CAP REQ :server-time

When enabled, the server-time extension enables optional `time` [message tag][] which can be used in messages from server to client.
The value of the tag MUST have the following syntax:

	<value> ::= YYYY-MM-DDThh:mm:ss.sssZ

It represents a calendar date and time of day in UTC using extended format, as specified by ISO 8601:2004(E) 4.3.2.
Precision of the time is one millisecond.

If the `time` is presented in a message received from server, client SHOULD treat the message as the one happened at the given time instead of now.

Servers MAY include the timestamp in messages when they see fit (in order to tell the client that the message really happened at the given time instead of now),
but MUST NOT do so before acknowledging the client capability using `CAP ACK`.
Clients MAY choose to simplify parsing by accepting timestamps at any point in the connection (e.g. even before `CAP REQ`).

Example 1:

Angel messaged Wiz at Wed, 19 Oct 2011 16:40:51.620 UTC

	@time=2011-10-19T16:40:51.620Z :Angel!angel@example.org PRIVMSG Wiz :Hello

Example 2:

John joined #chan during the leap second of June 2012 (notice the :60)

	@time=2012-06-30T23:59:60.419Z :John!~john@1.2.3.4 JOIN #chan


[cap]: ../core/capability-negotiation-3.1.html
[message tag]: ../core/message-tags-3.2.html

* * * * *

In case local client is in wrong time, timestamps looking weird can be
avoided by also supporting the `echo-message` capability. Running NTP
client is recommended.
