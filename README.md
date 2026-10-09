# CxChanger

CxChanger is a low-cost FDM multi-material and multi-color printing solution. It switches between preheated hotend modules loaded with different filaments to reduce tool-change waiting time. The design aims to remain inexpensive, mechanically simple, and reliable. Tool changes are driven by the printhead's XY motion, and the docking mechanism is designed to be adaptable to other hotends by changing the hotend mounting hardware.

![CxChanger printhead](https://github.com/user-attachments/assets/66c70b3b-5862-4e04-8ac4-0b782e6da9f5)

![CxChanger example](https://github.com/user-attachments/assets/bb0c0aa7-5b82-490f-add7-e86a71225352)

---

## Key Features

- **Automatic preheating:** Enable OrcaSlicer's Ooze Prevention feature and configure its preheat interval. OrcaSlicer can issue preheating commands before a material change. During the change, the old hotend is released and placed into its standby state while the next preheated hotend is picked up. If the next change is due soon, the next hotend can remain hot instead of cooling down.
- **Under-actuated tool-changing mechanism:** No dedicated powered lock actuator is required. The printhead's XY movement interacts with the dock to change hotends and open or close the filament idler.
- **Magnetic Maxwell kinematic coupling:** N52 magnets preload a Maxwell kinematic coupling. Three grooves made from locating pins are on the printhead side; three rounded locating pins on the removable hotend form the mating contact points. The three-point coupling constrains the tool's six degrees of freedom without over-constraint, helping it compensate for small manufacturing, assembly, wear, and thermal deviations.
- **Automatic nozzle-offset calibration:** A calibration fixture can measure each tool's XYZ offset relative to T0. Contact methods require a clean nozzle; non-contact eddy-current methods avoid that limitation. Recalibration is generally needed after initial assembly or after an event such as a tool crash, not before every print.
  - Contact calibration reference: https://github.com/Noisyfox/FoxChanger/tree/main/NozzleProbe
  - Non-contact eddy-current reference: https://oshwlab.com/cxg01/project_lbabffjk

## Hardware and Software Requirements

### Hardware

- **Printer:** In principle, any printer with an XY-moving printhead may be adaptable, including Voron 2.4, Voron Trident, and similar machines. This project provides printhead and basic dock designs; the dock mounts must be adapted to the frame and motion envelope of the target printer.
- **Controller:** Each removable hotend has a wired heater and thermistor. Spare mainboard outputs may support a small number of additional hotends. More tools may require an expansion board to add heater and thermistor interfaces: https://oshwlab.com/rayzark/project_qxihkkhj

### Software

- **Firmware:** Tool changes are implemented with Klipper macros. Use the klipper-toolchanger calibration extension if you want to use the supplied automatic nozzle-offset calibration macros: https://github.com/viesturz/klipper-toolchanger
- **Slicer:** OrcaSlicer is the documented slicer. Other slicers may be possible but are not covered here.

## System Overview

### Main assemblies

- **Permanent printhead:** All moving printhead components except the removable hotend module.
- **Removable hotend module:** Hotend, PTFE filament tube, heater, thermistor, and wiring.
- **Dock:** Stores each parked hotend, seals its nozzle to reduce ooze, and provides cooling for the parked tool. An open-frame printer can use a side-mounted centrifugal blower to cool the full dock array; an enclosed printer can use one axial fan per dock.

## Bill of Materials

### Printed-part version

- 6 × cylindrical locating pins, Ø1.5 × 5 mm, used to form the three grooves of the Maxwell coupling.
- 3 × rounded locating pins, Ø3 × 6 mm, per removable hotend, forming the mating kinematic contact points.
- 1 × internally threaded rounded pin, Ø5 × 20 mm, per dock, acting on the extruder lever.
- Ø6 × 4 mm N52 magnets: 3 per removable hotend, 1 per dock, and 3 on the permanent printhead.
- M3 × 6 mm countersunk screws.
- M3 × 25 mm screws.
- M3 × 8 mm screws.
- M3 × 10 mm screws.
- M3 × 12 mm screws.
- M3 × 14 mm screws.
- M3 × 20 mm screws.
- M2.5 × 16 mm screws.
- M2 × 8 mm screws.
- M3 heat-set inserts, 4 mm OD × 4 mm long (confirm the supplier's dimensional convention).
- HGX extruder gear set.
- 4020 blower fan.
- 2510 axial fan.
- Omron microswitch.
- 10 × 2 mm (width × thickness) self-adhesive, heat-resistant silicone strip.

### CNC version

Use the version-matched BOM supplied with the project. Do not combine the CNC parts, dock geometry, or toolchanger macros from different revisions without checking their interfaces.

## Installation and Configuration

### printer.cfg

Keep the motion, MCU, stepper-driver, heater, probe, and safety configuration that is already proven on your printer. Adapt the example instead of copying its board-specific pins unchanged.

1. Include toolchange.cfg and calibration.cfg.
2. Define an additional heater/thermistor section for each removable hotend, following the existing extruder1 pattern. The single physical extruder drive is shared; additional hotends are heater/sensor definitions rather than independent filament-drive motors. Wire heater and sensor inputs according to the actual controller.
3. The documented probing method uses a fixed microswitch. The tools are parked and the nozzle seals rest against high-temperature silicone during preheating. During leveling, the removable hotends remain docked so the fixed switch is the lowest sensing point. This configuration is specific to the documented implementation; other probe/leveling arrangements require an explicit integration.
4. Add dock-cooling fan configuration. Associate the relevant hotend/heatbreak cooling fans with every hotend whose temperature should keep the fan active.
5. Update print-start G-code so every hotend used by the sliced model is heated and primed before printing.
6. Update print-end G-code to call UNTOOL and turn off all hotend heaters so the machine finishes with the tool parked.

### toolchange.cfg

1. Include the toolchange macros. After the configuration loads, the UI should expose T0, T1, T2, T3, and UNTOOL for a four-tool setup. **Do not execute a tool-change macro before coordinates and the full motion path have been validated.** Incorrect coordinates can cause a collision.
2. Mount the docks and measure their coordinates mechanically:
   - Manually attach a hotend to the printhead.
   - Move the printhead to the intended dock side and align the first dock so the tool's hook screw passes cleanly through the dock opening.
   - Remove the tool and home the printer.
   - Reattach the tool and slowly jog to the same mechanical engagement point.
   - Record the machine coordinates and enter them as the T0 dock coordinates.
   - Nominal dock spacing is 30 mm. Derive the other dock positions from the pitch, then individually verify and correct each dock.
3. Test the motion path at a slow speed first. Keep your hand near the emergency stop and watch for misalignment.
4. Only increase speed after repeated cold and hot pickup/dropoff cycles work reliably.

### calibration.cfg

1. Install the calibration extension: https://github.com/viesturz/klipper-toolchanger
2. Home XYZ, level the bed/gantry, and mount the calibration fixture securely.
3. Pick up T0 and position its nozzle about 1 mm above the fixture center. Record the coordinates and enter the fixture location in calibration.cfg.
4. Define a separate safe calibration position outside the dock array and outside the path of all parked tools.
5. Calibrate T0 first, then the remaining tools. Depending on the installed extension version, the commands are TOOL_LOCATE_SENSOR and TOOL_CALIBRATE_TOOL_OFFSET; follow the extension's installed command names and verify the saved results.
6. Validate tool offsets with a test print before production.

### OrcaSlicer

Configure the printer's Start G-code, tool-change G-code, number of extruders, and tool-change retraction for the number of hotends actually installed. The following is a five-slot source example; remove unused tool parameters for a smaller build and do not configure T4 unless the hardware, heaters, and macros have been added.

**Start G-code example:**

    PRINT_START BED=[first_layer_bed_temperature] INITIAL_TOOL=[initial_tool] T0_TEMP={nozzle_temperature_initial_layer[0]} T0_USED={is_extruder_used[0]} T1_TEMP={nozzle_temperature_initial_layer[1]} T1_USED={is_extruder_used[1]} T2_TEMP={nozzle_temperature_initial_layer[2]} T2_USED={is_extruder_used[2]} T3_TEMP={nozzle_temperature_initial_layer[3]} T3_USED={is_extruder_used[3]} T4_TEMP={nozzle_temperature_initial_layer[4]} T4_USED={is_extruder_used[4]}

**Tool-change G-code example:**

    M104 S{nozzle_temperature[next_extruder]} T{next_extruder}
    T{next_extruder}

Enable the wipe tower and Ooze Prevention for initial validation. The original project suggests around 35 seconds as a starting preheat interval; tune it to the measured heat-up performance of your hotends. Coordinate slicer retraction with any pickup prime/retract configured in the macros to avoid stacking unintended extrusion moves.

## Build Notes and Critical Precautions

- Print structural parts in ABS using 4 walls and 60% infill, as specified by the project documentation.
- The HGX extruder spring requires more preload than a conventional arrangement because the compact lever geometry reduces mechanical advantage. Verify that the dock can still open the idler fully.
- For the documented TZ2.0 hotend arrangement, the project recommends removing the two heater-block retaining screws and using the side set screw for retention to reduce heat conduction. Verify this against the exact hotend revision before applying it.
- Install magnets to the depth and flushness shown in the matching CAD. Correct assembly should have the removable tool's rounded pins contacting the grooves on the printhead backing plate while the opposing magnets retain a small clearance. Check for rocking or backlash. If print accuracy degrades, inspect the locating pins and magnet retention; a loose magnet can migrate into contact with its mate and defeat the kinematic coupling.
- The tool's dock-hook screw should be a magnetic 12.9-grade steel socket-head screw. Ordinary 304 stainless steel may not be attracted properly by the dock magnet.
- The 30 mm dock pitch and sample coordinates are design references, not universal coordinates. Determine every dock position on the actual printer.

## Safety and Revision Notice

This repository includes example machine configuration. Heater pins, thermistor pins, probe pins, endstop logic, travel bounds, dock coordinates, and tool offsets are hardware- and frame-specific. Keep tool-change motion disabled until a complete manual dry-run and low-speed powered test pass. Never assume the software's saved tool state matches physical hardware after an E-stop or power loss.

This project is licensed under GPL-3.0. Preserve the original license and upstream attribution when redistributing modifications.
