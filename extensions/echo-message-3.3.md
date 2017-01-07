echo-message client capability specification
--------------------------------------------

Copyright (c) 2014 Attila Molnar <attilamolnar@hush.com>
Copyright (c) 2014 J-P Nurmi <jpnurmi@gmail.com>

Unlimited redistribution and modification of this document is allowed
provided that the above copyright notice and this permission notice
remains in tact.

## Description

This client capability MUST be named `echo-message`.

If enabled, servers MUST send `PRIVMSG` and `NOTICE` messages back to
the client that sent them. If servers apply any modifications to these
messages, they MUST send the final version of the message back to the
originating client.

For clients, receiving a message with themselves as the sender acts as
an acknowledgement that the message has been delivered to the server.
Clients that receive self-sent `PRIVMSG` and `NOTICE` messages, MUST
treat them the same way as if the client itself would have sent the
message to the target. Clients may choose to disable local echoing
of sent `PRIVMSG` and `NOTICE` messages altogether, or present them
in pending state.

## Use cases

The capability is useful for clients to get an acknowledgement that a
message was delivered. It can be also used for measuring lag between
client and server, that is, the time it takes from sending a message
to receiving it back.

Furthermore, the capability is useful for users that have multiple
clients attached to a bouncer. In this scenario, when users send
messages from one client, the messages get automatically relayed to
other attached clients. This allows all attached clients to display
full conversation.

## Examples

In the following examples, `example!ex@example.com` presents a client
that has enabled the `echo-message` capability.

    --> PRIVMSG Attila :hi
    :example!ex@example.com PRIVMSG Attila :hi

The client interprets the received message as if it had sent a `PRIVMSG`
with contents "hi" to `Attila`.

Another example where a server modifies a message by filtering out text
formatting and sends the final version back:

    --> PRIVMSG #ircv3 :back from \02lunch\0F
    :example!ex@example.com PRIVMSG #ircv3 :back from lunch

A client may add an `id` tag to correlate between sent messages and
echoed ones (which might have been modified by the server). If an `id`
tag is present, the server must echo it.

    --> @id=cd87d2 NOTICE jpnurmi :important note
    @id=cd87d2 :example!ex@example.com NOTICE jpnurmi :important note
