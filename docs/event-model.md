# Event model

Foundry describes what a Lightning node does. It never describes what a picture should do.

Foundry emits `channel.opening`. Ooga Booga Land decides that gorillas build a production
line. If an event name implies an animation, it is the wrong name.

Status: v0.1. Vocabulary is deliberately small and will grow from real LND output, not from
imagination.

## Two schemas, not one with a filter

| | [`foundry-event.v1`](../schemas/foundry-event.v1.schema.json) | [`foundry-public-event.v1`](../schemas/foundry-public-event.v1.schema.json) |
|---|---|---|
| Audience | the operator's own machine | anyone, including the Lightning Factory |
| Carries channel ids, pubkeys, balances | yes | **structurally cannot** |
| Timing | exact | bucketed; rebalances hourly, released when the hour closes |
| Amounts | exact, in msat or sat | bucketed magnitude |
| `id` | ULID / UUIDv7, sortable | random UUIDv4, **never** time-ordered |

The public schema is a separate document rather than a redacted view of the internal one.
A redaction step is a filter, and filters leak the one field somebody forgot to add. A
schema with no field capable of holding a channel id cannot leak a channel id.

**Why this matters.** Channel balances are private for a reason: discovering them is what
probing attacks are *for*. A public feed announcing "this channel is low on outbound" tells a
jamming adversary exactly where to strike, tells a competitor how to price against you, and
does it in real time. So the public vocabulary has no liquidity events at all, buckets
timestamps, and buckets amounts.

**Rebalances are published, under constraints.** A rebalance happens *because* a channel was
low, so it is the one public event that touches liquidity. The schema makes three things
impossible to express: which line it was, a time finer than the hour, and a success rate. The
export adds a fourth that no schema can express: rebalance events are held until their hour
closes, then released after every other event from that hour — because an event labeled
"sometime this hour" but emitted the instant it happens is timed exactly by its position in
the stream. The contract tests check all four: the first three by feeding the schema events
that try, the fourth against the examples until there is an export to test. The risks that
remain are set out in [`integrations/oogabooga.md`](integrations/oogabooga.md).

**Why public ids are random.** ULID and UUIDv7 are right for the internal stream — unique
without coordination, sortable by time. That same property makes them a leak in public: both
embed a millisecond timestamp in their leading bits, which would publish the exact event time
right next to a bucket field pretending to hide it. Public events get fresh random UUIDv4s,
and never reuse an internal id, which would link the two streams.

This constrains the Factory, deliberately. A visualization can show that a node is busy,
that channels open and close, that forwards succeed and fail, that maintenance happens. It
cannot show which channel is running dry, and it should not want to.

## Envelope

Every internal event carries:

| Field | Meaning |
|---|---|
| `schema` | `foundry.event.v1` |
| `id` | UUIDv7 or ULID — unique and sortable. **Not** a counter |
| `seq` | monotonic per node, assigned by the event store — a gap means an event was filtered out or missed |
| `time` | when it happened (the node's or chain's clock) |
| `emitted` | when Foundry emitted it — later than `time` on replay, on backfill, and for a derived event about the past |
| `node` | operator-chosen pseudonym, never a pubkey |
| `source` | `lnd`, `chain`, `foundry`, `simulation` |
| `origin` | `observed` or `derived` |
| `stream` | `live` or `replay` |
| `type` | the vocabulary below |
| `payload` | typed per event |

Two of these deserve explanation.

**`id` is not a counter.** A sequential `evt-0042` collides across restarts and across
sources, and it publishes your event volume to anyone counting. UUIDv7 and ULID are both
unique without coordination and sort by time, which is what a counter was really for.

**`origin` is the invariant made structural.** `observed` means LND or the chain said so.
`derived` means Foundry computed, estimated or assumed it. `liquidity.low` is always derived,
because LND has no such event — it is Foundry comparing a balance against a threshold an
operator chose. A consumer must never present a derived value as a node fact, and having the
distinction in the envelope means it cannot be lost in transit.

## Vocabulary, and where it comes from in LND

The vocabulary is grounded in LND's actual API surface. An event nobody can populate from a
real node is a fiction that a simulator will happily keep alive.

| Event | LND source | Origin |
|---|---|---|
| `node.started` / `node.ready` / `node.stopped` | `State.SubscribeState` (`RPC_ACTIVE`, `SERVER_ACTIVE`), `GetInfo` | observed |
| `peer.connected` / `peer.disconnected` | `SubscribePeerEvents` | observed |
| `channel.opening` | `SubscribeChannelEvents` → `PENDING_OPEN_CHANNEL` | observed |
| `channel.active` | `SubscribeChannelEvents` → `OPEN_CHANNEL` / `ACTIVE_CHANNEL` | observed |
| `channel.closing` | `PendingChannels` waiting-close, or a local close request | observed |
| `channel.closed` | `SubscribeChannelEvents` → `CLOSED_CHANNEL`, `ClosedChannels` | observed |
| `forward.settled` | `SubscribeHtlcEvents` (forward + settle), `ForwardingHistory` | observed |
| `forward.failed` | `SubscribeHtlcEvents` (`LinkFailEvent`, `ForwardFailEvent`) | observed |
| `liquidity.low` / `liquidity.recovered` | **none** — derived from `ListChannels` balances vs. an operator threshold | derived |
| `rebalance.started` / `succeeded` / `failed` | circular payments (`TrackPayments`, `ListPayments`), whichever rebalancing engine made them | observed (the payment) / derived (the decision) |
| `economics.snapshot` | **none** — computed per `docs/economics.md` | derived |

Three of seventeen event types have no LND equivalent. That is the honest shape of this
system: most of what Foundry says is a fact, and the interesting part is an opinion.

### Notes on specific events

**`forward.failed` carries a `failure` reason**, and `insufficient_liquidity` is the
economically valuable one: demand arrived, and the node could not serve it. That is a missed
sale, and it is the signal that justifies a rebalance. Jet already reasons about missed
forwards this way.

**`rebalance.failed` has `fee_paid_msat`.** Failed attempts still cost money. A schema that
assumed failure was free would quietly understate the cost of liquidity management, which is
half of the operating margin.

**A rebalance Foundry did not ask for is still a fact.** A rebalancing engine running on its
own makes circular payments Foundry never targeted. Foundry observes them like any other and
publishes them under the same rules; an engine never publishes anything itself. See
[`integrations/lightning-jet.md`](integrations/lightning-jet.md).

**`channel.closed` carries `close_type`.** A cooperative close and a force close differ by an
order of magnitude in cost and by a timelock in recovery. Economics cannot treat them alike.

## Amounts and identifiers

- Amounts routed and paid are **msat**, because that is what LND reports and rounding to
  sats at the edges loses real fee revenue. Capacities and on-chain fees are **sat**.
- Field names carry their unit: `amount_msat`, `capacity_sat`. No bare `amount`.
- Channel ids are **decimal strings**, not numbers. LND's short channel id is a uint64 and
  exceeds JSON's safe integer range; a parser that reads it as a number will silently corrupt
  it.

## Reading the public stream

Four things a consumer needs that the field names do not say:

**Order by `seq`, never by `bucket`.** Buckets are coarse, and rebalance buckets are hourly, so
a later event can legitimately carry an earlier bucket. `seq` is the order.

**`scale` is a power-of-ten bucket, in sats:**

| `scale` | sats |
|---|---|
| `dust` | under 10,000 |
| `small` | 10,000 – 99,999 |
| `medium` | 100,000 – 999,999 |
| `large` | 1,000,000 – 9,999,999 |
| `very_large` | 10,000,000 and up |

It measures whatever the event is about — a forward's amount, a channel's capacity, a
rebalance's size. Channels will mostly read `large` and forwards mostly `small` or `medium`,
which is the real shape of a routing node, so a visualization should scale within an event
type rather than across them.

**`slot` rotates every 24 hours, at 00:00 UTC.** Within a day the same slot is the same line,
so an animation stays coherent. Across days slots are reassigned, so a label cannot become a
long-term identifier for a channel. Rotation is a speed bump rather than a wall — an observer
can still try to re-link lines by their behavior — which is why nothing sensitive is attached
to a slot in the first place.

**`activity.summary` covers the hour ending at its bucket**, emitted at the top of each hour:
how many forwards the node attempted in that hour, the fraction that succeeded, and its peer
and channel counts at the time.

## Delivery

The schemas govern what an event may **say**. This section governs how it **travels**, and
the two are separate protections that fail separately:

- **Content** — the public schema carries nothing harmful even when read by everyone.
- **Transport** — only this node can publish as itself, anything altered in transit fails
  verification, and nobody can pull events out of the node.

Be precise about what transport can promise. Once Ooga Booga Land renders an event in the
Factory, **it is public by design** — anyone visiting the site sees it. Transport protects the
path from the node to OBL; it does not and should not hide the destination. That is exactly
why the content is designed adversarially: the public schema has to be safe to publish,
because publishing is the point.

### The internal stream never leaves the machine

Enforced by architecture rather than configuration. Only the export component translates
internal events to public ones, it is the only component with an outbound path, and it holds
no LND credential. See [`architecture.md`](architecture.md). There is no code path that sends
an internal event anywhere.

### Rules for the public stream

1. **Push, never pull.** The node sends each batch as an HTTPS POST to OBL's ingest endpoint.
   **The node listens on nothing.** There is no endpoint on the node that serves events — not
   to OBL, not to anyone — so there is no inbound surface to attack, scan or misconfigure.
2. **Encrypted in transit.** TLS 1.3 to the ingest endpoint. Where the receiver terminates
   TLS with a key it controls, that key is **pinned** in the node's configuration, so a
   compromised certificate authority or a hostile network cannot sit in the middle. Where a CDN
   terminates TLS with certificates it rotates, as Cloudflare does, the node validates the
   certificate the ordinary way. Either way the signature in rule 4 is what makes a forged
   batch fail: whoever sits in the middle can read events that are public on arrival, or delay
   and drop batches, which the gaps in `seq` expose, but cannot forge one or instruct the
   node.
3. **The node authenticates to OBL.** A per-node credential issued at enrollment, so OBL accepts
   events only from nodes it has enrolled. Without it, anyone could post events claiming to be
   `ooga`.
4. **Events are signed.** Each batch is signed by a dedicated **export key**. OBL verifies the
   signature before accepting anything, and because the signature travels with the batch,
   anything downstream can verify provenance too.
5. **The export key is never the node's Lightning key.** Signing with the LND identity key
   would cryptographically bind the pseudonymous `node` name to the node's public pubkey,
   undoing the pseudonym in a single step. The export key is generated for this purpose alone
   and has no relationship to any Lightning key.
6. **Replay and gaps.** `seq` and `id` let OBL reject duplicates and detect missing events.
7. **Batched on a fixed schedule.** Batches go out at fixed intervals whether or not anything
   happened, so an observer watching the node's network sees a constant pattern and learns
   nothing about activity from connection timing. Rebalances held to the close of their hour
   go out in the first batch after it.
8. **Opt-in.** Export is off unless the operator enables it.

### What a compromise of the delivery path costs

The export component holds an export key and a per-node credential, and **nothing else** — no
macaroon, no seed, no access to the internal store beyond the events it translates.
Compromising it lets an attacker **forge public events** for that node. It cannot move funds,
read balances, or reach LND. That bounded worst case is the reason export is a separate
component rather than a feature of Core.

### Wire format

One POST per scheduled batch, carrying a signed batch envelope — node, key identifier, a range
of events, signature. The response is an acknowledgement and nothing else: the highest `seq`
OBL has accepted. The export resends anything after it in the next scheduled batch and acts on
nothing else in the response, so the one connection the node makes can never become a way to
instruct it.

The exact envelope is specified alongside the export implementation. The M1 simulator
exercises it end to end against a reference receiver in this repository, and can drive a
simulated feed to the cave with no node present.

The ingest endpoint that receives the feed, and the leg from OBL to a visitor's browser, are
OBL's to build and secure — see
[OBL's side of delivery](integrations/oogabooga.md#obls-side-of-delivery).

## Versioning

`schema` is the contract, and the two schemas version differently because their audiences
differ. Both are closed at every level — `additionalProperties` is `false` throughout — so a
consumer validating strictly rejects any field it was not told about. That rules out the
usual "add fields freely, ignore what you don't recognize" convention, and each schema needs
its own answer.

**Internal — `foundry.event.v1`.** The schema ships with the code that emits it, and its
consumers are Foundry's own components, released together. Fields may be added and new event
types introduced within v1, provided the schema file changes in the same commit as the
producer. Removing or renaming a field, changing a unit, or changing what a type means requires
`foundry.event.v2`.

**Public — `foundry.public.event.v1`.** **Every change is a new version, additions included.**
Consumers are other people's software on their own schedules, so a strict consumer of v1 would
reject a field added to v1. More importantly, a new public field is a new disclosure, and it
deserves the scrutiny of a version bump rather than arriving as a minor addition nobody
reviewed. The public schema should change rarely, and every change should be visible.

In either schema, changing what an existing event *means* while keeping its name is the one
change that breaks consumers silently. Prefer a new type.

## What is not here yet

Fee policy changes, HTLC-level detail, peer scoring, on-chain sweeps, watchtower activity,
payments the node receives or makes for its own purposes, such as donations and payouts, and
anything about channel *opening decisions* as opposed to channel opening *facts*. Each
arrives when something real produces it.
