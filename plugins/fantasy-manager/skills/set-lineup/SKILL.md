---
name: set-lineup
description: Set the optimal starting lineup for a fantasy football week, respecting roster slots and bye weeks. Works with any connected fantasy data provider.
---

When asked to set or optimize a lineup:

1. Get the roster, this week's matchups, and player projections. Use whatever fantasy data
   provider MCP is connected -- Sleeper, ESPN, Yahoo, etc. (e.g. the `sleeper` plugin exposes
   rosters, matchups, and a lineup analyzer). If no provider is connected, ask the manager to
   paste their roster.
   - For matchup context, use the bundled provider-independent MCPs: `sports-data` for the
     opponent/game, `web-fetch` to read an official injury report, and `reddit`
     (r/fantasyfootball, r/nfl) for late-breaking inactives.
2. Read the league's roster requirements: the starting slots (QB, RB, WR, TE, FLEX, K, DEF) and
   how many of each.
3. For each slot, rank eligible players by projected points, then assign the highest projection
   that has not already been used.
   - FLEX takes the best remaining RB/WR/TE after the dedicated slots are filled.
   - Never start a player on a bye week or one marked OUT.
4. Output the lineup as a table: slot, player, opponent, projection. Then list the bench.
5. Call out the closest decisions (projection gap < 2 points) so the manager can override on
   matchup or game-script feel.

Optimize for projected points, but surface the judgment calls rather than hiding them. Treat the
provider as a data source only -- the ranking and slotting logic here is provider-independent.
