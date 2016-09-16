* MySQL X Protocol Specification

This document is to provide a description of my thoughts about the
MySQL X protocol.

## Overview

This page is to give some ideas and thooughts about the MySQL X
protocol.  This is the new MysQL protocol that was introduced in
5.7.12 as part of the document store functionality. The same protocol
can handle traditional SQL statements.

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
