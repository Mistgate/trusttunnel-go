# trusttunnel-go

The wire format of [TrustTunnel](https://github.com/TrustTunnel/TrustTunnel) in Go: the pieces a client or a server
needs to speak the protocol, with no transport and no dependencies outside the standard library.

It is an independent implementation written from the public specification of AdGuard's TrustTunnel. It contains no
code of that project. License: Apache-2.0.

```
go get github.com/Mistgate/trusttunnel-go
```

```go
import trusttunnel "github.com/Mistgate/trusttunnel-go"
```

## What is in it

| Part | API |
|---|---|
| `_udp2` datagram frames, both directions, with a streaming decoder | `EncodeUDPClientFrame`, `EncodeUDPServerFrame`, `UDPDecoder` |
| `Proxy-Authorization: Basic` credentials, read the way the official server reads them | `ParseProxyAuthorization`, `BuildProxyAuthorization` |
| CONNECT `:authority`, including the pseudo-hosts `_check`, `_udp2` and `_icmp` | `ParseAuthority` |
| `tt://` deep links, version 2 | `EncodeDeepLink`, `DecodeDeepLink`, `DeepLink` |
| QUIC variable-length integers, as the deep link uses them | `AppendQUICVarint`, `ReadQUICVarint` |

## What is not in it

- **No transport.** There is no HTTP/2 or HTTP/3 server or client here, no TLS, no tunnel loop. You bring those.
- **No ICMP tunnel.** `_icmp` is recognised as an authority and nothing more.
- **No client-random anti-DPI logic.** The deep link carries its fields; nothing here acts on them.

## How it is tested

The format is defined by what the official client and server do, so the readers accept what they accept, and the
tests pin each rule with bytes written out by hand: frame layouts, length limits, every credential and authority
form that must be taken or refused, deep link vectors. Round-trip tests alone would not show compatibility: a codec
that is wrong in the same way on both sides passes them.

Mistgate's server, built on this package, is also run against the official `trusttunnel_client` in an isolated lab,
over HTTP/2 and HTTP/3.

```
go test ./...
```

## Differences from the official implementation

- The deep link **decoder** takes every tag of the format. The **encoder** is stricter than the format allows: exactly
  one address (none when a subscription URL is set) and at most two IPv4 DNS upstreams.
- The decoder of server frames takes a payload of up to 65527 bytes, as the official client does, which is a little
  more than the server ever sends; the encoder never produces more than 65508.

## Who uses it

[Mistgate](https://github.com/Mistgate/mistgate), a self-hosted VPN panel, serves TrustTunnel from its nodes with its
own Go server built on this package, and keeps the same files in `internal/trusttunnel`. The server itself is part of
Mistgate (AGPL-3.0) and is not in this repository.

TrustTunnel is a project of AdGuard. This repository is not affiliated with or endorsed by AdGuard.
