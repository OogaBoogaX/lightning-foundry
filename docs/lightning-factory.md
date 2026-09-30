# Lightning Factory

What a consumer of Foundry's public event stream must do with it. Written for the Lightning
Factory, the cave in Ooga Booga Land that shows the OBL node at work, and binding on any other
consumer in the same way.

Foundry owns this contract; OBL owns the cave. [`event-model.md`](event-model.md) says what
the stream contains and why, and [`integrations/oogabooga.md`](integrations/oogabooga.md) says
what the Factory may show and what publishing costs. This document is the checklist between
them: the rules a consumer follows so that what it draws stays true to what the node said, and
never says more.

Status: v0.1. Its first consumer is the Lightning Factory's feed reader in Ooga Booga Land. It
changes when the schema's version does.

## Accept only the contract

- **Validate every event** against
  [`foundry-public-event.v1`](../schemas/foundry-public-event.v1.schema.json), strictly. The
  schema is closed at every level, so an unknown field is an error, not an extension.
- **Check `schema`.** Any other value is a different contract: drop the event and report it
  rather than guess. A new version arrives as a new value, never as new fields in the old one.
- **Drop, don't repair.** An event that fails validation is dropped whole and counted. A
  consumer that fixes up bad input has started inventing events.

## Order and completeness

- **Order by `seq`, per `node`.** Never by `bucket`: rebalance buckets are hourly and released
  late, so a later event can carry an earlier bucket.
- **A gap is missing events.** When `seq` skips, fetch again from the last one received. If
  the gap cannot be filled, carry on without animating what is missing. A gap is silence, not
  a hint.
- **Duplicates are harmless.** The same `seq` and `id` arriving twice is one event.

## Time

- **A bucket is a window, not a moment.** Animate an event anywhere within its minute, and
  never show a time finer than the bucket. A clock reading 17:44:05 would publish what the
  bucket exists to hide.
- **Rebalances are last hour's news.** They arrive after their hour closes. Show them as work
  done in that hour, not as something happening now.
- **`replay` is history.** Events with `stream: "replay"` retell the past, as when a page
  loads and catches up. Play them quickly and quietly; celebrations, sounds and notices are for
  `live` events.

## Identity

- **`node` names a room.** It is a pseudonym, not a pubkey, and nothing should treat it as one.
- **A `slot` lives for one UTC day.** Within the day, the same slot is the same line. At 00:00
  UTC every slot is reassigned, so rebuild the line labels, and never carry a slot's line across
  midnight: not in memory, not in localStorage, not in a URL. A consumer that remembers
  yesterday's slots rebuilds the long-term identifier the rotation exists to prevent.

## Size and volume

- **`scale` is a power-of-ten bucket.** Scale within an event type, never across types:
  channels mostly read `large`, forwards `small` or `medium`.
- **`count` is intensity.** A bucket with `count: 12` is a busy minute, not twelve events with
  times of their own.

## Liveness

Two different things, and the Factory must not blur them:

- **The node stopped.** A `node.stopped` event, observed. The factory goes dark.
- **The feed went quiet.** Batches stopped arriving on their schedule. The node may be fine and
  the path to OBL broken, so this is unknown, not stopped. Show "no signal".

## Observed and derived

`origin` travels with every event. `activity.summary` is `derived`: Foundry's count of the
hour, not something the node reported. A busier or quieter factory is a fair reading of it; a
readout that presents a derived number as the node's own is not.

## Never shown

Whatever the art suggests, and even when it would look better:

- **Balance or liquidity.** No gauge, tank level or color that reads as how full a channel is.
  Tanks, if there are any, follow throughput.
- **Why a forward failed.** The feed does not carry the reason, and the cave must not infer one.
- **Which line a rebalance served.** Rebalances belong to the machine room, never to a line.
- **Exact amounts or times.** Only `scale`, `count` and buckets exist.
- **Peer names on lines.** The node's channels are public in the network graph, and a line
  labeled with its peer undoes the slot in one step.
- **Donations on a line.** Donations can arrive in the factory, at the pile or on a cart, but
  never along a line. The leaderboard's exact amounts and times, joined to a line, would put
  back what the schema removed.

The last two are the easiest to break by accident. Each source is harmless alone; joined to the
feed, they can reconstruct a channel's balance.

## More than one node

Each node's stream stands alone: `seq` and slots are per node, and one node's events say
nothing about another's. When the Factory gains rooms for other operators, each room reads only
its own node's stream, and nothing in the Factory correlates across rooms. The multi-node risk
is set out in [`integrations/oogabooga.md`](integrations/oogabooga.md).

## Demo streams

A consumer may play a demo node before a real one exists, and because a demo's data is
invented, it may say more than the public contract allows. Two rules keep it honest:

- **Its own schema name.** A demo speaks a contract of its own, never extra fields under
  `foundry.public.event.v1`, so a real stream can never be mistaken for a demo or a demo for a
  real stream. OBL's `obl.factory.demo.v1` works this way.
- **Demo-only stays demo-only.** Whatever only the demo can express, such as a named channel,
  both lines of a forward or a fee, the real feed will never supply. The Factory must still read
  truthfully from the public stream alone, and must not fill those gaps from other sources when
  it switches over, for the reasons under Never shown.

## Building against it

- **Fixtures.** [`examples/public-events.jsonl`](../examples/public-events.jsonl) is one
  channel's life, ready to play. The M1 simulator will add seeded, repeatable streams for every
  scenario in [`roadmap.md`](roadmap.md).
- **Refusals.** Foundry's [contract tests](../tests/contract.test.mjs) show what the schema
  itself refuses. A consumer's own tests should feed it the same refusals, and confirm that it
  drops them and counts them.
