# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a ZMK firmware configuration repository for the DAO keyboard - a split wireless keyboard using the ZMK (Zephyr Mechanical Keyboard) firmware. The repo contains keymap definitions and board configurations for DAO44 and DAO56 variants.

**Key Points:**
- Firmware compilation is handled by GitHub Actions (not local)
- Changes to keymaps automatically trigger builds when pushed
- This is a configuration-only repo; you won't write firmware code
- Built firmware is distributed as `.uf2` files via GitHub Actions artifacts

## Repository Structure

```
config/
├── boards/arm/dao/          # Board definitions for DAO keyboard
│   ├── *.keymap            # Keymap definitions (main editing location)
│   ├── *.dts               # Device tree sources (board config)
│   ├── *.dtsi              # Shared device tree includes
│   └── *.yaml              # Board metadata
├── dao.keymap              # Legacy/alternate keymap file
└── west.yml                # Manifest for ZMK dependencies
.github/workflows/
├── build.yml               # CI/CD pipeline - automatically builds on push
docs/
├── FAQ.md                  # Flashing and pairing instructions
build.yaml                  # Specifies which boards to build
```

## Key Files to Know

- **`config/boards/arm/dao/dao.keymap`** - Primary keymap for merged left+right layout. Contains all four layers (default, lower, raise, adjust) with visual ASCII documentation for each.
- **`config/boards/arm/dao/dao_left.keymap` & `dao_right.keymap`** - Individual half keymaps (used when separate halves are needed)
- **`build.yaml`** - Specifies which boards get built. Add/remove boards here.
- **`.github/workflows/build.yml`** - Uses ZMK's official build workflow - don't modify unless changing build process
- **`config/west.yml`** - References ZMK firmware version (currently v0.1). Update here to use different ZMK versions.

## Keymap Format Overview

Keymaps use ZMK's device tree syntax (.keymap files are device tree sources):

```
#define DEF 0     // Layer definitions
#define LWR 1
#define RSE 2
#define ADJ 3

/ {
  keymap {
    compatible = "zmk,keymap";

    default_layer {
      bindings = <
        &kp KEY_NAME  &kp KEY_NAME  ...  // Key press
        &lt LAYER TAB                     // Layer tap (hold for layer, tap for key)
        &mt MODIFIER KEY                 // Mod-tap (hold for modifier, tap for key)
        &bt BT_SEL 0                     // Bluetooth select profile
      >;
    };
  };
};
```

Common ZMK behaviors:
- `&kp` - Key press
- `&lt` - Layer tap
- `&mt` - Mod-tap
- `&bt` - Bluetooth
- `&trans` - Transparent (pass-through to lower layer)

## Workflow

**Editing keymaps:**
1. Edit `.keymap` file (most common: `config/boards/arm/dao/dao.keymap`)
2. Commit and push to any branch
3. GitHub Actions automatically builds firmware
4. Download `firmware.zip` from Actions artifacts
5. Flash to keyboard halves (see `docs/FAQ.md` for flashing procedure)

**Adding/changing layers:**
1. Define layer number at top of keymap file (`#define NEWLAYER 4`)
2. Add new layer block in keymap (`newlayer_layer { bindings = < ... >; };`)
3. Update bindings to reference new layer with `&lt NEWLAYER`

## ZMK Resources

- **Official Docs**: https://zmk.dev/docs
- **Keycode Reference**: https://zmk.dev/docs/codes
- **Behaviors**: https://zmk.dev/docs/behaviors
- **FAQ**: Local at `docs/FAQ.md`

## Common Tasks

**Change a single key:**
Open `config/boards/arm/dao/dao.keymap`, find the key in the grid, replace with desired keycode. Keycodes are defined in ZMK docs (e.g., `N1` for 1, `Q` for Q, `SPACE` for space).

**Add a new layer:**
1. Add `#define NEWNAME 4` (or next available number)
2. Add complete layer block with all key bindings
3. Update bindings in other layers to include `&lt NEWNAME KEY` where you want layer access

**Modify Bluetooth/reset behavior:**
See the `adjust_layer` - contains `&bt BT_SEL`, `&reset`, `&bootloader` actions.

**Modify quick-tap timing:**
Find `&lt { quick_tap_ms = <200>; };` - adjust milliseconds to change responsiveness.

## Build Variants

The repository supports multiple keyboard variants. Check `build.yaml` for currently enabled boards. The workflow builds all included boards and provides them in the artifacts.

## Notes on Development

- **No local build needed** - This is a pure configuration repo. Compilation happens in GitHub Actions.
- **Version pinning** - ZMK version is pinned in `config/west.yml` (currently v0.1). Update here for new features/fixes.
- **Testing changes** - The only way to test is to build via GitHub Actions, download firmware, and flash to hardware.
- **Documentation** - Keymap layout has ASCII art comments showing key positions - keep these updated when changing layouts
