
# EMQ distribution over TLS

EMQ cluster supports distribution over TLS since 2.3.5 release.

Notice that this feature requires Erlang/OTP R20.2 to build if you install EMQ from source.

## Configure `node.proto_dist`

Configure the `node.proto_dist` and `node.ssl_dist_optfile` parameters in etc/emq.conf.

```
## Specify the erlang distributed protocol.
##
## Value: Enum
##  - inet_tcp: the default; handles TCP streams with IPv4 addressing.
##  - inet6_tcp: handles TCP with IPv6 addressing.
##  - inet_tls: using TLS for Erlang Distribution.
##
## vm.args: -proto_dist inet_tcp
node.proto_dist = inet_tls

## Specify SSL Options in the file if using SSL for Erlang Distribution.
##
## Value: File
##
## vm.args: -ssl_dist_optfile <File>
node.ssl_dist_optfile = etc/ssl_dist.conf


https://www.erlang-solutions.com/blog/erlang-distribution-over-tls.html
```

## Configure `etc/ssl_dist.conf`

The SSL options should be configured in `etc/ssl_dist.conf`, for example:

```
[{server,
  [{certfile, "etc/certs/cert.pem"},
   {keyfile, "etc/certs/key.pem"},
   {secure_renegotiate, true},
   {depth, 0}]},
 {client,
  [{secure_renegotiate, true}]}].
```

## More information

[Using SSL for Erlang Distribution](http://erlang.org/doc/apps/ssl/ssl_distribution.html)

