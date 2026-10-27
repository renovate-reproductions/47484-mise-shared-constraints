# Reproduction for renovatebot/renovate#47483 and #47484

Shows that artifact updates of other managers install the tool version pinned in `mise.toml`.

- [#47483](https://github.com/renovatebot/renovate/pull/47483): merges "shared constraints" from tool managers into each branch's `extractedConstraints`, scoped by directory
- [#47484](https://github.com/renovatebot/renovate/pull/47484): the `mise` manager reports its `[tools]` versions as shared constraints

## Layout

| Crate | mise scope | Expected `rust` for the `Cargo.lock` update |
| --- | --- | --- |
| `/` (`root-crate`) | `/mise.toml` | `1.86.0` |
| `nested/` (`nested-crate`) | `/nested/mise.toml` (nearest file wins) | `1.88.0` |
| `other/` (`other-crate`) | none of its own, falls under `/mise.toml` | `1.86.0` |

Each crate is a separate package (no Cargo workspace) with a committed `Cargo.lock` and an outdated dependency, so every crate gets its own branch and the artifact update runs `cargo`.
The cargo manager resolves `rust` without deriving it from the repository, so the mise pin is the only source of a version.
Only `cargo` and `mise` are enabled, and updates of `mise` itself are disabled to reduce noise.

## Expected behaviour

- On `main`: Renovate installs the latest `rust` for every cargo artifact update.
- On the PR branch: Renovate installs `rust 1.86.0` for the root and `other/` crates and `rust 1.88.0` for `nested/`.

## Verify

Check out the PR branch of renovatebot/renovate, then run:

```sh
LOG_LEVEL=debug RENOVATE_BINARY_SOURCE=install RENOVATE_TOKEN=<token> \
  pnpm start renovate-reproductions/47484-mise-shared-constraints
```

In the debug log, look for the containerbase `install-tool rust <version>` lines, or the `toolConstraints` debug output of each cargo artifact update.
