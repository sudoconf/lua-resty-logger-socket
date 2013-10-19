Name
====

lua-resty-logger-socket - nonblocking remote access logging for Nginx

Status
======

This library is still experimental and under early development.

Description
===========

This lua library is a remote logging module for ngx_lua:

http://wiki.nginx.org/HttpLuaModule

This is aimed to replace Nginx's standard [ngx_http_log_module](http://nginx.org/en/docs/http/ngx_http_log_module.html) to push access logs to a remote server via an nonblocking socket. A common remote log server supporting sockets is [syslog-ng](http://www.balabit.com/network-security/syslog-ng).

This Lua library takes advantage of ngx_lua's cosocket API, which ensures
100% nonblocking behavior.

Synopsis
========


    lua_package_path "/path/to/lua-resty-logger-socket/lib/?.lua;;";

    server {
        location / {
            log_by_lua '
                local logger = require "resty.logger.socket"
                if not logger.initted() then
                    local ok, err = logger.init{
                        host = 'xxx',
                        port = 1234,
                        flush_limit = 1234,
                        drop_limit = 5678,
                    }
                    if not ok then
                        ngx.log(ngx.ERR, "failed to initialize the logger: ",
                                err)
                        return
                    end
                end

                -- construct the custom access log message in
                -- the Lua variable "msg"

                local ok, err = logger.log(msg)
                if not ok then
                    ngx.log(ngx.ERR, "failed to log the message: ", err)
                    return
                end
            ';
        }
    }

Methods
=======

This logger module is designed to be shared inside an nginx worker process by all the requests. So currently only one remote log server is supported. We may support multiple log server sharding in the future.

init
----
`syntax: ok, err = logger.init(user_config)`

Initialize logger with user configurations. Logger must be inited before use. If you do not initialize the logger, you will get an error.

Available user configurations are listed as follows:

* `flush_limit`

    If the buffered messages' size plus the current message size reaches (`>=`) this limit (in bytes), the buffered log messages will be written to log server. Default to 4096 (4KB).

* `drop_limit`

    If the buffered messages' size plus the current message size is larger than this limit (in bytes), the current log message will be dropped because of limited buffer size. Default drop_limit is 1048576 (1MB).

* `timeout`

    Sets the timeout (in ms) protection for subsequent operations, including the *connect* method. Default value is 1000(1 sec).

* `host`

    log server host.

* `port`

    log server port number.

* `path`

    If the log server uses a stream-typed unix domain socket, `path` is the socket file path. Note that host/port and path cannot both be empty. At least one must be supplied.

inited
--------
`syntax: inited = logger.inited()`

Get a boolean value indicating whether this module has been inited (by calling the [init](#init) method).

log
---
`syntax: ok, err = logger.log(msg)`

Log a message. By default, the log message will be buffered in the logger module until `flush_limit` is reached in which case the logger will flush all the buffered messages to remote log server via a socket.

Installation
============

You need to compile at least [ngx_lua 0.9.0](https://github.com/chaoslawful/lua-nginx-module/tags) with your Nginx.

You need to configure
the [lua_package_path](https://github.com/chaoslawful/lua-nginx-module#lua_package_path) directive to
add the path of your `lua-resty-logger-socket` source tree to ngx_lua's Lua module search path, as in

    # nginx.conf
    http {
        lua_package_path "/path/to/lua-resty-logger-socket/lib/?.lua;;";
        ...
    }

and then load the library in Lua:

    local logger = require "resty.logger.socket"

TODO
====

* Multiple log server sharding and/or failover support.

Author
======
Jiale Zhi <vipcalio@gmail.com>, CloudFlare Inc.


Copyright and License
=====================

This module is licensed under the BSD license.

Copyright (C) 2013, by Jiale Zhi <vipcalio@gmail.com>, CloudFlare Inc.

All rights reserved.

Redistribution and use in source and binary forms, with or without modification, are permitted provided that the following conditions are met:

* Redistributions of source code must retain the above copyright notice, this list of conditions and the following disclaimer.

* Redistributions in binary form must reproduce the above copyright notice, this list of conditions and the following disclaimer in the documentation and/or other materials provided with the distribution.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
