---
name: start-sit
description: Resolve a head-to-head start/sit decision between two players for the current scoring period. Works for any fantasy sport.
---

Compare the two players the user names and give a single, decisive start/sit verdict.

Steps:

1. Pull each player's projection, opponent/schedule, and recent usage (snaps/targets, minutes,
   rotation role, etc.) from whatever fantasy data provider MCP is connected (Sleeper, ESPN, Yahoo,
   etc.). If none is connected, work from what the user provides.
2. Weigh projection first, then matchup/opponent strength, then floor vs. ceiling for the manager's
   situation (favorites want floor, underdogs want ceiling). For daily sports, factor how many games
   each plays in the period and whether they are confirmed active.
3. Give one clear answer: "Start X over Y." Then one sentence of why, and the single risk that would
   flip it.

Be decisive. The manager wants a call, not a hedge.
