# Petkit T4 ESPHome Firmware

https://github.com/user-attachments/assets/c50be11d-95dd-4898-9601-6adc21f130d6/#Firmware.Dreams.mp4


**Custom.** Fully replaces the stock firmware while introducing some extra neat features, as well as native Home Assistant integration.

It's able to work completely offline & without HA, although it's recommended to calibrate the load cell weight sensor before use.


## Features

### Core Features
- Top (menu) button: Maintenance mode / Reboot
- Bottom (ok) button: Maintenance done & Level the litter
- Advanced status displaying on OLED
- Quieter functioning
- Collected waste amount tracking
- Beep after reaching set amount of litter in maintenance mode
- QR code with a Wi-Fi AP connection shortcut
- Optional [Improv-BLE](https://improv-wifi.com) support
- Various internal sensors exposed to HA
- Additional safety checks compared to the stock fw
- Manual motor controls

### New in v0.11.0

#### Automatic Weight Calibration (Auto-Tare)
- Automatically zeros the weight sensor when conditions are right:
  - Drum is level
  - Tray is closed
  - No pet detected
  - Weight reading is stable
- Can be enabled/disabled via the "Auto Tare" switch
- Manual calibration still available via buttons

#### Multi-Pet Tracking
- Support for up to 4 pets
- Automatic pet detection by weight range
- Configurable weight ranges per pet
- Per-pet statistics:
  - Total visit count
  - Daily visit count (resets at midnight)
  - Last recorded weight
  - Average weight tracking

#### Pet Health Statistics
- Visit frequency tracking
- Weight trend monitoring
- Per-pet average weight calculation
- Reset statistics button

#### Smart Litter Level Tracking
- Tracks litter consumption rate
- "Cleans Remaining" prediction sensor
- "Litter Low" binary sensor (warns when ~3 cleans left)
- "All Litter Removed" button resets counters

#### Quiet Mode / Night Hours
- Configurable quiet hours (default: 22:00 - 07:00)
- During quiet hours:
  - Auto-clean is deferred until morning
  - Motor speeds reduced (50% instead of 100%)
  - Beeper sounds suppressed
- Deferred cleans are processed when quiet time ends
- "Deferred Cleans" counter shows pending operations


## Configuration

### Pet Names
Give your pets names in Home Assistant for better identification:
- `Pet 1 Name` through `Pet 4 Name` - Set custom names (e.g., "Luna", "Max")

The names will appear in:
- "Current Pet Name" sensor
- "Last Pet Event" text sensor
- Home Assistant events

### Pet Weight Ranges
Configure each pet's weight range in Home Assistant:
- `Pet 1 Min Weight` / `Pet 1 Max Weight`
- `Pet 2 Min Weight` / `Pet 2 Max Weight`
- (Up to 4 pets supported)

The system will automatically identify which pet is using the litter box based on their weight.

### Quiet Mode
1. Enable the "Quiet Mode" switch
2. Set "Quiet Start Hour" (e.g., 22 for 10 PM)
3. Set "Quiet End Hour" (e.g., 7 for 7 AM)

### Auto Clean Settings
- `Clean After`: Delay in minutes before auto-clean starts
- `Clean Threshold`: Minimum waste (grams) to trigger cleaning
- `Clean Times`: Number of cleaning cycles per visit


## Weight Sensor Calibration

### Automatic Calibration (Recommended)
1. Enable the "Auto Tare" switch
2. Ensure the litter box is empty (no litter, no pet)
3. Close the tray and wait for the drum to level
4. The system will automatically zero the weight sensor when stable

### Manual Calibration
1. Place the T4 on a reasonably flat surface, make sure it's standing on all four its feet, as load cells are attached to each of them.
2. Press the `Weight Calibration Zero` configuration button until the `Weight` diagnostic sensor reads positive zero exactly.
3. Put a precisely (to 0.01 kg) known weight on top of the T4.
4. Enter its exact weight in decimal kilograms to `Weight Calibration` configuration number input **and press enter**.
5. Remove the weight and check for exact positive zero again, press the button again to reach it.
6. Place the weight again and check for its weight drift, re-input the value if needed.
7. Rinse, repeat as desired (a perfect 0.01 kg precision is known to be reachable on T4).


## Sensors Exposed

### Weight & Pet Sensors
| Sensor | Description |
|--------|-------------|
| Weight | Raw calibrated weight (diagnostic) |
| Pet Weight | Current weight on scale minus litter |
| Pet Body Mass | Calculated pet weight (after visit) |
| Current Pet | Currently detected pet ID (1-4, 0=none) |
| Pet X Visits Today | Daily visit count per pet |
| Pet X Total Visits | Lifetime visit count per pet |
| Pet X Average Weight | Running average weight per pet |

### Litter Management
| Sensor | Description |
|--------|-------------|
| Litter Mass | Amount of litter in grams |
| Waste Collected | Waste in tray (grams) |
| Litter Discarded | Total litter discarded (kg) |
| Cleans Remaining | Estimated cleans before tray full |
| Total Cleans | Lifetime clean count |

### Status Sensors
| Sensor | Description |
|--------|-------------|
| Status | Human-readable current status |
| Tray Full | Waste tray needs emptying |
| Litter Low | Litter level getting low |
| Level Needed | Drum needs leveling |
| Clean Pending | Auto-clean is scheduled |
| Quiet Mode Active | Currently in quiet hours |
| Deferred Cleans | Cleans waiting for quiet time to end |

### Text Sensors
| Sensor | Description |
|--------|-------------|
| Current Pet Name | Name of pet currently inside |
| Last Pet Event | Last pet activity (e.g., "Luna finished (2 min 30 s, 4.25 kg)") |
| Status | Current device status in plain text |

## Home Assistant Integration

### Events for Automations
The device fires native Home Assistant events that you can use in automations:

**`esphome.petkit_pet_entered`** - Fired when a pet enters
```yaml
event_data:
  pet_id: 1          # Pet number (1-4)
  pet_name: "Luna"   # Pet's configured name
  weight: 4.25       # Current weight in kg
```

**`esphome.petkit_pet_exited`** - Fired when a pet leaves
```yaml
event_data:
  pet_id: 1          # Pet number (1-4)
  pet_name: "Luna"   # Pet's configured name
  weight: 4.20       # Final weight in kg
  duration_seconds: 150  # Time spent inside
```

### Example Automation
```yaml
automation:
  - alias: "Notify when cat uses litter box"
    trigger:
      - platform: event
        event_type: esphome.petkit_pet_exited
    action:
      - service: notify.mobile_app
        data:
          title: "Litter Box Used"
          message: "{{ trigger.event.data.pet_name }} finished ({{ (trigger.event.data.duration_seconds / 60) | int }} min)"
```

### Dashboard Card Example
```yaml
type: entities
title: Petkit T4
entities:
  - entity: text_sensor.petkit_t4_status
  - entity: text_sensor.petkit_t4_current_pet_name
  - entity: sensor.petkit_t4_pet_1_visits_today
  - entity: sensor.petkit_t4_pet_1_last_weight
  - entity: sensor.petkit_t4_cleans_remaining
  - entity: binary_sensor.petkit_t4_litter_low
```


## Yet to implement

* [x] Auto cleaning start (a.k.a. non-kitten mode)
* [x] Pet weight saving
* [x] Multiple pets support
* [x] Automated calibration
* [ ] Extra unknown packets
* [ ] Better & more failsafe motor control
* [ ] Parametrized MCU init
* [ ] Dumping the stock calibration data from factory binaries (per-chip)
* [x] Weight as additional safety check
* [ ] Tune the timeouts and speeds


## Thanks to:

- @earlynerd with his [repo](https://github.com/earlynerd/petkit-pura-max-serial-bus) for the pioneering and _enormous_ help in the reverse engineering.
- @dwyschka for the idea and the upstart (earlynerd/petkit-pura-max-serial-bus#1).
- My loved girlfriend Sharea & my lovely kitty Xayah for the lots of testing.


---

https://github.com/user-attachments/assets/ad489f28-23f9-497a-a08b-c182428b3873/#Petkit.T4.ESPHome.Firmware.mp4
