# ZMK Firmware for Lily58 Pro (Sofle Hardware)

This repository contains ZMK firmware configuration for a Lily58 Pro keyboard that uses Sofle internal hardware. Yes, you read that right - the keyboard is marketed as a Lily58 Pro but runs on Sofle firmware due to its internal wiring.

## Hardware Specifications

- **Controller**: nice!nano v1 (both halves)
- **Case/PCB**: Lily58 Pro form factor with hot-swap sockets
- **Internal Hardware**: Sofle wiring and component layout
- **Features**:
  - OLED displays on both halves
  - Rotary encoders on both halves
  - Wireless operation via Bluetooth

## The Challenge: Why Sofle Firmware?

### The Problem

When initially attempting to build firmware as a standard Lily58, several critical features failed:
- OLED displays would not activate
- Rotary encoders either didn't work or both performed identical functions
- Standard Lily58 shield definitions from ZMK didn't match the hardware

### The Discovery

After 30+ build attempts and extensive troubleshooting, we discovered:
1. The seller shipped the keyboard with Sofle firmware (files named `sofle-left.uf2` and `sofle-right.uf2`)
2. The original Sofle firmware had working OLED displays and independent encoder control
3. Building as Lily58 couldn't replicate this functionality

### The Solution

The keyboard's internal wiring follows the Sofle schematic, not the Lily58. By building with Sofle shields instead of Lily58 shields, all features work correctly:
- ✅ Both OLED displays functional
- ✅ Independent encoder control (left and right can do different things)
- ✅ All keys properly mapped
- ✅ Wireless connectivity stable

## Current Encoder Configuration

**Left Encoder:**
- Turn: Page Up/Down
- Press: Screenshot (macOS Cmd+Shift+4 - area selection)

**Right Encoder:**
- Turn: Volume Up/Down
- Press: Mute toggle

## Building Firmware

This repository uses GitHub Actions to automatically build firmware when changes are pushed.

### Automatic Build (Recommended)

1. Make changes to `config/sofle.keymap` or `config/sofle.conf`
2. Commit and push to GitHub
3. GitHub Actions will automatically build the firmware
4. Download the firmware artifacts from the Actions tab
5. Flash both halves

### Manual Build

If you want to build locally:
```bash
west build -b nice_nano -d build/left -- -DSHIELD=sofle_left
west build -b nice_nano -d build/right -- -DSHIELD=sofle_right
```

## Flashing Instructions

1. Download the firmware artifacts from GitHub Actions
2. Extract the zip file
3. Flash the left half:
   - Connect left controller via USB
   - Double-tap reset button (NICENANO drive appears)
   - Drag `sofle_left-nice_nano-zmk.uf2` to the drive
   - Wait for automatic disconnect
4. Flash the right half:
   - Connect right controller via USB
   - Double-tap reset button
   - Drag `sofle_right-nice_nano-zmk.uf2` to the drive
   - Wait for automatic disconnect

## Keymap Visualization

This repository automatically generates visual keymap diagrams using [keymap-drawer](https://github.com/caksoylar/keymap-drawer).

View the current keymap: [keymap-drawer/sofle.svg](keymap-drawer/sofle.svg)

The diagrams are automatically updated whenever `config/sofle.keymap` is modified.

## Customization

### Modifying Encoder Behavior

Edit `config/sofle.keymap` and find the `sensor-bindings` line in the `default_layer`:
```c
sensor-bindings = <&inc_dec_kp C_VOL_UP C_VOL_DN &inc_dec_kp PG_UP PG_DN>;
```

- First binding pair: Left encoder behavior
- Second binding pair: Right encoder behavior

Common encoder actions:
- `C_VOL_UP C_VOL_DN` - Volume control
- `PG_UP PG_DN` - Page up/down
- `UP DOWN` - Arrow keys
- `C_PREV C_NEXT` - Media previous/next track
- `LC(Z) LC(LS(Z))` - Undo/Redo (macOS: `LG(Z) LG(LS(Z))`)

### Modifying Key Layout

Edit the `bindings` section in each layer of `config/sofle.keymap`. Refer to [ZMK keycodes documentation](https://zmk.dev/docs/codes) for available key codes.

### Enabling/Disabling Features

Edit `config/sofle.conf`:
```
# Enable OLED display
CONFIG_ZMK_DISPLAY=y

# Enable encoders
CONFIG_EC11=y
CONFIG_EC11_TRIGGER_GLOBAL_THREAD=y

# Other configuration options...
```

## Repository Structure
```
.
├── .github/
│   └── workflows/
│       ├── build.yml           # Firmware build workflow
│       └── draw-keymap.yml     # Keymap visualization workflow
├── boards/
│   └── shields/
│       ├── lily58/             # Lily58-specific shield definitions
│       └── sofle/              # Sofle shield definitions (ACTIVE)
├── config/
│   ├── sofle.keymap            # Main keymap configuration
│   └── sofle.conf              # Feature configuration
├── keymap-drawer/              # Auto-generated keymap visualizations
│   ├── sofle.svg               # Visual keymap diagram
│   └── sofle.yaml              # Parsed keymap data
├── keymap_drawer.yaml          # Keymap visualization config
└── build.yaml                  # Build matrix configuration
```

## Troubleshooting

### OLED Not Working

- Ensure you're building with **Sofle** shields, not Lily58
- Verify `CONFIG_ZMK_DISPLAY=y` in `config/sofle.conf`
- Check that both `.uf2` files are flashed correctly

### Encoders Not Working

- Verify `CONFIG_EC11=y` and `CONFIG_EC11_TRIGGER_GLOBAL_THREAD=y` in config
- Ensure sensor-bindings are properly defined in keymap
- Flash both halves with matching firmware versions

### Both Encoders Do The Same Thing

This was a major issue during development. Solution:
- Remove sensor array overrides from `lily58_left.overlay` and `lily58_right.overlay`
- Ensure the shared `sensors` array in `lily58.dtsi` includes both encoders without commas:
```
  sensors = <&left_encoder &right_encoder>;
```

## Lessons Learned

1. **Trust the seller's firmware**: The original `.uf2` files were Sofle for a reason
2. **Hardware can be mislabeled**: Just because it looks like a Lily58 doesn't mean it's wired like one
3. **Read the shield definitions**: The overlay files revealed the true hardware configuration
4. **Devicetree syntax matters**: Commas in sensor arrays break things
5. **Each half is independent**: Split keyboards compile separately for left and right

## Credits

- **ZMK Firmware**: [zmkfirmware.dev](https://zmk.dev/)
- **Keymap Drawer**: [caksoylar/keymap-drawer](https://github.com/caksoylar/keymap-drawer)
- **Sofle Keyboard**: Original design inspiration
- **Lily58**: Case and form factor

## License

This configuration is provided as-is for personal use. Refer to ZMK's license for firmware components.

---

**Note**: If you have a similar "Lily58 Pro" from the same seller and are experiencing issues, try building with Sofle firmware instead!


https://nickcoutsos.github.io/keymap-editor/


