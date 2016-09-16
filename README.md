# mysql-x-protocol-specification

This document is to provide a description of my thoughts about the
MySQL X protocol.

## Overview

This page is to give some ideas and thooughts about the MySQL X protocol.
This is the new MysQL protocol that was introduced in 5.7.12 as
part of the document store functionality. The same protocol can
handle traditional SQL statements.

## Where to find informaation on the protocol.

* https://dev.mysql.com/worklog/task/?id=8338 - WL#8338: X Plugin
* https://dev.mysql.com/worklog/task/?id=8639 - WL#8639: X Protocol
* https://dev.mysql.com/doc/internals/en/x-protocol.html in Chapter 15 of the MySQL Internals Manual

## Comparison of old and new formats

* https://dev.mysql.com/doc/internals/en/x-protocol-comparison-comparison-to-mysql-c-s-protocol.html

## Information where Oracle talks about X Protocol

Blog posts
* https://www.percona.com/blog/2016/05/27/asynchronous-query-execution-mysql-5-7-x-plugin/

## No Protocol Specification

The first thing is there is no specification. The MySQL Internals
documentation is basic and gives you an idea of how the protocol
can be used. It does not tell you exactly how the protocol behaves.
The worklogs are quite detailed and are a good start but are not
written as a specification.

A good guide for this would e a RFC type format which is more formalised.

This document tries to identify what things seem to be unclear and
would be good in such a documentation.

## Protobuf format

There SHOULD be a link to the raw protobuf files as these define
the message format that's wrapped within each message sent over the
wire.  Currently this information is mixed in within the code in
the mysql-server code base here:
https://github.com/mysql/mysql-server/tree/5.7/rapid/plugin/x/protocol.
Ideally the MySQL X protocol specification should be kept OUTSIDE
of the code base and should be versioned seperately from it so the
changes can be seen more easily.

## Message formats

The Protobuf message format is described in the previous section yet when the
message is sent over the wire it is prefixed with a 4-byte little endian
length indicator + 1-byte message type.

We need to have a table that relates each message type with each protobuf
message. That's easy but it should also be clearly defined.

## Timing

There is no information on what either end SHOULD do regarding
timting.  A message sent by one side is expected to arrive at the
other end. How long should the other end wait by default and can
these values be changed? This is not defined. Defining these timers
helps speed up recovery when errors occur.

## Sizes

Message sizes because of the X protocol can be up from 1 to 2^32-1
bytes of protobuf encoded messages. Therefore the actual messages
sizes of the raw message that's encoded is not clear. This may be
problematic for some users who want to fill the packet as much as
possible.

What minimum size of packet is allowed? This should be specified.
While the default setting in 5.7.12 seems to show mysqlx_max_allowed_packet
= 1048576 what does this value actually refer to? The packet including
the length indicator? The protobuf packet size?

https://dev.mysql.com/doc/refman/5.7/en/x-plugin-system-variables-options.html#option_mysqld_mysqlx_max_allowed_packet

is not clear if the 4 byte length is included in this size.

The minimum allowed size should be specified.

We have experienced in the past synchronisation issues between a
client and server or master and slave because of inconsistencies with this setting.
I use a value for max_allowed_packet which is significantly larger than
the default. So having a recommended practice to catch a mismatch
before it happens (and exchange values) would be useful.

Having a better way to solve this would be good.

## Session termination

The old protocol had COM_CLOSE. It's not clear what the new protocol has.

## Protocol Violation

What happens if the client or server thinks the protocol has been
violated?  Since we do not currently have any specification of the
behaviour what should happen? Ideally it would be good to "indicate"
this to the other end. How?

I would suggest some sort of packet for closing a session. Normally
this would be a clean "close" but a protocol violation could trigger
an indication to the other end that it was sending something
incorrectly.

e.g..

`Close (OK [default])`, or 
`Close (ERROR, with perhaps optional message?)`

It might be best if these messages are as short as possible (as a
problem may have happened).  After sending this message the end
sending it SHOULD immediately close the socket.

## General thoughts

The protocol seems a bit noisy, perhaps more noisy than normal.
Given that every message must first be decoded from it's protobuf
format this requires extra work.  So while it's good to split up
things into clearly separate parts perhaps it is not necessary to
do it as much as has been done and perhaps that triggers a higher
overhead. (Not measured so don't know.)

## Sending queries.

### general thoughts

### How to I send prepared messages etc?



* 31/05/2016 - I've sent a message to oracle. Let's see what they say.

### Why multiple column meta data packets?

I notice the protocol sends back one or more column meta data
packets, and then follows this by one packet per row. Why send each
column meta data separtely ? This seems inconsistent with the way
the row data is sent.

I'd suggest sending a single column metadata packet consisting of
a repeated number of single column meta data entries.

### query flow

This does not seem to match the 


### What happens on errors

What happens when there are errors? Right now I see that an error
can be sent after sending the query. Can an error be generated at
any time later?

### Why use NOTICE when this can come at any time?

The notice message seems to be sent to send "rows_affected". Yet
it's also a message type which can arrive at any time. Mixing the
2 is confusing and makes the coding less obvious.  If you need to
send the rows affected as a separate message send it via its own
message type.

I'd expect this information to be an optional value to the added
to the end of the SQL_STMT_EXECUTE_OK message.

## Testing

Right now there are no defined behaviours for testing. I see there
are test suites for the different Oracle drivers but it seems helpful
to provide some of these examples as part of the protocol
specification. e.g sample inputs and outputs which are not
code specific. This allows testing different parts of the specification
without actually needing to understand the unit test code which
may change over time.

The sort of things that would be useful here are:
* authentication for each of the understood authentication mechanisms

## more detail in the current messages

The current message descriptions on dev.mysql.com do not provide
complete information on the expected message flow.

e.g. sending a query ends u with more messages than mentioned:

* [RESULTSET_COLUMN_META_DATA] x number of columns
* [RESULTSET_ROW]
* [RESULTSET_FETCH_DONE]
* [NOTICE]   << ---- not mentioned.
* [SQL_STMT_EXECUTE_OK]

e.g. while authenticating 

* an extra notice message is sent.

## Copyright

Copyright (C) Simon Mudd 2016
