# Pocket Foundry

A phone-playable factory game in the Satisfactory tradition, running as a single
self-contained HTML page.

**Play:** https://claude.ai/code/artifact/dee01e9d-1d0f-4630-934c-51abe23e3fbd

## What it is

Everything is a rate. Machines are specified in units per minute, derived from
`quantity x 60 / cycleSeconds`, so ratios are exact: a smelter turns 30 iron ore
into 30 iron ingots a minute, and three of them feed one 90-ingot plate line.
Belts are abstracted into per-item buffers, which is what makes the game about
balancing throughput rather than routing.

- **40 items, 53 recipes, 15 building types** across smelting, construction,
  assembly, foundry, manufacturing and oil refining
- **Power grid** that throttles proportionally instead of tripping, with
  biomass, coal and fuel generation
- **Overclocking** on the real `clock^log2(2.5)` power curve, so 250% costs
  3.36x power and 50% costs 0.40x
- **Exploration**: 32 deposits across 7 sectors, none of them on the map until
  you survey for them
- **14 hub milestones** and a **4-phase Ascent Elevator** that interleave to
  pace the tier unlocks
- **Resource sink** feeding a voucher catalog of 11 alternate recipes
- Byproducts genuinely clog: refine plastic without handling heavy oil residue
  and the refinery stops
- Offline production for up to four hours, local save, portable backup codes

## Getting started

A new save opens straight into an **11-step guided tour** that runs on the live
game, not a scripted mock-up. It measures ~2.2 minutes of reading plus a tap per
step, and it is skippable at any point.

The build lesson costs nothing, which is the trick that makes it work: because
dismantling refunds 100%, the tour has you take your constructor apart and
rebuild it through the recipe picker. You learn the steppers, the refund rule
and the picker, and finish with exactly the factory you started with.

Everything meta lives in the **game menu** (the button top right): resume,
replay the tour, how it runs, backup or restore a save, and **New game**, which
wipes the factory and drops you back at the pod with the tour running.

## Exploration

You land on four deposits. Every other one is found by dispatching a survey from
the Scan tab. Survey stations clear a sector's effort, each expedition returns
one deposit plus a supply cache, and a resource you have never seen before also
teaches you how to process it:

- **Sulfur** unlocks Compacted Coal (630 MJ against coal's 300, so a generator
  burns 7.1/min instead of 15) and Black Powder
- **Raw Quartz** unlocks Silica and Quartz Crystal, leading to the Crystal
  Oscillator that a Survey Station Mk.2 is built from

Black Powder feeds back into exploration: arming charges halves an expedition's
effort. The **Geological Survey** milestone brings each sector's deeper pure
deposits into range. Surveys run offline like everything else.

## Files

- `index.html` - the whole game: markup, styles and simulation

## Verification

Five suites run against the build:

1. **Integrity and balance** - reference validity, the unlock graph, progression
   reachability band by band, 38 recipe rate assertions, generator burn rates
2. **Simulation behaviour** - 16 tests: starvation, buffer backup, brownout
   scaling, byproduct clogging, fuel exhaustion, offline/realtime drift,
   deadlock recovery, survey completion, repeat surveying, blasting, and v1
   save migration
3. **Tutorial** - walks all 11 steps, asserting every spotlight finds a visible
   target, no coach card covers the control it points at, the tour ends clean,
   does not nag on reload, and replays correctly mid-game on a different factory
4. **UI interactions** - every control on every tab driven in a real browser
5. **Soak** - hundreds of random control activations, then a save/reload round
   trip, checking for NaN, negative stock, corrupt state and stalls

Bugs these caught, in order of severity:

- **The recipe picker did nothing.** Sheets are appended to `document.body` but
  the click handler was bound to the view, so no button inside a sheet ever
  fired. This made the game unplayable past its two starting machines.
- **Infinite render recursion.** `tickUI` announced a survey result, which
  re-rendered, which called `tickUI` again with the result still set. The stack
  blew and the main thread stalled for nine seconds.
- **Two unrecoverable deadlocks.** A coal-only grid that ran dry could never
  restart, and an under-watered coal plant killed itself permanently. Fixed with
  hand-mineable ores and a 5 MW hub reserve that never runs out.
- **Three material-loss paths** - dismantling into a full buffer, refunding a
  miner upgrade, and hand-crafting with no room for the output all silently
  destroyed items.
- **A tour step nobody could finish.** The hand-mine step waited on ore *stock*
  rising, so a player replaying it with a full ore buffer was trapped forever.
  It counts the tap now, and any action step grows a "skip step" button after
  twelve seconds.
- **The Scan tab was empty before its first unlock**, so there was nothing to
  point at and no sense of what was out there. Sectors are always listed now,
  with dispatch disabled until Base Building.
