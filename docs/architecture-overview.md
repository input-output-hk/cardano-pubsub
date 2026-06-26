# Design Note: From a Layered Stack to a List-Based Architecture

This note explains the architectural direction of `cardano-pubsub` and the
reasoning behind it, at a level intended for a general technical audience. It is
a design overview, not a security analysis: detailed adversarial evaluation is
conducted internally and is not published here.

## Summary

The project began from an inherited three-layer design — gossip-based peer
sampling, a topic-navigation layer, and a dissemination layer on top. During
formal evaluation we concluded that the guarantees the upper layers needed were
difficult to uphold cleanly under that composition, and that the stack carried
more complexity and trust surface than the prototype warranted.

We therefore pivoted to a **list-based architecture**: peer sampling and
navigation collapse into an **on-chain subscription list**, the dissemination
layer is kept as-is, and the module interfaces are held fixed so the sampling
implementation can later be swapped for a research-delivered protocol without
re-architecting the layers above it.

## Where we started

The inherited design stacked three layers:

- **Peer sampling** (gossip-based). Each node keeps a small, periodically
  refreshed random view of the network.
- **Topic navigation.** Same-topic peers are organised into a structured overlay
  so the dissemination layer can reason about coverage and latency.
- **Dissemination.** A structured backbone combined with random links drawn from
  the sampling view.

A recurring theme across the upper layers is that each one substitutes
*"sample from a node's local view"* for *"sample from the whole network."* That
substitution is load-bearing: the higher-layer arguments are only as strong as
the property that local views faithfully stand in for the global network.

## Why we changed direction

Our evaluation focused on whether that load-bearing assumption actually holds
across the composed stack, and on the trust and complexity each layer added.
Without going into specifics, we found that:

- The properties the upper layers relied on were **not ones we were comfortable
  depending on** under the composed design, and the available fixes did not
  compose cleanly — improving one layer's property tended to weaken another's.
- The layers were **structurally at odds**: one was designed to avoid imposing
  structure, while another deliberately introduced it, and the costs of
  inheriting properties up the stack were not free.
- The overall **trust surface and complexity** were higher than a first
  prototype needs, and several of the hardest questions had no finished
  specification.

The detailed analysis behind these conclusions is kept internal. The takeaway
for this repository is the architectural one: we preferred a design with a
**smaller, more explicit trust model** over one whose guarantees were hard to
establish.

## The list-based architecture

Peer sampling and navigation collapse into an **on-chain subscription list**.
Each entry corresponds to a subscribed node and carries that node's set of topic
interests. Given the list:

- **Sampling becomes a local computation.** To find peers for a topic, a node
  filters the list rather than relying on what other peers choose to tell it.
- **Topic membership and roles are verifiable.** They are anchored on-chain, so
  peers establish who may participate from on-chain state rather than trusting a
  central authority.
- **Dissemination is unchanged.** The messaging layer above sampling keeps the
  same interface, so this is a substitution underneath it, not a redesign.

A small bootstrap set is treated as trusted infrastructure during early stages.
That assumption is deliberately explicit, narrow, and revisable.

## Alternatives considered

- **Repair and extend the layered stack.** Resolve the composition problems and
  re-derive the upper-layer guarantees without the assumption we could not rely
  on. No finished specification existed for this path.
- **Run the sampling layer once per topic.** Recovers per-topic behaviour if the
  underlying sampling issues are resolved, but scales poorly with the number of
  topics a node subscribes to.
- **Collapse sampling and navigation behind an on-chain list (chosen).**
  Simplest to ship, removes the load-bearing assumption, and keeps the
  dissemination layer untouched.

## Trade-offs of the chosen design

The list-based approach is a deliberate trade, not a free win:

- **On-chain footprint.** Subscribe, unsubscribe, and interest updates are
  transactions; on-chain state grows with the number of active subscribers.
- **List-view integrity.** A node that trusts a single source for list state can
  be misled about it; mitigations include multi-source verification, light-client
  sync, or running a local follower.
- **Privacy.** Subscriber identities and topic interests are durable and publicly
  aggregable on-chain — acceptable for operator-class participants, more
  significant for private subscribers.
- **Visibility.** Holding the full list gives each participant broad visibility
  into membership; reducing that is precisely what the research direction below
  targets.

## Research direction

The list-based design is a strong first step, not the end state. The list is an
interface we want to be able to retire without re-architecting its consumers.
The open research problem is:

> Given a network in which each node is associated with a subset of topics,
> design a protocol that lets each node **sample uniformly at random from the
> subscribers of a given topic**, without requiring the node to hold the full
> subscription list.

A protocol that delivers this primitive can replace the list-based sampling
implementation behind the fixed module interfaces, shrinking both the on-chain
footprint and the amount of global membership any single participant can see.
