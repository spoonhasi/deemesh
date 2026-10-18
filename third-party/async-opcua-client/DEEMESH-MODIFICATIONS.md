# Modifications by deemesh

This directory is async-opcua-client 0.18.0 (https://crates.io/crates/async-opcua-client, part of
https://github.com/freeopcua/async-opcua), licensed under the Mozilla Public License 2.0 (see `LICENSE`).
deemesh-sdk and deemesh-hub are built with this modified copy.

## Change (2026-10-16)

`src/transport/state.rs`, in `end_issue_or_renew_secure_channel`: the lifetime of an issued or renewed secure
channel token is counted from the moment the response arrived, on this machine's clock, instead of from the
creation time the server reported. When the server's clock was ahead of the client's by more than half the token
lifetime, the original code never renewed the token, and the server closed the channel when the token expired.
The changed lines are marked with the comment "deemesh modification".

No other file of the crate was changed. `LICENSE` and this file were added.
