# Compact-Power-Card

![GitHub Downloads (all assets, all releases)](https://img.shields.io/github/downloads/pacemaker82/compact-power-card/total?label=Total%20Downloads) ![GitHub Downloads (all assets, latest release)](https://img.shields.io/github/downloads/pacemaker82/compact-power-card/latest/total?label=Latest%20Version)

<img width="509" height="209" alt="Screenshot 2025-12-11 at 09 09 14" src="https://github.com/user-attachments/assets/b09648a2-8364-40b7-84fb-2475cc28d631" />

<img width="513" height="211" alt="Screenshot 2025-12-10 at 12 27 40" src="https://github.com/user-attachments/assets/14b4b3f4-6fa3-4aa6-b56c-87ccf567c58d" />

<img width="478" height="198" alt="Screenshot 2025-12-11 at 09 00 40" src="https://github.com/user-attachments/assets/c25e187b-ed8a-4973-aa86-eb7f367f29c3" />

Inspired by the excellent [power flow card plus](https://github.com/flixlix/power-flow-card-plus) - A compact power card for Home Assistant that supports a tighter user experience, and 8 sources of power from the home in a single card. In addition, the card can show 6 entity labels for whatever you want, colour and configure them how you need.

## Functionality

- Up to 8 power measurement entities for the home that help calculate the rest of home usage
- Up to 6 additional state labels to show related info, like battery %, grid voltage, PV energy or whatever you want.
- Thresholds can be set on entities to fade out / hide the entity below the threshold.
- Home power is calculated by default based on the grid/power/battery. Alternatively, use a home power sensor.
- Grid & Battery sensors expect +/- values for import/export or charge/discharge. These can be inverted if the default behaviour isn't what you want.
- Icons, colors and units can be customised.

## Installation

1. Goto HACS (if you dont have that installed, install HACS)
2. Add a custom repository
3. Add the URL to this repo: `https://github.com/pacemaker82/Compact-Power-Card` using the category `Dashboard` (used to be `Lovelace` pre HACS 2.0.0)
4. Go back to HACS and search for "compact power card" in the HACS store
5. Download and refresh
6. Goto dashboard, edit dashboard, select 'add card' button, and add the new custom Compact Power Card. Use the configuration below to setup.

## Card Configuration:
```yaml
type: custom:compact-power-card
threshold_mode: calculations
entities:
  pv:
    entity: sensor.solar_power
    color: var(--energy-solar-color)
    threshold: 50                   
    decimal_places: 1
    labels:                         
      - entity: sensor.solar_energy_today
        unit: kWh
        color: "#ffb300"             
  grid:
    entity: sensor.grid_power
    invert_state_values: false
    threshold: 25
    labels:
      - entity: sensor.grid_voltage
        icon: mdi:lightning-bolt 
        unit: V
  home:
    entity: sensor.home_power
    decimal_places: 1
  battery:
    entity: sensor.battery_power
    invert_state_values: false
    threshold: 25
    labels:
      - entity: sensor.battery_soc
        unit: "%"
  sources:
    subtract_from_home: true 
    list:
      - entity: sensor.ev_charger_power
        icon: mdi:car-electric
        threshold: 50
        color: "#FFFFFF"
      - entity: sensor.pool_pump_power
        icon: mdi:pool
        threshold: 50
      - entity: climate.garage
        attribute: temperature
        unit: "C"
        icon: mdi:pool
        threshold: 50
```

# Compact Power Card Settings (Quick Reference)

## Card-level

| Name           | Setting slug    | What it does                                                                                   |
| -------------- | --------------- | ---------------------------------------------------------------------------------------------- |
| Threshold mode | `threshold_mode`| Chooses whether sub-threshold values are zeroed in math (`calculations`) or only dimmed (`display_only`). |

In display_only mode:

- Thresholds determine the opacity of the icon and labels (faded when below threshold)
- Thresholds determine the animation of power flow lines (off when below threshold)
- If home entity isn't provided: Home calculation is based on the raw values from PV, Grid, Battery and auxiliary sources (unless `sources.subtract_from_home: false`)
- If home entity is provided: Home calculation is based on the home entity state, minus the auxiliary sources by default (see `sources.subtract_from_home`)

In calculation mode:

- Thresholds determine the opacity of the icon and labels (faded when below threshold)-
- Thresholds determine the animation of power flow lines (off when below threshold)
- If threshold isn't met, then the raw value is 0
- If home entity isn't provided: Home calculation is based on the raw values from PV, Grid, Battery and auxiliary sources (unless `sources.subtract_from_home: false`)
- If home entity is provided: Home calculation is based on the home entity state, minus the auxiliary sources by default (see `sources.subtract_from_home`)

## Entities
Common keys for `pv`, `grid`, `home`, `battery`. The following settings are possible:

| Name                | Setting slug            | What it does                                                                    |
| ------------------- | ----------------------- | ------------------------------------------------------------------------------- |
| Entity              | `entity`                | Sensor/entity id used for values.                                               |
| Color               | `color`                 | Overrides line/icon/text color for that entity.                                 |
| Threshold           | `threshold`             | Values below this are zeroed (per `threshold_mode`) and dimmed.                 |
| Decimals            | `decimal_places`        | Number of decimals to display (default 1).                                      |
| Unit override       | `unit` / `unit_of_measurement` | Display-only unit override; math always uses raw numeric values.         |
| Invert state values | `invert_state_values`   | Flip sign of grid/battery readings (e.g., import/export, charge/discharge).     |

```
entities:
  pv:
    entity: sensor.solar_power            
  grid:
    entity: sensor.grid_power
  home:
    entity: sensor.home_power
  battery:
    entity: sensor.battery_power
```

The card supports many combinations: PV/Grid/Home/Battery, PV/Grid/Home, Battery/Grid/Home, Grid/Home

Home-specific:
- If `home` is provided, the card uses its value (minus any aux sensor power usage); if omitted, home is inferred from pv/grid/battery power, minus any aux power sources. (see aux sources below for more)

## Aux sources

Aux sources are up to 8 power sources within your home that you want to show in the card. By default, their power values will be subtracted from the home power value. If you don't want that to happen, set `subtract_from_home` under `sources`.

| Name                    | Setting slug                      | What it does                                                                     |
| ----------------------- | --------------------------------- | -------------------------------------------------------------------------------- |
| Source entity           | `entity`                | Sensor/entity id for an auxiliary load.                                          |
| Source attribute        | `attribute`             | Read from an attribute instead of state.                                         |
| Source icon             | `icon`                  | Optional icon for the source badge.                                              |
| Source color            | `color`                 | Optional color override for that source.                                         |
| Source threshold        | `threshold`             | Dims/zeros source below threshold (per `threshold_mode`).                        |
| Source decimals         | `decimal_places`        | Decimal places for that source.                                                  |
| Source unit             | `unit` / `unit_of_measurement` | Display unit override for that source.                       |
| Subtract from home      | `subtract_from_home`      | If true (default), subtract summed aux usage from home value/calculation.        |

```
  sources:
    subtract_from_home: true 
    list:
      - entity: sensor.ev_charger_power
        icon: mdi:car-electric
        threshold: 50
        color: "#FFFFFF"
      - entity: sensor.pool_pump_power
        icon: mdi:pool
        threshold: 50
      - entity: climate.garage
        attribute: temperature
        unit: "C"
        icon: mdi:pool
        threshold: 50
```

## Labels (per pv/grid/battery)

Labels can be used to display "other" information - that can be more power stats, energy stats, weather, whatever you want. Note: these are just labels, they do not factor into the power diagram or calculations at all. 

| Name          | Setting slug                 | What it does                                                         |
| ------------- | ---------------------------- | -------------------------------------------------------------------- |
| Labels list   | `labels`                     | Array (max 2) of label objects.                                     |
| Label entity  | `labels[].entity`            | Sensor/entity id for the label value.                                |
| Label attribute | `labels[].attribute`       | Read from an attribute instead of state.                             |
| Label icon    | `labels[].icon`              | Optional icon shown next to the label.                               |
| Label color   | `labels[].color`             | Optional color override for the label text/icon.                     |
| Label threshold | `labels[].threshold`       | Dims/hides label when below threshold (per `threshold_mode`).        |
| Label decimals | `labels[].decimal_places`   | Decimal places for that label.                                       |
| Label unit    | `labels[].unit` / `labels[].unit_of_measurement` | Display unit override for that label.                    |

## Unit & formatting behavior
- W values auto-convert to kW when ≥ 1000 W.
- kWh values auto-convert to MWh when ≥ 1000 kWh.
- `decimal_places` controls formatting everywhere the value is shown.
- `unit|unit_of_measurement` overrides display text only; calculations always use numeric state/attribute.
