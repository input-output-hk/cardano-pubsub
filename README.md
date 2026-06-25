# cardano-pubsub

> Decentralized publish/subscribe messaging layer for the Cardano network, anchored on-chain and usable across Cardano and beyond.

[![Status: Experimental](https://img.shields.io/badge/status-experimental-orange)](#project-status)
[![License: Apache 2.0](https://img.shields.io/badge/license-Apache%202.0-blue)](LICENSE)

`cardano-pubsub` is a peer-to-peer publish/subscribe communication layer that lets nodes exchange messages reliably without relying on centralized brokers. Topic membership and relay (forwarder) roles are anchored on the Cardano blockchain, so peers can independently verify who may participate and route traffic from on-chain state rather than trusting a central authority.

It is built for the Cardano network first, but the messaging layer itself is chain-neutral. Anchoring is an implementation detail of where trust lives, not a constraint on where the system can run.

## Background

This work grew out of our research into secure, dependable peer sampling: the service that continuously supplies each node with a fresh, random set of live peers. Peer sampling is the foundation of any robust gossip-based overlay, and its weakest point is the ability of malicious nodes to overrepresent themselves by injecting links that point back to themselves.

Our point of origin is SecureCyclon, a peer-sampling protocol that targets that specific attack by turning node descriptors into verifiable communication certificates:

- Alexandros Antonov and Spyros Voulgaris, 'SecureCyclon: Dependable Peer Sampling', ICDCS '23, July 2023. [IOG paper page](https://www.iog.io/papers/securecyclon-dependable-peer-sampling), [PDF (arXiv 2309.02952)](https://arxiv.org/pdf/2309.02952.pdf).

`cardano-pubsub` builds on that line of work and on our broader analysis of peer-sampling protocols, applying the lessons to a production-oriented messaging layer anchored on Cardano.

## How it works

At a high level:

- Nodes subscribe to **topics** and publish messages to them. Subscribers receive messages for the topics they care about.
- Messages propagate through a **gossip-based** overlay, so delivery does not depend on any single node staying online.
- A subset of nodes can take on a **forwarder** (relay) role to help edge nodes reach the wider network. These roles are registered on-chain, so peers can verify them rather than trust them blindly.
- **On-chain anchoring** provides the registry of participants and roles, giving the network a decentralized source of truth for membership and trust.

> [!NOTE]
> The trust and forwarder model is rolled out in stages. Early deployments use a known set of peers; later stages introduce the on-chain forwarder role and the associated trust rules for edge nodes. See the [architecture decision records](node/docs/decisions/) for the design rationale to date.

## Project status

`cardano-pubsub` is **experimental and under active research and development.** Interfaces, message formats, and on-chain schemas may change without notice, and the project is not yet recommended for production use. We are validating the architecture and hardening the protocol before declaring a stable release.

Feedback and design discussion are welcome while the design settles — see [Contributing](#contributing).

> [!WARNING]
> **Important Disclaimer & Acceptance of Risk**
>
> This is a proof-of-concept implementation that has not undergone security auditing. This code is provided "as is" for research and educational purposes only. It has not been subjected to a formal security review or audit and may contain vulnerabilities. **Do not use this code in production systems or any environment where security is critical without conducting your own thorough security assessment.** By using this code, you acknowledge and accept all associated risks, and Input Output Global disclaims any liability for damages or losses.

## Getting started

> [!NOTE]
> This release is a single-process, in-memory prototype: the node registers on an in-memory network and reads its peer set, subscription list, and topic registry from local TOML files. On-chain anchoring and a real transport are future work (see [Project status](#project-status)).

### Prerequisites

- Rust 1.75 or newer (stable toolchain, 2021 edition) — install via [rustup](https://rustup.rs/)

### Build

```bash
git clone https://github.com/input-output-hk/cardano-pubsub.git
cd cardano-pubsub
cargo build --release
```

### Run a node

The node takes its own id, a node-config file, and the two registry files (the
mock stand-ins for on-chain state):

```bash
cargo run --release -- \
  --self-id node-a \
  --config <node-config.toml> \
  --subscription-list <subscription-list.toml> \
  --topic-registry <topic-registry.toml>
```

Example registry files are in [`node/tests/fixtures/`](node/tests/fixtures/). Pass `--log-level debug` for verbose output.

### Test

```bash
cargo test
```

## Documentation

- Architecture decision records: [`node/docs/decisions/`](node/docs/decisions/)
- Node crate overview: [`node/README.md`](node/README.md)
- Contributing guide: [`CONTRIBUTING.md`](CONTRIBUTING.md)

## Contributing

This repository is published as a **research artifact and is not under active external development.** Issues are open for **questions and design discussion** — feel free to open one. We are not accepting feature pull requests at this stage, and interfaces may change without notice. See [`CONTRIBUTING.md`](CONTRIBUTING.md) for details.

## License

Licensed under the Apache License, Version 2.0. See [`LICENSE`](LICENSE) for the full text.

Copyright 2026 Input Output Global.
