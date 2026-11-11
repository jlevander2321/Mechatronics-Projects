# Practice Log

| Date | CAD part | Code | Track rep | Note |
|---|---|---|---|---|
| Nov 8 | — | [Seal-in](practice/plc/week01/seal-in/): 4/4 tests pass; [Interlock](practice/plc/week01/interlock/): 5/5 tests pass | Setup: QElectroTech installed; Python 3.13 + pymodbus, matplotlib; [Multisim](practice/multisim/week01/): series RLC (Boylestad 15.29) and parallel RLC (16.13) matched hand calcs within 0.03 % | Stop wins on seal-in; on the interlock, the top rung wins when both starts are pressed in the same scan |
| Nov 10 | — | [SET/RESET vs seal-in](practice/plc/week01/set-reset-vs-seal-in/): 8/8 tests pass; first Productivity Suite 4.7.3 project, run in its simulator | Setup: Productivity Suite 4.7.3 installed | Seal-in comes back off after a power loss; retentive SET/RESET comes back on, so seal-in is safer for a motor |
