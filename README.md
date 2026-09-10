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

- **35 items, 47 recipes, 13 building types** across smelting, construction,
  assembly, foundry, manufacturing and oil refining
- **Power grid** that throttles proportionally instead of tripping, with
  biomass, coal and fuel generation
- **Overclocking** on the real `clock^log2(2.5)` power curve, so 250% costs
  3.36x power and 50% costs 0.40x
- **26 resource nodes** at three purities, gated behind milestones
- **14 hub milestones** and a **4-phase Ascent Elevator** that interleave to
  pace the tier unlocks
- **Resource sink** feeding a voucher catalog of 11 alternate recipes
- Byproducts genuinely clog: refine plastic without handling heavy oil residue
  and the refinery stops
- Offline production for up to four hours, local save, portable backup codes

## Files

- `index.html` - the whole game: markup, styles and simulation

## Verification

The build was checked by a headless harness covering reference integrity, the
unlock graph, progression reachability, 32 recipe rate assertions, generator
burn rates, and ten behavioural simulation tests (starvation, buffer backup,
brownout scaling, byproduct clogging, fuel exhaustion, offline/realtime drift,
and deadlock recovery). Two genuine deadlocks were found and fixed during that
pass: a coal-only grid that ran dry could never restart, and an under-watered
coal plant killed itself permanently. Both are resolved by hand-mineable ores
and a 5 MW hub reserve that never runs out.
