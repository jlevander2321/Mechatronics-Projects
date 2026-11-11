# SET/RESET vs Seal-In (Power-Loss Comparison)

Two latch methods compared after a power loss. Built in AutomationDirect Productivity Suite 4.7.3 (P1000 CPU) and tested in its built-in simulator. This is the first exercise on the Productivity platform.

![Ladder online in the simulator](ladder-online.png)

## Tags
All internal Booleans (no I/O modules configured).

| Tag | Description | Retentive |
| --- | --- | --- |
| Start_PB | Momentary start | No |
| Stop_PB | Momentary stop | No |
| Motor_Run | Seal-in output | No |
| Latch_Set | Set input | No |
| Latch_Reset | Reset input | No |
| Motor_Latch | SET/RESET output | No (test 7), Yes (test 8) |

## Logic
| Rung | Logic |
| --- | --- |
| 1 / 1.1 | (Start_PB OR Motor_Run) AND NOT Stop_PB → OUT Motor_Run |
| 2 | Latch_Set → SET Motor_Latch |
| 3 | Latch_Reset → RST Motor_Latch |

The seal-in branch (rung 1.1) rejoins between Start_PB and Stop_PB, so Stop always breaks the seal. SET/RST write the bit only when their rung is true; the bit holds its value otherwise.

## Tests
Bits toggled with Ctrl+T on the contacts while online in Monitor.

| # | Action | Expected | Result |
| --- | --- | --- | --- |
| 1 | Start_PB on, then off | Motor_Run = 1, stays 1 | Pass |
| 2 | Stop_PB on, then off | Motor_Run = 0 | Pass |
| 3 | Latch_Set on, then off | Motor_Latch = 1, stays 1 | Pass |
| 4 | Latch_Reset on, then off | Motor_Latch = 0 | Pass |
| 5 | Stop_PB and Start_PB both on | Motor_Run = 0 (stop wins) | Pass |
| 6 | Latch_Set and Latch_Reset both on | Motor_Latch = 0 (RST is the later rung) | Pass |
| 7 | Both outputs on, Motor_Latch non-retentive; CPU stop → run | Motor_Run = 0, Motor_Latch = 0 | Pass |
| 8 | Both outputs on, Motor_Latch retentive; CPU stop → run | Motor_Run = 0, Motor_Latch = 1 | Pass |

**After test 8:** Motor_Run came back 0; the retentive Motor_Latch came back 1.

![Monitor after test 8](monitor-after-test8.png)

## Conclusion
The seal-in is the safer choice for a motor. After a power loss it comes back off and needs a deliberate Start press. A retentive SET/RESET latch can restart the motor on its own when power returns, which could injure someone working on the machine. Unexpected start-up is a hazard listed in ISO 12100, and the electrical-equipment standards (IEC 60204-1 / NFPA 79) address restart after a supply interruption (verify the exact clauses against the standards). The PLC's off state is not a substitute for lockout/tagout when working on the machine.

In a test 6 tie, the last rung to write the bit in the scan wins, so rung order sets priority.

## Notes
- Stop is an NC contact because the simulated stop is a momentary NO bit. On real hardware, wire stop NC and use a NO contact in logic.
- Stop → Run in the simulator was used as the power-loss stand-in.

## Files
- Productivity Suite project (`practice1.adpro`): to be added.
