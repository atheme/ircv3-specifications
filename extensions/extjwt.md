---
title: IRCv3 `extjwt` extension
layout: spec
work-in-progress: true
copyrights:
  -
    name: Darren Whitlen
    period: 2018
    email: darren@kiwiirc.com
---
## Introduction

IRC networks and clients often provide external web hosted services for their user base such as forums, wikis and pastebins. However, without extra development work on the IRC network and the external service they usually require separate user authentication.

This feature provides a way for the IRC network to offer proof that a user is connected with specific permissions and / or is joined to any channels, so that the external service can use this proof to authenticate users without any development ties between the IRC network and the external service such as using XMLRPC or custom bots or shared database access.

Once in use, this:
1. Makes it easier to deploy an external service as it does not need access to your IRC server
2. Improves security in that a misconfigured or vulnerable external service cannot impact the IRC network or user accounts
3. Provides a common method to verify user status on a channel (joined, channel modes) while using well known libraries and methods that are available for many languages and frameworks (JWT)

### How the proof works

The IRC network creates the proof by generating JWT tokens and sending them to the client. The client may use this token to open an external service with the token in its URL.

The JWT token (https://jwt.io/) is an encoded JSON payload that is signed with a shared secret string between the IRC server and the external service. The JSON payload consists of known properties (claims) that include:
* `exp` `1529917513` Expiry time for this token. Usually less than 1 minute from the token generation.
* `iss` `"irc.example.org"` The server name that generated this token.
* `nick` `"somenick"` The nick of the user that generated this token.
* `account` `"somenick"` The account name of the user that generated this token. Empty if not logged in.
* `net_modes` `["o"]` User modes the IRCd wishes to disclose. Eg, if the user an operator.

When an external service is opened with this token in its URL, the external service verifies that the token has not been tampered with using its pre-configured secret string and can then use the available claims to create any required user accounts and log the user in automatically.

## Usage

If the feature is available on the IRC server, the `EXTJWT` token is added to its ISUPPORT list.

Only one new command is introduced in this extension, `EXTJWT`.

### The EXTJWT Command

Syntax: `EXTJWT [channel]`

Response syntax: `EXTJWT <requested_target> [*] <JWT_token>`

The client may send `EXTJWT` or `EXTJWT *` to the server to request a new JWT token. The server must then reply with `EXTJWT *` and a JWT token as its second parameter, containing the following claims that are relevant to the client at that time:

* `exp` Number; Unix timestamp for when this token expires. Usually less than 1 minute from the token generation.
* `iss` String; The server name that generated this token.
* `nick` String; The nick of the client that requested this token.
* `account` String; The account name of the user that requested this token. Empty if not available.
* `net_modes` []String; An array of user modes the IRCd may want to disclose. Eg, if the user is an operator.

The command must also support a single parameter of a channel name. Eg. `EXTJWT #channel`. The server must then reply with the channel name as its first parameter and the JWT token containing the above claims and also the following claims relevant to the channel at that time:

* `channel` String; The channel name this token is related to.
* `joined` Boolean; True if the client that requested this token is joined to the channel.
* `time_joined` Number; The time in which the user joined the channel.
* `modes` []String; An array of the channel modes the client has in this channel.

The IRC server must include the above claims but may include any extra claims.

#### Handling long responses

In some cases the encoded token may be longer than the maximum line length allowed between the client and server. In this case, the first parameter of the response must be `*` to indicate that further data will follow. The final chunk of the response sent to the client must not include `*` as the first parameter.

Eg:
~~~
[C -> S] EXTJWT #channel
[S -> C] EXTJWT #channel * eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1Mjk5MTc1MTMsImlzcyI6ImlyYy5leGFtcGxlLm9yZyIsIm5pY2siOiJ0ZXN0bmljayIsImFjY291bnQiOiJ0ZXN0bmljayIsIm5ldF9tb2RlcyI6W10sImNoYW5uZWwiOiIjY2hhbm5lbCIsImpvaW5lZCI6dHJ1ZSwidGltZV9qb2luZWQiOjE1Mjk5MTc1MDEsIm1vZGVzIjpbIm8iXSwiY2xhaW0xIjoic29tZSBsb25nIHZhbHVlIiw
[S -> C] EXTJWT #channel * iY2xhaW0yIjoic29tZSBsb25nIHZhbHVlIiwiY2xhaW0zIjoic29tZSBsb25nIHZhbHVlIiwiY2xhaW00Ijoic29tZSBsb25nIHZhbHVlIiwiY2xhaW01Ijoic29tZSBsb25nIHZhbHVlIiwiY2xhaW02Ijoic29tZSBsb25nIHZhbHVlIiwiY2xhaW03Ijoic29tZSBsb25nIHZhbHVlIiwiY2xhaW04Ijoic29tZSBsb25nZXIgdmFsdWUgdG8gbWFrZSBzdXJlIHRoaXMgdG9rZW4gaXMgdG9vIGxvbmc
[S -> C] EXTJWT #channel gdG8gc2VuZCBvbiBvbmUgSVJDIDUxMiBjaGFyYWN0ZXIgbGluZSJ9.c9_pKy1jFsDeevja7o6spPa-JUyzg4z4k3A65fxwZWw
~~~

## Examples

All examples may be verified using the secret of "your-256-bit-secret".

#### A client logged into the IRC server with operator privileges
~~~
[C -> S] EXTJWT *
[S -> C] EXTJWT eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1Mjk5MTc1MTMsImlzcyI6ImlyYy5leGFtcGxlLm9yZyIsIm5pY2siOiJzb21lbmljayIsImFjY291bnQiOiJzb21lbmljayIsIm5ldF9tb2RlcyI6WyJvIl19.NREHeoO-aewAry44erDgCHuVmUW9zyJjG05mJYCXXfs
~~~

Where the replied token is decoded into:
~~~json
{"exp":1529917513,"iss":"irc.example.org","nick":"somenick","account":"somenick","net_modes":["o"]}
~~~

#### A client connected to the IRC server without a registered account or operator privileges
~~~
[C -> S] EXTJWT *
[S -> C] EXTJWT * eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1Mjk5MTc1MTMsImlzcyI6ImlyYy5leGFtcGxlLm9yZyIsIm5pY2siOiJzb21lbmljayIsImFjY291bnQiOiIiLCJuZXRfbW9kZXMiOltdfQ.Vkm2XJXHz6rkq-R93fJUp88kNmAU9J65w46ZsQLjJrY
~~~

Where the replied token is decoded into:
~~~json
{"exp":1529917513,"iss":"irc.example.org","nick":"somenick","account":"","net_modes":[]}
~~~

#### A client logged into the IRC server and has channel operator privileges on a channel
~~~
[C -> S] EXTJWT #channel
[S -> C] EXTJWT #channel eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1Mjk5MTc1MTMsImlzcyI6ImlyYy5leGFtcGxlLm9yZyIsIm5pY2siOiJ0ZXN0bmljayIsImFjY291bnQiOiJ0ZXN0bmljayIsIm5ldF9tb2RlcyI6W10sImNoYW5uZWwiOiIjY2hhbm5lbCIsImpvaW5lZCI6dHJ1ZSwidGltZV9qb2luZWQiOjE1Mjk5MTc1MDEsIm1vZGVzIjpbIm8iXX0.jd1VHnEN02mSw4g2BfB-gYOooktpua2HSd9qtcUBZ4M
~~~

Where the replied token is decoded into:
~~~json
{"exp":1529917513,"iss":"irc.example.org","nick":"testnick","account":"testnick","net_modes":[],"channel":"#channel","joined":true,"time_joined":1529917501,"modes":["o"]}
~~~