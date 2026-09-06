# rust-electrum-client with BLAKE2b proof of work

This is an unofficial fork of [rust-electrum-client](https://github.com/bitcoindevkit/rust-electrum-client) that follows the BLAKE2b proof-of-work hardfork of Bitcoin. It is not affiliated with the Bitcoin Dev Kit project. Upstream has not adopted the fork, so use upstream instead if that is what you want.

> **Not reviewed by upstream.** Everything below the divider is upstream's documentation and describes rust-electrum-client rather than this fork.

## What differs from upstream

- **Concatenated headers are split by length, not by a fixed stride.** The protocol 1.4 `blockchain.block.headers` response packs the headers into one blob, and upstream splits it with `i * 80`. Past the hardfork a header may be 164 bytes, so a fixed stride desyncs the moment an extended header appears. Each header is now parsed and the buffer advanced by what it actually consumed.
- **Two bugs fall out of that, both reachable from a remote server.** A short response used to panic on the slice index, which is a denial of service from any server the client talks to. A blob that did not tile into `count` headers used to be accepted silently and return wrong headers, because any 80 bytes parse as a header. Both are errors now.
- **The server-supplied `count` is bounded before it sizes an allocation.** Unchecked, `{"count":1000000000000}` reaches `Vec::with_capacity` and aborts the process with an uncatchable allocation failure.
- **A build guard.** `bitcoin` must be the fork carrying hardfork support. Cargo honours `[patch]` only in the workspace root, so a consumer that forgets it would otherwise compile clean and mis-parse headers at runtime, past the activation height only.

Everything else needs no change: the single-header paths hand a complete byte string to `deserialize`, which is length-agnostic, and the protocol 1.6 response sends an array rather than a blob.

## Branches

| Branch | Base | Use |
| --- | --- | --- |
| `master` | upstream `master` | The fork's line. Carries the changes above. |

Releases are tagged so a dependent can pin an immutable rev rather than a moving branch.

## Using it

```toml
[patch.crates-io]
bitcoin = { git = "https://github.com/privkeyio/rust-bitcoin", rev = "<pinned rev>" }
electrum-client = { git = "https://github.com/privkeyio/rust-electrum-client", rev = "<pinned rev>" }
```

Both are needed: this crate's fix is the header splitting, and the header parsing itself lives in `bitcoin`.

---

# rust-electrum-client

<p>
    <a href="https://crates.io/crates/electrum-client"><img src="https://img.shields.io/crates/v/electrum-client.svg"/></a>
    <a href="https://docs.rs/electrum-client"><img src="https://img.shields.io/badge/docs.rs-electrum--client-blue"/></a>
    <a href="https://blog.rust-lang.org/2023/12/28/Rust-1.75.0.html"><img src="https://img.shields.io/badge/MSRV-1.75.0%2B-orange.svg"/></a>
    <a href="https://github.com/bitcoindevkit/rust-electrum-client/blob/master/LICENSE.md"><img src="https://img.shields.io/badge/License-MIT%2FApache--2.0-red.svg"/></a>
    <a href="https://github.com/bitcoindevkit/rust-electrum-client/actions/workflows/cont_integration.yml"><img src="https://github.com/bitcoindevkit/rust-electrum-client/actions/workflows/cont_integration.yml/badge.svg"></a>
</p>

Bitcoin Electrum client library. Supports plaintext, TLS and Onion servers.

## Security Policy

To report a security issue, please refer to the [security policy](SECURITY.md).

## Minimum Supported Rust Version (MSRV)

This library should compile with any combination of features with Rust 1.75.0.

To build with the MSRV you will need to pin dependencies by running:

``` bash
cargo update -p openssl --precise "0.10.78"
cargo update -p openssl-sys --precise "0.9.114"
cargo update -p zeroize --precise "1.8.2"
cargo update -p jobserver --precise "0.1.34"
```

## License

Licensed under either of

* Apache License, Version 2.0, ([LICENSE-APACHE](LICENSE-APACHE) or <https://www.apache.org/licenses/LICENSE-2.0>)
* MIT license ([LICENSE-MIT](LICENSE-MIT) or <https://opensource.org/licenses/MIT>)

at your option.

### Contribution

Unless you explicitly state otherwise, any contribution intentionally
submitted for inclusion in the work by you, as defined in the Apache-2.0
license, shall be dual licensed as above, without any additional terms or
conditions.
