# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).
Releases up to 10.9 used a two-part version number; their entries are kept below as they were written, sorted
into Added, Changed, Removed, Fixed and Security. From 10.10.0 on, versions follow Semantic Versioning.

## [Unreleased]

### Added

- `AlphaESS Total Energy PV Inverter Consumption` (register 0x0090): the AC-coupled PV inverter's own standby draw.
- `AlphaESS System Mode` (register 0x0805).
- `AlphaESS Total Energy from PV Offset` helper and `AlphaESS Total Energy from PV Calibrated`: Total Energy from PV
  plus the offset, for an inverter whose lifetime counter was reset or replaced.
- `AlphaESS Current House Load Filtered` and `AlphaESS Current PV Production Filtered`: low-pass filtered copies of
  Current House Load and Current PV Production.
- `AlphaESS PV Inverter Energy` (0x08D0) and `AlphaESS PV System Total Energy` (0x08D2): the lifetime PV inverter
  energy and the system total PV energy.
- `AlphaESS Battery SOC Calibration` (0x012F) and `AlphaESS Battery Single Cut Error Code` (0x0130).
- `AlphaESS Inverter Work Mode Name`: the inverter work mode as text (Wait, Online, UPS, Bypass, Fault, ...).
- `AlphaESS Excess Export Inverter AC Limit Offset` helper (0 to 2000 W, default 0): Excess Export treats the
  inverter AC limit as this many W lower.
- Force Charging, Force Discharging, Force Import and Force Export Hold helpers and the `AlphaESS Cutoff SoC Hold
  Monitoring` automation: with its Hold helper off, a Force mode ends once the battery has idled for 10 s, which is
  taken as its cutoff SoC reached (only Force Export also checks the SoC: below its cutoff + 2). Force Charging Hold
  and Force Import Hold are on after every Home Assistant start, the other two off.
- Peak Shaving: the `AlphaESS Peak Shaving`, `AlphaESS Peak Shaving Pause` and `AlphaESS Peak Shaving Resume`
  automations, the Peak Shaving, Peak Shaving Pause, Peak Shaving Cutoff SoC and Peak Shaving Power helpers and the
  `AlphaESS Template Peak Shaving Power` number. At or above the Peak Shaving Cutoff SoC, or while PV production
  exceeds the house load, Peak Shaving pauses and the inverter runs its own mode; below the cutoff the battery holds
  at 0 W and discharges only by what the house load minus PV production exceeds the Peak Shaving Power, so the grid
  import stays at about that power. A new install starts at 4 % and 0 kW, which keeps it paused until the cutoff is
  set.

### Changed

- README: a new dashboard screenshot, a screenshot and time-window table for every power diagram view, an Energy
  Dashboard setup guide, and the recorder history (3 days) the Yesterday and 3 Day views need.
- `AlphaESS Total Energy from PV Meter` is rounded to 2 decimals, and the ApexCharts helper sensors are named
  `AlphaESS ...` (was `AlphaEss ...`); their entity ids do not change.
- The filtered house load and PV production now feed Force Export, Force Import, Excess Export and these
  sensors: Excess Power, "House Load - PV Production > Force Import Power" and the House Load and PV Production
  ApexCharts sensors. Each read falls back to the unfiltered sensor while the filtered one has no value.

### Fixed

- Changing the Force Import Power slider while Force Import runs no longer makes the battery discharge: the update
  now writes the same setpoint as Force Import itself (charge = import power + PV - house load). Changing the Force
  Export Power slider likewise writes the Force Export setpoint (discharge = house load + export power - PV)
  instead of the bare export power.
- Force Import no longer pauses while PV production still covers the gap: the pause and resume now compare house
  load minus PV production with the import power, and the pause also re-checks when PV production changes. The
  helper sensor is renamed to "AlphaESS House Load - PV Production > Force Import Power"
  (`sensor.alphaess_house_load_pv_production_force_import_power`).
- Total Energy Feed to Grid (PV) reads register 0x0092 instead of 0x0090: on AC-coupled PV inverters 0x0090 counts
  the inverter's standby draw and stayed near zero. Systems without an AC PV meter read 0 on both. With an
  AC-coupled PV inverter the sensor jumps once to the real total after the update; if it feeds the Energy
  dashboard, adjust that step under Developer Tools > Statistics.
- Current House Load no longer reads 0 during a grid outage: in UPS mode it falls back to PV production + battery
  power when the grid reads 0.
- Inverter values that read `unknown` or `unavailable` no longer break automations or template sensors: the Force
  Export, Force Import, Excess Export and helper sync automations skip the run until the source sensors are numbers
  again (Force Import Pause skips only its PV check and still pauses on the inverter work mode), and template sensors
  (Current House Load, Current PV Production, Total House Load and others) go unavailable instead of showing a
  partial value. Battery Full no longer goes unavailable when the battery status reads 0.
- Current House Load no longer drops to 0 for a moment when PV, battery and grid power update at different
  moments: it keeps its last value while their sum is 0 or below.
- Dispatch SoC shows 100% instead of 102% after a full-charge dispatch: the sensor is capped at 100.

## [10.10.0] - 2026-11-24

Dispatch corrections for [#1](https://github.com/ramonvanraaij/ha-alphaess-modbus/issues/1) and
[#2](https://github.com/ramonvanraaij/ha-alphaess-modbus/issues/2), and coexistence with an external dispatch
controller such as an energy supplier's virtual power plant (VPP) or cloud battery trading. When upgrading,
replace the whole `integration_alpha_ess.yaml` package: the dispatch automations now depend on the new VPP Mode
helper.

### Added

- VPP Mode switch `input_boolean.alphaess_helper_vpp_mode` ("AlphaESS VPP Mode") for setups where an external
  controller drives the dispatch registers `0x0880-0x088A`. Its state survives restarts. While it is on, Home
  Assistant makes no dispatch writes of its own:
  - the resets on Home Assistant start, on Modbus connection loss and on dispatch timer expiry skip the write;
  - Force Charging, Force Discharging, Force Export, Force Import, Manual Dispatch and Excess Export refuse to
    start: they switch back off and post a notification;
  - the slider updates (power, duration, cutoff SoC, dispatch mode, PV switch) and the Excess Export / Force
    Import pause logic write nothing;
  - only the Dispatch Reset Full button still writes the neutral block.
- `AlphaESS_VPP_Mode_On` automation: turning VPP Mode on releases every dispatch helper and timer, and clears a
  running Home Assistant dispatch once.
- The VPP Mode switch on the Charging and Dispatch cards of `alphaess_view.yaml`.

### Changed

- README: documented VPP Mode, how to recognise a dispatch that comes from outside Home Assistant, and the short
  battery/export transient a dispatch mode change can cause.

### Fixed

- The Dispatch SoC target now uses the vendor scale of 0.4 %/bit (SoC x 2.5), as measured on a SMILE-G3-T10;
  100% is still written as 255 so a full charge fills the battery. `AlphaESS_Dispatch_SoC` reads back with
  scale 0.4 (a raw 255 shows as 102%). This reverses the 0.392 change of 8.6 for targets below 100%; a SMILE-B3
  measures 0.392 %/bit.
- The alias of `AlphaESS_Update_Dispatch_Duration` duplicated "AlphaESS Update Force Charging Duration".
- README dispatch reference: the power value sits in the low word `0x0882` (`0x0881` is the high word and stays
  0), SoC is 0.4 %/bit, the PV Switch and Flow Direction values, which modes use the power parameter (Mode 5
  takes a power limit, Modes 4, 6, 7 and 19 ignore it), the OSW mode list, Scenario B and the dead man's switch
  description.

## [10.9] - 2026-07-16

Added battery cell-health registers and daily self-sufficiency/self-consumption calculations, cross-checked against the senalse/ha-alphaess-modbus fork; the two KPI formulas were verified live against the running SMILE-G3-T10 instance via the Home Assistant REST API.

### Added

- Added AlphaESS_Battery_Min_Cell_Voltage (0x0107), AlphaESS_Battery_Max_Cell_Voltage (0x010A) and AlphaESS_Battery_Capacity (0x0119); addresses, scales and units verified against the vendor register map.
- Added AlphaESS_Battery_Cell_Voltage_Delta template sensor for imbalance monitoring.
- Added AlphaESS_Self_Sufficiency_Today and AlphaESS_Self_Consumption_Today template sensors, built from the existing daily utility meters and AlphaESS_Today_s_House_Load.

### Changed

- Updated alphaess_view.yaml to display the new sensors in the Battery and Today sections.

### Fixed

- Fixed a double space in the AlphaESS Backup Power Inverter L2 sensor name (0x0416); unique_id, address, and entity_id are unaffected.

## [10.8] - 2026-05-10

Extensive refactor and stabilization for SMILE-G3-T10.

### Added

- Added commented-out registers for AUX (Dry Contact) and Generator Controls (0x2200-0x225D) for users with external appliances or auto-start generators.
- Added commented-out dashboard cards for AUX and Generator controls in alphaess_view.yaml.
- Added developer attribution to all YAML files.
- Added missing current and voltage sensors verified against live inverter:
  - AlphaESS_Current_Phase_A/B/C_Grid (0x0017-0x0019): grid current per phase
  - AlphaESS_Current_Phase_A/B/C_PV_Meter (0x0097-0x0099): PV meter current per phase
  - AlphaESS_Voltage_Inverter_L1/L2/L3 (0x0400-0x0402): inverter AC voltage per phase
  - AlphaESS_Current_Inverter_L1/L2/L3 (0x0403-0x0405): inverter AC current per phase
  - AlphaESS_Battery_Charge_Cutoff_Voltage (0x0113): battery charge cutoff voltage
  - AlphaESS_Battery_Discharge_Cutoff_Voltage (0x0114): battery discharge cutoff voltage

### Changed

- Restructured and modernized alphaess_view.yaml to use a standard 'sections' dashboard layout.
- Standardized utility_meter naming to 'today_s' and normalized entity IDs to lowercase.
- Standardized documentation filenames in docs/ to lowercase snake_case.
- Reorganized documentation: extracted historical change log to CHANGELOG.md, added a comprehensive README.md.
- Updated alphaess_view.yaml to display all new sensors in Grid, PV Output, and Battery sections.

### Removed

- Removed HHE MEC specific registers (0x072C-0x072F) which are unsupported and cause "Entity not found" errors on standard SMILE-G3 models.

### Security

- Secured sensitive data by extracting it to SECRETS.md encrypted via git-crypt.

## [10.7] - 2026-05-02

### Changed

- Adjusted Sensor Data Types

## [10.6] - 2026-04-29

### Added

- Added AlphaESS_Dispatch_PV_Switch, alphaess_helper_dispatch_pv_switch, AlphaESS_Update_Dispatch_PV_Switch, adjusted all Dispatch related automations

## [10.5] - 2026-04-13

### Added

- Added AlphaESS_Force_Import, AlphaESS_Force_Import_Pause, AlphaESS_Force_Import_Resume, AlphaESS_Update_Force_Import_Cutoff_SoC, AlphaESS_Update_Force_Import_Duration, AlphaESS_Update_Force_Import_Power and related helpers, modified affected automations accordingly

## [10.4] - 2026-04-05

### Changed

- Updated input_boolean.turn_off, timer.start, timer.cancel syntax

## [10.3] - 2026-03-28

### Added

- Added AlphaESS_EMS_Version_Low_Suffix, adjusted AlphaESS_EMS_Version_Normalised accordingly

## [10.2] - 2026-03-27

### Fixed

- Fixed AlphaESS_Today_s_House_Load, AlphaESS_Total_Energy_from_PV_Meter

## [10.1] - 2026-03-13

### Fixed

- Fixed AlphaESS_Current_House_Load (again) to avoid double figures when in UPS Mode

## [10.0] - 2026-03-10

### Added

- Added AlphaESS_Filter_Clipping and alphaess_helper_clipping

## [9.9] - 2026-02-14

### Added

- Added AlphaESS_System_Fault

## [9.8] - 2026-02-12

### Changed

- Adjusted template sensors, adding device_class and state_class

## [9.7] - 2026-01-24

### Added

- Added AlphaESS_Force_Export and related helpers, added AlphaESS_Update_Force_Export_Cutoff_SoC, AlphaESS_Update_Force_Export_Duration, AlphaESS_Update_Force_Export_Power, modified affected automations accordingly

## [9.6] - 2026-01-05

### Added

- Added AlphaESS_IP_Method, AlphaESS_Local_IP, AlphaESS_Subnet_Mask, AlphaESS_Gateway and their corresponding normalised templates

## [9.5] - 2026-01-05

### Changed

- Reduced scan_interval for AlphaESS_Power_Grid, AlphaESS_Power_Battery (5s to 1s)

## [9.4] - 2026-01-01

### Added

- Added warnings & faults sensors

## [9.3] - 2025-12-25

### Fixed

- Fixed bug in AlphaESS_Update_Force_Discharging_Power

## [9.2] - 2025-12-15

### Added

- Added sensors to improve Power Diagram performance

## [9.1] - 2025-12-07

### Added

- Added AlphaESS_Synchronise_Date_Time and related helper

## [9.0] - 2025-11-07

### Changed

- Adjusted AlphaESS_Excess_Export calculation to support hybrid setups, reduced scan_interval for AlphaESS_Active_Power_PV_Meter (5s to 1s)

## [8.9] - 2025-10-28

### Changed

- Modified AlphaESS_Current_House_Load to include inverter's power consumption, commented out initial values

## [8.8] - 2025-10-14

### Changed

- Simplified writing the Dispatch registers, improving performance and readability

## [8.7] - 2025-09-27

### Changed

- Modified Power Diagram, making "AlphaESS Energy Feed to Grid (Meter) (Derivative)" and AlphaESS_Total_Energy_Feed_to_Grid_Meter_with_Heartbeat obsolete

## [8.6] - 2025-09-21

### Changed

- Adjusted AlphaESS_Dispatch_SoC scale from 0.4 to 0.392 to compensate for stopping before reaching 100%, changed related automations accordingly

## [8.5] - 2025-09-15

### Fixed

- Fixed bug in AlphaESS_Excess_Export_Pause and AlphaESS_Excess_Export_Resume that prevented proper operation on new installations

## [8.4] - 2025-09-09

### Changed

- Adjusted alphaess_helper_inverter_ac_limit

## [8.3] - 2025-09-04

### Fixed

- Fixed bug in AlphaESS_Excess_Export_Pause

## [8.2] - 2025-08-25

### Added

- Added AlphaESS_Power_Inverter_L1/L2/L3, AlphaESS_Backup_Power_Inverter_L1/L2/L3

## [8.1] - 2025-08-21

### Changed

- Replaced all Dispatch Mode related automations and helpers with one AlphaESS_Dispatch automation

## [8.0] - 2025-08-14

### Changed

- Reworked all Dispatch Mode related automations and helpers

## [7.9] - 2025-08-11

### Added

- Added AlphaESS_No_Battery_Charge and related helper, modified AlphaESS_Dispatch_Reset accordingly, modified AlphaESS_Current_House_Load to avoid double figures when in UPS Mode

## [7.8] - 2025-08-10

### Changed

- Modified AlphaESS_Current_House_Load to filter out negative values

## [7.7] - 2025-08-04

### Added

- Added AlphaESS_Template_Force_Charging_Power, AlphaESS_Template_Force_Disharging_Power to enable dynamic range adjustment for AlphaESS_Force_Charging_Power and AlphaESS_Force_Discharging_Power

## [7.6] - 2025-07-24

### Added

- Added AlphaESS_OvP10, AlphaESS_OvP10_Time

## [7.5] - 2025-07-18

### Added

- Added AlphaESS_Battery_Full

## [7.4] - 2025-07-16

### Added

- Added AlphaESS_Force_Discharging and related helpers, renamed AlphaESS_Charging_Now to AlphaESS_Force_Charging, renamed related helpers, added AlphaESS_Force_Charging_Power, AlphaESS_Force_Charging_Duration, modified AlphaESS_Dispatch_Reset accordingly

## [7.3] - 2025-07-05

### Added

- Added AlphaESS_Battery_Status, AlphaESS_Battery_Current

## [7.2] - 2025-06-12

### Changed

- Adjusted alphaess_helper_inverter_ac_limit

## [7.1] - 2025-06-10

### Changed

- Adjusted AlphaESS_OvP_L2 and AlphaESS_OvP_L2_Time, added AlphaESS_OvP_L3, AlphaESS_OvP_L3_Time, AlphaESS_UvP_L3, AlphaESS_UvP_L3_Time, AlphaESS_OfP_L3, AlphaESS_OfP_L3_Time, AlphaESS_UfP_L3, AlphaESS_UfP_L3_Time

## [7.0] - 2025-05-17

### Added

- Added alphaess_helper_inverter_ac_limit and adjusted AlphaESS_Excess_Export accordingly

## [6.9] - 2025-05-07

### Changed

- Rounded AlphaESS_Total_House_Load down to two decimal points

## [6.8] - 2025-05-05

### Changed

- Adjusted AlphaESS_Excess_Export sensitivity

## [6.7] - 2025-04-15

### Added

- Added AlphaESS_OfP_L1, AlphaESS_OfP_L1_Time, AlphaESS_OfP_L2, AlphaESS_OfP_L2_Time, AlphaESS_UfP_L1, AlphaESS_UfP_L1_Time, AlphaESS_UfP_L2, AlphaESS_UfP_L2_Time

## [6.6] - 2025-04-14

### Added

- Added Grid Safety registers: AlphaESS_Grid_Regulation (Details see Note 8 in AlphaESS-ModbusRegisterParameterList.pdf), AlphaESS_OvP_L1, AlphaESS_OvP_L1_Time, AlphaESS_OvP_L2, AlphaESS_OvP_L2_Time, AlphaESS_UvP_L1, AlphaESS_UvP_L1_Time, AlphaESS_UvP_L1, AlphaESS_UvP_L2_Time

## [6.5] - 2025-04-04

### Changed

- Adjusted AlphaESS_Excess_Export_Pause sensibility, added AlphaESS_Excess_Export_Resume condition

## [6.4] - 2025-04-03

### Changed

- Increased AlphaESS_Excess_Export_Resume wait time after fault to 10 minutes

## [6.3] - 2025-03-28

### Changed

- Changed AlphaESS_Excess_Export_Resume trigger from "state" to "time_pattern", improved overall AlphaESS_Excess_Export_Pause/AlphaESS_Excess_Export_Resume logic

## [6.2] - 2025-03-24

### Added

- Added Inverter Work Mode handling to both AlphaESS_Excess_Export_Pause and AlphaESS_Excess_Export_Resume

## [6.1] - 2025-03-08

### Changed

- Reduced scan_interval for AlphaESS_PV1_Power, AlphaESS_PV2_Power, AlphaESS_PV3_Power, AlphaESS_PV3_Power (5s to 1s) and AlphaESS_Voltage_Phase_A_Grid, AlphaESS_Voltage_Phase_B_Grid, AlphaESS_Voltage_Phase_C_Grid (60s to 5s)

## [6.0] - 2025-03-05

### Added

- Added AlphaESS_Clipping, reworked AlphaESS_Excess_Export

## [5.9] - 2025-02-28

### Changed

- Changed trigger for AlphaESS_Excess_Export_Pause and AlphaESS_Excess_Export_Resume

## [5.8] - 2025-02-27

### Added

- Added AlphaESS_Update_Charging_Discharging_Control

## [5.7] - 2025-02-20

### Added

- Added AlphaESS_PV3_Voltage, AlphaESS_PV3_Current, AlphaESS_PV4_Voltage, AlphaESS_PV4_Current

## [5.6] - 2024-11-28

### Added

- Added AlphaESS_Modbus_Baud_Rate, moved from Dec addresses to Hex, removed holding, precision and scale when default

## [5.5] - 2024-11-18

### Fixed

- Fixed broken utility meters

## [5.4] - 2024-11-14

### Fixed

- Fixed broken entities by re-adding "AlphaESS" prefixes, removed "AlphaESS" prefixes in alphaess_view.yaml instead

## [5.3] - 2024-11-12

### Fixed

- Fixed AlphaESS_Total_House_Load to avoid spikes during start-up, adjusted syntax for automations

## [5.2] - 2024-11-11

### Removed

- Removed obsolete "AlphaESS" prefixes

## [5.1] - 2024-11-10

### Added

- Added total and today sensor/utility meter for AlphaESS_Active_Power_PV_Meter

## [5.0] - 2024-11-09

### Added

- Added AlphaESS_Throttle_Charge, alphaess_helper_throttle_charge, AlphaESS_Load_Following, alphaess_helper_load_following and AlphaESS_Active_Power_PV_Meter

## [4.9] - 2024-11-02

### Added

- Added AlphaESS_System_Time_YYMM, AlphaESS_System_Time_DDHH, AlphaESS_System_Time_MMSS sensors and AlphaESS_System_Date, AlphaESS_System_Time templates, changed some templates to suppress errors in home-assistant.log during shutdown

## [4.8] - 2024-10-25

### Added

- Added AlphaESS_BMS_Version_Normalised and AlphaESS_EMS_Version_Normalised

## [4.7] - 2024-10-24

### Added

- Added AlphaESS_SoH_Battery

## [4.6] - 2024-10-22

### Added

- Added AlphaESS_PV_Capacity_Storage, AlphaESS_PV_Capacity_of_Grid_Inverter and AlphaESS_Backup_Power_Inverter, renamed AlphaESS_BMU_Version to AlphaESS_BMS_Version

## [4.5] - 2024-10-21

### Added

- Added AlphaESS_Charging_Cutoff_SoC and AlphaESS_Update_Charging_Cutoff_SoC_Slider automation

## [4.4] - 2024-10-20

### Added

- Added AlphaESS_Discharging_Periods and all dependencies, updated complementing alphaess_view.yaml

## [4.3] - 2024-10-19

### Changed

- Changed the modbus.write_register syntax to support Modbus RTU > TCP converters

## [4.2] - 2024-10-17

### Added

- Added AlphaESS_Charging_Periods and AlphaESS_Update_Charging_Periods, added AlphaESS_Charging_Now Timer, deleted AlphaESS_Charging_Default, limited alphaess_helper_discharging_cutoff_soc to 4%-100%, replaced AlphaESS_Today_s_House_Load

## [4.1] - 2024-10-12

### Added

- Added normalised templates AlphaESS_Charging_Period_1 and AlphaESS_Charging_Period_2

## [4.0] - 2024-10-09

### Fixed

- Fixed AlphaESS_Total_Energy_Feed_to_Grid_Meter_with_Heartbeat to eliminate template error in home-assistant.log during start-up

## [3.9] - 2024-10-08

### Changed

- Changed AlphaESS_Charging_Now to use Dispatch registers instead of Time Period Control registers to reduce unnecessary flash memory writes

## [3.8] - 2024-10-07

### Added

- Added alternative AlphaESS_Current_House_Load calculation to replicate the "Real-time Power Graph" of App/WebUI, added alternative AlphaESS_Total_House_Load calculation

## [3.7] - 2024-10-03

### Changed

- Adjusted syntax for automations following HA 2024.10 update

## [3.6] - 2024-09-23

### Added

- Added AlphaESS_Update_Discharging_Cutoff_SoC_Slider and AlphaESS_Update_Max_Feed_To_Grid_Slider automation

## [3.5] - 2024-09-22

### Changed

- Tweaked AlphaESS_Excess_Export_Pause and AlphaESS_Excess_Export_Resume sensitivity

## [3.4] - 2024-09-09

### Changed

- Changed AlphaESS_Excess_Export calculation for two different scenarios, PV Output > House Load and PV Output > Inverter Limit

## [3.3] - 2024-09-09

### Changed

- Changed AlphaESS_Excess_Export_Pause threshold from "above: 0" to "above: 50", changed AlphaESS_Excess_Export_Resume trigger from AlphaESS_Power_Grid to AlphaESS_Excess_Power and changed from "below: 0" to "above: 50", both to reduce sensitivity

## [3.2] - 2024-09-03

### Changed

- Changed AlphaESS_Excess_Power, introducing a dead man's switch by reducing Dispatch Time from 10h to 5min and changed AlphaESS_Dispatch_Reset to trigger at startup and connection loss

## [3.1] - 2024-09-01

### Changed

- Changed AlphaESS_Excess_Power to suppress negative values, added AlphaESS_Excess_Export_Pause and AlphaESS_Excess_Export_Resume and logic to pause/resume the export to serve house loads

## [3.0] - 2024-08-31

### Changed

- Renamed AlphaESS_Dispatch_Initiate to AlphaESS_Force_Discharge and alphaess_helper_dispatch_initiate to alphaess_helper_force_discharge

## [2.9] - 2024-08-31

### Added

- Added AlphaESS_Excess_Export automation, AlphaESS_Helper_Excess_Export helper switch, AlphaESS_Excess_Power and changed AlphaESS_Dispatch_Reset to react on AlphaESS_Helper_Excess_Export

## [2.8] - 2024-08-28

### Added

- Added AlphaESS_BMU_Version, AlphaESS_LMU_Version, AlphaESS_ISO_Version, AlphaESS_Inverter_Version AlphaESS_Inverter_ARM_Version, AlphaESS_EMS_Version and tidied up some sensor measurement units

## [2.7] - 2024-08-13

### Changed

- Updated author information and documentation link

## [2.6] - 2024-08-12

### Added

- Added AlphaESS_Inverter_Work_Mode (Details see Note 5 in AlphaESS-ModbusRegisterParameterList.pdf)

## [2.5] - 2024-05-24

### Changed

- Replaced "Alpha ESS*" and "Alpha_ESS*" with "AlphaESS*", changed "Total Energy Charge Battery from Grid" to "AlphaESS Total Energy Charge Battery from Grid"

## [2.4] - 2024-05-13

### Added

- Added additional header information

## [2.3] - 2024-03-07

### Fixed

- Fixed "Charging Now" delay

## [2.2] - 2024-03-05

### Changed

- Changed "Charging Default" to charge from 2:00-6:00 and extended "Dispatch" to 2.5h

## [2.1] - 2024-03-01

### Changed

- Changed Dispatch SoC to 20%

## [2.0] - 2024-02-27

### Added

- Added 6 Utility Meters for Today's Calculations, renamed some _Current and some _Total entities and fixed calculations based on these

## [1.9] - 2024-02-23

### Added

- Added "AlphaESS Energy Feed to Grid (Meter) (Derivative)" for Power Diagram support

## [1.8] - 2024-02-23

### Added

- Added AlphaESS_Total_Energy_Feed_to_Grid_Meter_with_Heartbeat to fix Derivative integration

## [1.7] - 2024-02-21

### Changed

- Renamed AlphaESS_Total_Energy_Feed_to_Grid (added _Meter), AlphaESS_Total_Energy_Consumption_from_Grid (added _Meter), added AlphaESS_Total_Energy_Feed_to_Grid_PV and adjusted AlphaESS_Total_House_Load

## [1.6] - 2024-02-03

### Changed

- Renamed AlphaESS_Time_Period_Control to AlphaESS_Charging_Time_Period_Control

## [1.5] - 2024-01-26

### Added

- Added day rollover logic to "Charging Now" automation

## [1.4] - 2024-01-25

### Added

- Added "Charging Now" automation and cleaned up helpers/automations names

## [1.3] - 2024-01-21

### Added

- Added "Charging" sensors and "Charging" automation

## [1.2] - 2024-01-17

### Changed

- Adjusted "Dispatch" values

## [1.1] - 2024-01-07

### Added

- Added grid power and voltage for 3 phases

## [1.0] - 2024-01-05

### Changed

- Replaced hexadecimal values in "Dispatch" automations with their decimal values

## [0.9] - 2024-01-04

### Changed

- Modified "Dispatch" automations and ran first successful "Dispatch" sequence

## [0.8] - 2024-01-02

### Added

- Added first "Dispatch" automations and started testing

## [0.7] - 2023-12-29

### Changed

- Found additional "Dispatch" details here: https://forums.whirlpool.net.au/thread/9xv6wzmm

## [0.6] - 2023-12-29

### Changed

- Tidied up several senor names and precision values

## [0.5] - 2023-12-14

### Changed

- Found "Dispatch" sequence details here:
  - https://www.storion4you.de/thread/831-modbus-befehl-senden-speicherladung-über-netz-aktivieren/

## [0.4] - 2023-12-11

### Fixed

- Fixed AlphaESS_Total_House_Load and AlphaESS_Current_House_Load calculations

## [0.3] - 2023-12-08

### Changed

- Reviewed https://github.com/umrath/homeassistant_alphaess_modbus_tcp/tree/dispatch but the "Dispatch" implementation was incomplete

## [0.2] - 2023-12-08

### Changed

- Updated with parts from https://github.com/umrath/homeassistant_alphaess_modbus_tcp/tree/target_soc

## [0.1] - 2023-12-05

### Changed

- Took initial copy from https://github.com/snitzelweck92/homeassistant_alphaess_modbus_tcp

[Unreleased]: https://github.com/ramonvanraaij/ha-alphaess-modbus/compare/v10.10.0...HEAD
[10.10.0]: https://github.com/ramonvanraaij/ha-alphaess-modbus/compare/v10.9...v10.10.0
[10.9]: https://github.com/ramonvanraaij/ha-alphaess-modbus/compare/v10.8...v10.9
[10.8]: https://github.com/ramonvanraaij/ha-alphaess-modbus/releases/tag/v10.8
