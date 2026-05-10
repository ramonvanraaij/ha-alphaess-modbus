# Disclaimer

**USE AT YOUR OWN RISK. READ THIS BEFORE INSTALLING.**

This project is provided "as is", without warranty of any kind, express or implied. By using it you accept full and sole responsibility for any consequences.

## Risk of equipment damage

This integration **writes** to Modbus dispatch and configuration registers on your AlphaESS inverter. Incorrect, malformed, or out-of-range writes can:

- Cause the inverter to behave unpredictably or shut down.
- Trigger protection faults that require manual reset or service intervention.
- Wear out the inverter's internal flash memory through excessive write cycles, leading to permanent failure of the control board.
- Damage the connected battery, PV array, grid-tie hardware, or any load attached to the system.

You are solely responsible for verifying every register address, data type, scaling factor, and value range against the official AlphaESS Modbus Register Parameter List for **your specific inverter model and firmware revision** before enabling any automation that performs writes.

## Regulatory and warranty risks

- **Grid-tie compliance:** Feeding power back to the grid, force-charging from the grid, or operating outside utility-approved parameters may violate local electricity regulations, your grid connection agreement, or your net metering contract. Compliance is your responsibility and varies by country, region, and utility.
- **Manufacturer warranty:** Modifying inverter operating parameters via third-party Modbus tooling may void your AlphaESS warranty, your installer's service agreement, or any extended support contracts. Check before use.
- **Electrical safety:** Operating high-voltage DC and AC equipment carries risk of fire, electrocution, and property damage. Installation and configuration of grid-connected energy storage should be performed by a qualified electrician where required by law.

## No liability

The author accepts no liability whatsoever for any direct, indirect, incidental, consequential, or punitive damages, including but not limited to: hardware damage, data loss, lost generation revenue, regulatory fines, warranty voidance, personal injury, or property damage arising from the use, misuse, or inability to use this project.

## No affiliation

This project is an independent, unofficial, community effort. It is **not affiliated with, endorsed by, sponsored by, or supported by**:

- AlphaESS Co., Ltd. or any AlphaESS subsidiary or distributor
- Nabu Casa or the Home Assistant project
- Any inverter installer, electricity utility, or regulatory body
- Any of the upstream authors credited in [LICENSE.md](./LICENSE.md)

All trademarks are the property of their respective owners.
