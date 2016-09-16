# Thinking about the future

I think that the x protocol is currently designed to pipeline commands
but not to runs stuff in parallel.

It would be good to allow this running of queries asynchronously.

What changes would be needed?

## Protocal changes

If you try to send more queries concurrently what happens?
I'd expect to get an error indicating that the request was not accepted

`error: too many concurrent queries running`

but other than that things should progress as usual

## Server side

In theory on the server side some sort of control on the list of parallelism.
This might be by the addition of some new variables:

`mysqlx_max_concurrent_queries = 5` // default ? maybe default should be 1 but then
people need to change the configuration to use this feature? So having a default
of > 1 would allow some concurrency out of the box.

Also via the extension of the grant syntax?

`... WITH MAX_CONCURRENCY X` ?

would limit the maximum number of concurrent sessions that you can do.


## Hinting on authentication of various parameters

it would be good to get some information on certain settings as part of the 
pipelined authentication, thus avoiding us having to request these variables
explicitly.


There appears to be already a set of variables which you can ask the server to tell you stuff

| session_track_schema | ON
| session_track_state_change | OFF
| session_track_system_variables | time_zone,autocommit,character_set_client,character_set_results,character_set_connection
| session_track_transaction_info | OFF

However, this set of values may not be what you want in which case you need to send a new command to get the new values.
(read the docs about these new variables)




Optimistic command pipelining could be done but that would possibly trigger
errors later as an unexpected command might come through if say authentication
failed. It would probably be ignored with an error: `not authenticated`?

## Way to avoid common problems with the legacy protocol

* no ping command ?
** no library (ping if idle) type command (client to server)
** no command / request to get the server to ping the client if the connection is idle for a period of time.
( and the client responds back )
** these should be optional but they should avoid the typical problem of a connection going away
and the server not noticing.

* no way to verify and check the max_allowed_packet settings
** in most other protocols the negotiation of the connection informs the client and server of the other's 
settings and then a common setting is agreed. (e.g. MTU) This does not happen in MySQL and by default
clients and servers can have different values. When you send a large packet it will get dropped by the
server (but you still don't know the server's setting) so you have to query this yourself.  Ideally
this information will be available to the client on connecting  (negotiating the connection) and if
a "too large" query is generated on the client the client libraries will be able to provide a 
contextual error: "tried to send a long query of X bytes but msyqlx_max_allowed_packet = Y so can
not send to server.




Do not expose version  information until logged in.
- only expose functionality
- e.g. get_capabilities does this:

capabilities {
  name: "plugin.version"
  value {
    type: SCALAR
    scalar {
      type: V_STRING
      v_string {
        value: "1.0.2"   <=======

Once you're logged in getting/showing this version info is fine. (if you need to)
