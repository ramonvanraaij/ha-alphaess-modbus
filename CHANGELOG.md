# Change History

```text
  0.1     2023-12-05      Took initial copy from https://github.com/snitzelweck92/homeassistant_alphaess_modbus_tcp
  0.2     2023-12-08      Updated with parts from https://github.com/umrath/homeassistant_alphaess_modbus_tcp/tree/target_soc
  0.3     2023-12-08      Reviewed https://github.com/umrath/homeassistant_alphaess_modbus_tcp/tree/dispatch
                          but the "Dispatch" implementation was incomplete
  0.4     2023-12-11      Fixed AlphaESS_Total_House_Load and AlphaESS_Current_House_Load calculations
  0.5     2023-12-14      Found "Dispatch" sequence details here:
                          https://www.storion4you.de/thread/831-modbus-befehl-senden-speicherladung-über-netz-aktivieren/
  0.6     2023-12-29      Tidied up several senor names and precision values
  0.7     2023-12-29      Found additional "Dispatch" details here: https://forums.whirlpool.net.au/thread/9xv6wzmm
  0.8     2024-01-02      Added first "Dispatch" automations and started testing
  0.9     2024-01-04      Modified "Dispatch" automations and ran first successful "Dispatch" sequence
  1.0     2024-01-05      Replaced hexadecimal values in "Dispatch" automations with their decimal values
  1.1     2024-01-07      Added grid power and voltage for 3 phases
  1.2     2024-01-17      Adjusted "Dispatch" values
  1.3     2024-01-21      Added "Charging" sensors and "Charging" automation
  1.4     2024-01-25      Added "Charging Now" automation and cleaned up helpers/automations names
  1.5     2024-01-26      Added day rollover logic to "Charging Now" automation
  1.6     2024-02-03      Renamed AlphaESS_Time_Period_Control to AlphaESS_Charging_Time_Period_Control
  1.7     2024-02-21      Renamed AlphaESS_Total_Energy_Feed_to_Grid (added _Meter),
                          AlphaESS_Total_Energy_Consumption_from_Grid (added _Meter), added
                          AlphaESS_Total_Energy_Feed_to_Grid_PV and adjusted AlphaESS_Total_House_Load
  1.8     2024-02-23      Added AlphaESS_Total_Energy_Feed_to_Grid_Meter_with_Heartbeat to fix Derivative integration
  1.9     2024-02-23      Added "AlphaESS Energy Feed to Grid (Meter) (Derivative)" for Power Diagram support
  2.0     2024-02-27      Added 6 Utility Meters for Today's Calculations, renamed some _Current and some _Total entities
                          and fixed calculations based on these
  2.1     2024-03-01      Changed Dispatch SoC to 20%
  2.2     2024-03-05      Changed "Charging Default" to charge from 2:00-6:00 and extended "Dispatch" to 2.5h
  2.3     2024-03-07      Fixed "Charging Now" delay
  2.4     2024-05-13      Added additional header information
  2.5     2024-05-24      Replaced "Alpha ESS*" and "Alpha_ESS*" with "AlphaESS*", changed
                          "Total Energy Charge Battery from Grid" to "AlphaESS Total Energy Charge Battery from Grid"
  2.6     2024-08-12      Added AlphaESS_Inverter_Work_Mode (Details see Note 5 in AlphaESS-ModbusRegisterParameterList.pdf)
  2.7     2024-08-13      Updated author information and documentation link
  2.8     2024-08-28      Added AlphaESS_BMU_Version, AlphaESS_LMU_Version, AlphaESS_ISO_Version, AlphaESS_Inverter_Version
                          AlphaESS_Inverter_ARM_Version, AlphaESS_EMS_Version and tidied up some sensor measurement units
  2.9     2024-08-31      Added AlphaESS_Excess_Export automation, AlphaESS_Helper_Excess_Export helper switch,
                          AlphaESS_Excess_Power and changed AlphaESS_Dispatch_Reset to react on AlphaESS_Helper_Excess_Export
  3.0     2024-08-31      Renamed AlphaESS_Dispatch_Initiate to AlphaESS_Force_Discharge and
                          alphaess_helper_dispatch_initiate to alphaess_helper_force_discharge
  3.1     2024-09-01      Changed AlphaESS_Excess_Power to suppress negative values, added AlphaESS_Excess_Export_Pause and
                          AlphaESS_Excess_Export_Resume and logic to pause/resume the export to serve house loads
  3.2     2024-09-03      Changed AlphaESS_Excess_Power, introducing a dead man's switch by reducing Dispatch Time from 10h
                          to 5min and changed AlphaESS_Dispatch_Reset to trigger at startup and connection loss
  3.3     2024-09-09      Changed AlphaESS_Excess_Export_Pause threshold from "above: 0" to "above: 50",
                          changed AlphaESS_Excess_Export_Resume trigger from
                          AlphaESS_Power_Grid to AlphaESS_Excess_Power and changed from "below: 0" to "above: 50",
                          both to reduce sensitivity
  3.4     2024-09-09      Changed AlphaESS_Excess_Export calculation for two different scenarios, PV Output > House Load and
                          PV Output > Inverter Limit
  3.5     2024-09-22      Tweaked AlphaESS_Excess_Export_Pause and AlphaESS_Excess_Export_Resume sensitivity
  3.6     2024-09-23      Added AlphaESS_Update_Discharging_Cutoff_SoC_Slider and AlphaESS_Update_Max_Feed_To_Grid_Slider
                          automation
  3.7     2024-10-03      Adjusted syntax for automations following HA 2024.10 update
  3.8     2024-10-07      Added alternative AlphaESS_Current_House_Load calculation to replicate the "Real-time Power Graph"
                          of App/WebUI, added alternative AlphaESS_Total_House_Load calculation
  3.9     2024-10-08      Changed AlphaESS_Charging_Now to use Dispatch registers instead of Time Period Control registers to
                          reduce unnecessary flash memory writes
  4.0     2024-10-09      Fixed AlphaESS_Total_Energy_Feed_to_Grid_Meter_with_Heartbeat to eliminate template error in
                          home-assistant.log during start-up
  4.1     2024-10-12      Added normalised templates AlphaESS_Charging_Period_1 and AlphaESS_Charging_Period_2
  4.2     2024-10-17      Added AlphaESS_Charging_Periods and AlphaESS_Update_Charging_Periods, added AlphaESS_Charging_Now
                          Timer, deleted AlphaESS_Charging_Default, limited alphaess_helper_discharging_cutoff_soc to 4%-100%,
                          replaced AlphaESS_Today_s_House_Load
  4.3     2024-10-19      Changed the modbus.write_register syntax to support Modbus RTU > TCP converters
  4.4     2024-10-20      Added AlphaESS_Discharging_Periods and all dependencies, updated complementing alphaess_view.yaml
  4.5     2024-10-21      Added AlphaESS_Charging_Cutoff_SoC and AlphaESS_Update_Charging_Cutoff_SoC_Slider automation
  4.6     2024-10-22      Added AlphaESS_PV_Capacity_Storage, AlphaESS_PV_Capacity_of_Grid_Inverter and
                          AlphaESS_Backup_Power_Inverter, renamed AlphaESS_BMU_Version to AlphaESS_BMS_Version
  4.7     2024-10-24      Added AlphaESS_SoH_Battery
  4.8     2024-10-25      Added AlphaESS_BMS_Version_Normalised and AlphaESS_EMS_Version_Normalised
  4.9     2024-11-02      Added AlphaESS_System_Time_YYMM, AlphaESS_System_Time_DDHH, AlphaESS_System_Time_MMSS sensors and
                          AlphaESS_System_Date, AlphaESS_System_Time templates, changed some templates to suppress errors in
                          home-assistant.log during shutdown
  5.0     2024-11-09      Added AlphaESS_Throttle_Charge, alphaess_helper_throttle_charge, AlphaESS_Load_Following,
                          alphaess_helper_load_following and AlphaESS_Active_Power_PV_Meter
  5.1     2024-11-10      Added total and today sensor/utility meter for AlphaESS_Active_Power_PV_Meter
  5.2     2024-11-11      Removed obsolete "AlphaESS" prefixes
  5.3     2024-11-12      Fixed AlphaESS_Total_House_Load to avoid spikes during start-up, adjusted syntax for automations
  5.4     2024-11-14      Fixed broken entities by re-adding "AlphaESS" prefixes, removed "AlphaESS" prefixes in
                          alphaess_view.yaml instead
  5.5     2024-11-18      Fixed broken utility meters
  5.6     2024-11-28      Added AlphaESS_Modbus_Baud_Rate, moved from Dec addresses to Hex, removed holding, precision and
                          scale when default
  5.7     2025-02-20      Added AlphaESS_PV3_Voltage, AlphaESS_PV3_Current, AlphaESS_PV4_Voltage, AlphaESS_PV4_Current
  5.8     2025-02-27      Added AlphaESS_Update_Charging_Discharging_Control
  5.9     2025-02-28      Changed trigger for AlphaESS_Excess_Export_Pause and AlphaESS_Excess_Export_Resume
  6.0     2025-03-05      Added AlphaESS_Clipping, reworked AlphaESS_Excess_Export
  6.1     2025-03-08      Reduced scan_interval for AlphaESS_PV1_Power, AlphaESS_PV2_Power, AlphaESS_PV3_Power,
                          AlphaESS_PV3_Power (5s to 1s) and AlphaESS_Voltage_Phase_A_Grid, AlphaESS_Voltage_Phase_B_Grid,
                          AlphaESS_Voltage_Phase_C_Grid (60s to 5s)
  6.2     2025-03-24      Added Inverter Work Mode handling to both AlphaESS_Excess_Export_Pause and
                          AlphaESS_Excess_Export_Resume
  6.3     2025-03-28      Changed AlphaESS_Excess_Export_Resume trigger from "state" to "time_pattern", improved
                          overall AlphaESS_Excess_Export_Pause/AlphaESS_Excess_Export_Resume logic
  6.4     2025-04-03      Increased AlphaESS_Excess_Export_Resume wait time after fault to 10 minutes
  6.5     2025-04-04      Adjusted AlphaESS_Excess_Export_Pause sensibility, added AlphaESS_Excess_Export_Resume condition
  6.6     2025-04-14      Added Grid Safety registers: AlphaESS_Grid_Regulation (Details see Note 8 in
                          AlphaESS-ModbusRegisterParameterList.pdf), AlphaESS_OvP_L1, AlphaESS_OvP_L1_Time, AlphaESS_OvP_L2,
                          AlphaESS_OvP_L2_Time, AlphaESS_UvP_L1, AlphaESS_UvP_L1_Time, AlphaESS_UvP_L1, AlphaESS_UvP_L2_Time
  6.7     2025-04-15      Added AlphaESS_OfP_L1, AlphaESS_OfP_L1_Time, AlphaESS_OfP_L2, AlphaESS_OfP_L2_Time, AlphaESS_UfP_L1,
                          AlphaESS_UfP_L1_Time, AlphaESS_UfP_L2, AlphaESS_UfP_L2_Time
  6.8     2025-05-05      Adjusted AlphaESS_Excess_Export sensitivity
  6.9     2025-05-07      Rounded AlphaESS_Total_House_Load down to two decimal points
  7.0     2025-05-17      Added alphaess_helper_inverter_ac_limit and adjusted AlphaESS_Excess_Export accordingly
  7.1     2025-06-10      Adjusted AlphaESS_OvP_L2 and AlphaESS_OvP_L2_Time, added AlphaESS_OvP_L3, AlphaESS_OvP_L3_Time,
                          AlphaESS_UvP_L3, AlphaESS_UvP_L3_Time, AlphaESS_OfP_L3, AlphaESS_OfP_L3_Time, AlphaESS_UfP_L3,
                          AlphaESS_UfP_L3_Time
  7.2     2025-06-12      Adjusted alphaess_helper_inverter_ac_limit
  7.3     2025-07-05      Added AlphaESS_Battery_Status, AlphaESS_Battery_Current
  7.4     2025-07-16      Added AlphaESS_Force_Discharging and related helpers, renamed AlphaESS_Charging_Now to
                          AlphaESS_Force_Charging, renamed related helpers, added AlphaESS_Force_Charging_Power,
                          AlphaESS_Force_Charging_Duration, modified AlphaESS_Dispatch_Reset accordingly
  7.5     2025-07-18      Added AlphaESS_Battery_Full
  7.6     2025-07-24      Added AlphaESS_OvP10, AlphaESS_OvP10_Time
  7.7     2025-08-04      Added AlphaESS_Template_Force_Charging_Power, AlphaESS_Template_Force_Disharging_Power to enable
                          dynamic range adjustment for AlphaESS_Force_Charging_Power and AlphaESS_Force_Discharging_Power
  7.8     2025-08-10      Modified AlphaESS_Current_House_Load to filter out negative values
  7.9     2025-08-11      Added AlphaESS_No_Battery_Charge and related helper, modified AlphaESS_Dispatch_Reset accordingly,
                          modified AlphaESS_Current_House_Load to avoid double figures when in UPS Mode
  8.0     2025-08-14      Reworked all Dispatch Mode related automations and helpers
  8.1     2025-08-21      Replaced all Dispatch Mode related automations and helpers with one AlphaESS_Dispatch automation
  8.2     2025-08-25      Added AlphaESS_Power_Inverter_L1/L2/L3, AlphaESS_Backup_Power_Inverter_L1/L2/L3
  8.3     2025-09-04      Fixed bug in AlphaESS_Excess_Export_Pause
  8.4     2025-09-09      Adjusted alphaess_helper_inverter_ac_limit
  8.5     2025-09-15      Fixed bug in AlphaESS_Excess_Export_Pause and AlphaESS_Excess_Export_Resume that prevented proper
                          operation on new installations
  8.6     2025-09-21      Adjusted AlphaESS_Dispatch_SoC scale from 0.4 to 0.392 to compensate for stopping before reaching
                          100%, changed related automations accordingly
  8.7     2025-09-27      Modified Power Diagram, making "AlphaESS Energy Feed to Grid (Meter) (Derivative)" and
                          AlphaESS_Total_Energy_Feed_to_Grid_Meter_with_Heartbeat obsolete
  8.8     2025-10-14      Simplified writing the Dispatch registers, improving performance and readability
  8.9     2025-10-28      Modified AlphaESS_Current_House_Load to include inverter's power consumption, commented out initial
                          values
  9.0     2025-11-07      Adjusted AlphaESS_Excess_Export calculation to support hybrid setups, reduced scan_interval for
                          AlphaESS_Active_Power_PV_Meter (5s to 1s)
  9.1     2025-12-07      Added AlphaESS_Synchronise_Date_Time and related helper
  9.2     2025-12-15      Added sensors to improve Power Diagram performance
  9.3     2025-12-25      Fixed bug in AlphaESS_Update_Force_Discharging_Power
  9.4     2026-01-01      Added warnings & faults sensors
  9.5     2026-01-05      Reduced scan_interval for AlphaESS_Power_Grid, AlphaESS_Power_Battery (5s to 1s)
  9.6     2026-01-05      Added AlphaESS_IP_Method, AlphaESS_Local_IP, AlphaESS_Subnet_Mask, AlphaESS_Gateway and their
                          corresponding normalised templates
  9.7     2026-01-24      Added AlphaESS_Force_Export and related helpers, added AlphaESS_Update_Force_Export_Cutoff_SoC,
                          AlphaESS_Update_Force_Export_Duration, AlphaESS_Update_Force_Export_Power, modified affected
                          automations accordingly
  9.8     2026-02-12      Adjusted template sensors, adding device_class and state_class
  9.9     2026-02-14      Added AlphaESS_System_Fault
 10.0     2026-03-10      Added AlphaESS_Filter_Clipping and alphaess_helper_clipping
 10.1     2026-03-13      Fixed AlphaESS_Current_House_Load (again) to avoid double figures when in UPS Mode
 10.2     2026-03-27      Fixed AlphaESS_Today_s_House_Load, AlphaESS_Total_Energy_from_PV_Meter
 10.3     2026-03-28      Added AlphaESS_EMS_Version_Low_Suffix, adjusted AlphaESS_EMS_Version_Normalised accordingly
 10.4     2026-04-05      Updated input_boolean.turn_off, timer.start, timer.cancel syntax
 10.5     2026-04-13      Added AlphaESS_Force_Import, AlphaESS_Force_Import_Pause, AlphaESS_Force_Import_Resume,
                          AlphaESS_Update_Force_Import_Cutoff_SoC, AlphaESS_Update_Force_Import_Duration,
                          AlphaESS_Update_Force_Import_Power and related helpers, modified affected automations accordingly
 10.6     2026-04-29      Added AlphaESS_Dispatch_PV_Switch, alphaess_helper_dispatch_pv_switch,
                          AlphaESS_Update_Dispatch_PV_Switch, adjusted all Dispatch related automations
 10.7     2026-05-02      Adjusted Sensor Data Types
 10.8     2026-05-10      Extensive refactor and stabilization for SMILE-G3-T10.
                          - Restructured and modernized alphaess_view.yaml to use a standard 'sections' dashboard layout.
                          - Standardized utility_meter naming to 'today_s' and normalized entity IDs to lowercase.
                          - Removed HHE MEC specific registers (0x072C-0x072F) which are unsupported and cause "Entity not found" errors on standard SMILE-G3 models.
                          - Added commented-out registers for AUX (Dry Contact) and Generator Controls (0x2200-0x225D) for users with external appliances or auto-start generators.
                          - Added commented-out dashboard cards for AUX and Generator controls in alphaess_view.yaml.
                          - Standardized documentation filenames in docs/ to lowercase snake_case.
                          - Reorganized documentation: extracted historical change log to CHANGELOG.md, added a comprehensive README.md.
                          - Secured sensitive data by extracting it to SECRETS.md encrypted via git-crypt.
                          - Added developer attribution to all YAML files.
                          - Added missing current and voltage sensors verified against live inverter:
                            AlphaESS_Current_Phase_A/B/C_Grid (0x0017-0x0019): grid current per phase
                            AlphaESS_Current_Phase_A/B/C_PV_Meter (0x0097-0x0099): PV meter current per phase
                            AlphaESS_Voltage_Inverter_L1/L2/L3 (0x0400-0x0402): inverter AC voltage per phase
                            AlphaESS_Current_Inverter_L1/L2/L3 (0x0403-0x0405): inverter AC current per phase
                            AlphaESS_Battery_Charge_Cutoff_Voltage (0x0113): battery charge cutoff voltage
                            AlphaESS_Battery_Discharge_Cutoff_Voltage (0x0114): battery discharge cutoff voltage
                          - Updated alphaess_view.yaml to display all new sensors in Grid, PV Output, and Battery sections.
 10.9     2026-07-16      Added battery cell-health registers and daily self-sufficiency/self-consumption
                          calculations, cross-checked against the senalse/ha-alphaess-modbus fork; the two
                          KPI formulas were verified live against the running SMILE-G3-T10 instance via the
                          Home Assistant REST API.
                          - Added AlphaESS_Battery_Min_Cell_Voltage (0x0107), AlphaESS_Battery_Max_Cell_Voltage
                            (0x010A) and AlphaESS_Battery_Capacity (0x0119); addresses, scales and units
                            verified against the vendor register map.
                          - Added AlphaESS_Battery_Cell_Voltage_Delta template sensor for imbalance monitoring.
                          - Added AlphaESS_Self_Sufficiency_Today and AlphaESS_Self_Consumption_Today template
                            sensors, built from the existing daily utility meters and AlphaESS_Today_s_House_Load.
                          - Updated alphaess_view.yaml to display the new sensors in the Battery and Today sections.
                          - Fixed a double space in the AlphaESS Backup Power Inverter L2 sensor name (0x0416);
                            unique_id, address, and entity_id are unaffected.
```
