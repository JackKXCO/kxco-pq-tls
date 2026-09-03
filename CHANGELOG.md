# Changelog

## 1.2.0

Documentation and positioning. No code change: the handshake, the API and the
wire format are untouched, and all 22 tests pass unchanged.

### New TLS.md

The exact Node, nginx and OpenSSL settings to get `X25519MLKEM768` on the wire
for `relay.kxco.ai` and `chain.kxco.ai`, and — the part that actually matters —
the one command that proves the group was negotiated rather than silently
skipped.

Everything in it was measured on OpenSSL 3.5.6 with Node 26.1.0, not read off a
specification. One measured finding is worth repeating here: **on a Node
server, `groups` is a preference, not a restriction.** A Node server configured
with the PQ group only still accepts a client that offers only X25519, and
completes the handshake on a classical group. The same test against
`openssl s_server -groups X25519MLKEM768` fails the handshake with alert 40. If
you need refusal, terminate TLS in front of Node.

`getEphemeralKeyInfo()` returns `{}` for this group and cannot tell you what
was negotiated. `openssl s_client ... | grep "Negotiated TLS1.3 group"` can.
No output means no hybrid group.

### Repositioned

The README now says, at the top, that **for HTTPS you do not want this
package** — you want TLS with `X25519MLKEM768`, which is standardised, reviewed
by the whole ecosystem, and a configuration change rather than a library.

This package is for what TLS does not cover: mutual ML-DSA-65 identity over a
channel you already have. Two Node processes on a socket that is not HTTP. A
WebSocket where both ends must prove which key they hold rather than which
certificate a CA signed.

Its handshake is our own and has had no third-party review. A handshake nobody
has attacked is not one to put on a public endpoint, and saying so is more
useful than not.

### Corrected

The Security section claimed all the Noble libraries were "independently
audited by Cure53 (2024)". `@noble/curves` was audited by Trail of Bits
(Feb 2023), Kudelski (Sep 2023) and Cure53 (Sep 2024); `@noble/ciphers` by
Cure53 (Sep 2024). **`@noble/post-quantum` was audited by nobody** — it is
self-audited by its maintainer only. Corrected in the README and in
`.socket.yml`. The `quantum-safe` keyword is removed from the manifest.


## 1.0.0 — 2026-05-24

Initial release.

### Added
- `wrapStream(socket, options)` — wrap any Node.js Duplex (net.Socket, etc.) with a PQ-TLS channel
- `wrapWebSocket(ws, options)` — wrap a WebSocket (ws package or native API) with a PQ-TLS channel
- Hybrid key exchange: ML-KEM-768 (post-quantum) + X25519 (classical) combined via HKDF
- AES-256-GCM data encryption with per-sequence-number nonces (replay protection)
- Separate encryption keys per direction (C2S / S2C) — cross-channel forgery impossible
- Optional mutual authentication via ML-DSA-65 identity keys (opt-in per connection)
- Cloudflare Workers compatible (pure JS, no native addons, Web Crypto via @noble/ciphers)
- `initiatorHandshake` / `responderHandshake` — low-level API for custom transports
- 20+ tests: crypto primitives, in-memory handshake, TCP stream E2E, tamper detection
