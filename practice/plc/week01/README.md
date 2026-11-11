# Week 1 PLC Exercises

Ladder logic exercises, week of Nov 9, 2026 (OpenPLC; Productivity Suite from the SET/RESET exercise on).

| Exercise | What it shows | Tests |
| --- | --- | --- |
| [Seal-in](seal-in/) | Start/stop latch with a holding contact; stop wins | 4/4 pass |
| [Two-motor interlock](interlock/) | Each motor's run bit blocks the other; top rung wins on a same-scan start | 5/5 pass |
| [SET/RESET vs seal-in](set-reset-vs-seal-in/) | Latch behavior after a power loss (Productivity Suite simulator) | 8/8 pass |
