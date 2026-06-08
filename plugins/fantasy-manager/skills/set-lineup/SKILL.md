---
name: set-lineup
description: Set the optimal starting lineup for a fantasy scoring period, respecting your league's roster slots and player availability. Works for any sport (football, basketball, baseball, hockey) and any connected fantasy data provider.
---

When asked to set or optimize a lineup:

1. Get the roster, the slate of games for this scoring period, and player projections. Use whatever
   fantasy data provider MCP is connected -- Sleeper, ESPN, Yahoo, etc. (e.g. the `sleeper` plugin
   exposes rosters, matchups, and a lineup analyzer). If no provider is connected, ask the manager
   to paste their roster, the starting slots, and the scoring period.
   - For matchup and availability context, use the bundled provider-independent MCPs: `sports-data`
     for opponents/schedule, `web-fetch` to read an official injury or lineup report, and `reddit`
     (the sport's fantasy sub, e.g. r/fantasyfootball, r/fantasybball) for late-breaking inactives.
2. Read the league's roster requirements -- the starting slots and how many of each. These vary by
   sport, so use whatever the league defines, for example:
   - Football: QB, RB, WR, TE, FLEX (RB/WR/TE), K, DEF.
   - Basketball: PG, SG, SF, PF, C, G, F, UTIL.
   - Baseball: C, 1B, 2B, 3B, SS, OF, UTIL, SP, RP.
   - Hockey: C, LW, RW, D, G.
3. For each slot, rank eligible players by projection, then assign the highest projection not already
   used. A FLEX/UTIL slot takes the best remaining player eligible for it.
   - Never start a player who is not playing this period: on a bye (football), on an off day or ruled
     OUT (basketball/hockey/baseball), or otherwise inactive.
   - For daily-lineup sports (basketball, hockey, baseball), favor players with more games in the
     period and confirm each is in their team's lineup that day.
4. Output the lineup as a table: slot, player, opponent/game, projection. Then list the bench.
5. Call out the closest decisions (small projection gaps) so the manager can override on matchup,
   role, or schedule feel.

Optimize for projected points/value, but surface the judgment calls rather than hiding them. Treat
the provider as a data source only -- the ranking and slotting logic here is sport- and
provider-independent.
