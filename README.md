# Daikin FTV28 APGS02 SmartIR codes

Home Assistant SmartIR and Broadlink IR codes, with decode notes, for the Daikin FTV28PV1L9-3WM-G1 indoor unit and APGS02 remote.

> Community-maintained device codes, not an official Daikin configuration. The IR data was captured with Broadlink and decoded against the Daikin64 field/checksum layout. The full temperature/fan matrix has not been physically verified on every unit. Reports from users with the same model are welcome.

## Files

- `9028.json`: SmartIR climate code set for Cool mode, 16-30 C, and Auto/Low/Medium/High fan speeds.
- `9028.annotated.yaml`: human-readable copy of the same codes. SmartIR loads the JSON file, not this YAML file.
- `9028-code-segments.md`: Base64 character spans, Broadlink pulse byte offsets, and Daikin64 state fields for a sample command.

## Why is the file named `9028.json`?

SmartIR identifies custom climate code sets by a numeric `device_code` and loads the JSON file with the same number as its filename. `9028` is a custom ID chosen to be easy to associate with the FTV28 model; it is not a Daikin model number, remote model, or IR protocol identifier. The official SmartIR climate code directory does not currently contain `9028.json`. If you choose another unused numeric ID, rename the JSON file and set `device_code` to that same number.

## SmartIR setup

Place `9028.json` in SmartIR's `codes/climate` directory, then configure the climate entity:

```yaml
climate:
  - platform: smartir
    name: Daikin AC
    unique_id: daikin_ac
    device_code: 9028
    controller_data: remote.your_broadlink_remote
    temperature_sensor: sensor.your_room_temperature
```

Replace the Broadlink remote and temperature sensor with your own entity IDs. See the [SmartIR climate documentation](https://github.com/smartHomeHub/SmartIR/blob/master/docs/CLIMATE.md) for requirements specific to your installed version.

## Important behavior

- `commands.off` is a power-toggle frame, not an absolute off command. Its decoded saved state is 23 C, Medium fan, remote clock 11:05, with the power-toggle bit set.
- Cool temperature/fan codes carry the full AC state and the remote clock. Their power-toggle bit is clear. The clock is from the remote when the code was learned, not the current Home Assistant time.
- Broadlink sends IR without receiving AC state. Home Assistant may display a different state than the unit; sending a toggle while state is uncertain may switch the unit the wrong way.
- The annotated YAML is for reading. Do not change a temperature by replacing a Base64 substring; regenerate the full IR frame and checksum.

## Protocol references

- [Daikin64 protocol fields](https://github.com/crankyoldgit/IRremoteESP8266/blob/master/src/ir_Daikin.h)
- [Daikin64 decoder, BCD, and checksum implementation](https://github.com/crankyoldgit/IRremoteESP8266/blob/master/src/ir_Daikin.cpp)

## Verification limits

The state fields and checksum were checked in software. No IR command was sent during repository preparation, and this device code set has not been exhaustively verified on multiple units.
