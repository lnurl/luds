# LNURLcash address management (non-normative design note)

This note is not part of LUD-25. It records the security requirements that
must be resolved before a portable register/update/unregister API is added.

## Boundary

LUD-25 can define receiving on a provisioned `cx1` branch and discovery via
`text/xpub` without defining how a `SERVICE` allocates usernames. A branch
proof demonstrates control of note keys; it does not establish a right to a
human-readable username or ownership of an `npub`.

The existing experimental message `LNURLcash:register:<username>` is not a
safe update authorization. It does not cover the replacement `cx1`, optional
`npub`, or `SERVICE` origin, so a captured signature can be replayed with a
substituted record.

## Required properties

A future management protocol should:

1. advertise its management callback instead of assuming a fixed `/p` path;
2. use an HTTPS request body rather than putting proofs in the URL;
3. normalize the service origin and username before signing;
4. bind the protocol version, service origin, action, username, complete
   resulting record and a short-lived one-use service challenge;
5. consume the challenge atomically with the mutation;
6. require a new branch's index-0 key to prove possession on registration;
7. require both authorization by the current branch and possession of the
   replacement branch on update;
8. require authorization by the current branch on unregister; and
9. return ordinary, deterministic LNURL success/error envelopes without
   exposing whether unrelated usernames or branches exist.

One possible unambiguous signed message is:

```text
LNURLcash-address-management-v1
origin=<canonical https origin>
action=<register|update|unregister>
username=<normalized ASCII local-part>
cx1=<normalized new cx1, or empty for unregister>
npub=<normalized new npub, or empty>
challenge=<32-byte lowercase hex>
```

The allowed value grammars exclude newlines. A create request carries a
`newBranchProof`; an update carries `currentBranchAuthorization` and
`newBranchProof`; an unregister carries `currentBranchAuthorization`.

## Accounts and privacy

A published `cx1` exposes every derived note key on its branch for public
enumeration and status/value correlation. Multiple receiving identities
should not share one published branch.

Adding a hardened account component beneath the per-service branch and above
the public note index is cryptographically appropriate, but inserting it into
the existing path would make current notes unrecoverable under the new rule.
Any account derivation therefore needs an explicit version and a migration
story. The existing branch should remain the legacy/default recovery path;
additional account branches can be introduced without changing `cx1`'s wire
payload because a watcher needs only the resulting public key and chain code.
