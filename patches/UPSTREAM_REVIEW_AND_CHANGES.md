# CxChanger review notes (2026-10-09)

Upstream sample defines four tools (T0-T3), nominal 30 mm dock pitch, sample coordinates T0 (232,-20), T1 (202,-20), T2 (172,-20), T3 (142,-20), and dock path offsets X=-3 mm, Y dodge=14 mm, safe Y=30 mm. These are author-machine values, not universal Voron coordinates.

Key proposed fixes:
- Add a fail-closed `motion_enabled: 0` commissioning gate.
- Check XYZ homed and tool index before any dock movement.
- Change `RESTORE_GCODE_STATE ... MOVE=0` to `MOVE=1` to physically return to saved XYZ, only after verifying the return path is collision-free.
- Reduce commissioning speeds/acceleration and leave prime/retract at zero until tuned with slicer.
- Do not blindly trust saved active tool state after restart/E-stop.
- Use `last_x_result`, `last_y_result`, `last_z_result` only if the installed calibration extension version exposes them.
- Upstream tool sensor comment mentions `expander:PA9` but active sample pin is `PC14`; verify actual board pin.
- Do not guess heater/thermistor/probe/CAN/fan pins for a Voron.

Do not mark production-ready until dock coordinates, negative Y limits, full return route, wiring, switch polarity and repeated hot/cold pickup cycles have been physically validated.
