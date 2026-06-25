# Contributing to cardano-pubsub

Thanks for your interest in `cardano-pubsub`.

## Project posture

This repository is published as a **research and reference artifact**. It is
**experimental** and **not under active external development**. Its primary
purpose is to share the design and a working prototype with the Cardano
community and anyone working on similar peer-sampling and gossip-overlay
problems.

What this means in practice:

- **Questions and design discussion are welcome.** Open a
  [GitHub issue](https://github.com/input-output-hk/cardano-pubsub/issues) — to
  ask how something works, discuss the architecture, or report a problem you hit
  while reading or running the code.
- **We are not accepting feature pull requests at this stage.** The design is
  still settling and interfaces change without notice, so we cannot commit to
  reviewing or maintaining external contributions. Small fixes (typos, broken
  links, obvious documentation errors) are the exception and may be considered.
- **No support or maintenance guarantees.** There is no SLA on responses, and
  the project may change direction or be archived.

If you are exploring a substantial collaboration, please raise an issue first so
we can talk before you invest effort.

## Running the code

See the [Getting started](README.md#getting-started) section of the README for
build, run, and test instructions. In short:

```bash
cargo build --release
cargo test
```

## Design background

The rationale behind the implementation is recorded as architecture decision
records under [`node/docs/decisions/`](node/docs/decisions/). Reading these is
the fastest way to understand why the code is shaped the way it is.

## License

By participating in this project you agree that any contribution you submit is
provided under the project's [Apache 2.0 license](LICENSE).
