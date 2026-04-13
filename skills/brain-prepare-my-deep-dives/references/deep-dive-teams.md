# Deep Dive Team Mapping

Configure this file to match the teams you run deep dives with. The skill reads this table at runtime.

| Calendar pattern (case-insensitive) | File slug | Linear teams |
|-------------------------------------|-----------|--------------|
| `TEAM1` | team1 | TEAM1 |
| `TEAM2` | team2 | TEAM2 |
| `DEVOPS` or `DEVOPS & IT` | devops | DEVOPS, IT |
| `DATA` | data | Data Engineering |
| `Tech` | tech | STAFF |
| `TEAM3` | team3 | TEAM3 |
| `TEAM4` | team4 | TEAM4 |
| `TEAM5` | team5 | TEAM5 |

If an event doesn't match any known mapping, log a warning and skip it.
