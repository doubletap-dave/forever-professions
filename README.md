# Forever Profession Guides

A community addon for [Polestar](https://polestar.guide), the leveling guide addon for **World of Warcraft: Forever** (`1.60.1`), by **chvckmachinegun**. It is an independent community addon and is not made by Polestar.

## What's Inside

- **12 Solo 1–300 Profession Guides** (`guides/*.guide`): Alchemy, Blacksmithing, Cooking, Enchanting, Engineering, First Aid, Fishing, Herbalism, Leatherworking, Mining, Skinning, and Tailoring — all curated for WoW: Forever with Mix/Gather/Buy material modes and per-faction trainer splits.
- **Paired Profession Routes** (`guides/pairs/*.guide`): Lockstep multi-profession guides built under our [Multi-Profession Leveling Rules](docs/MULTI_PROFESSION_RULES.md), starting with **Fishing + Cooking 1–300** (universal 0-primary-slot pair for +5% Well Fed kill XP and Fishing Bowl camp buff).
- **Automated CI/CD** (`docs/GITHUB_ACTIONS.md`): Lint and playtest on every push/PR, GitHub Releases zip publishing, and automated uploads to `polestar.guide` on version tags.

## For Players

1. Install [Polestar](https://polestar.guide) for World of Warcraft: Forever first.
2. Install **Forever Profession Guides** via the [Polestar Client](https://polestar.guide/client), from [polestar.guide/community/forever-professions](https://polestar.guide/community/forever-professions), or by downloading the latest zip from [GitHub Releases](https://github.com/doubletap-dave/forever-professions/releases) and unzipping it into `World of Warcraft/_forever_/Interface/AddOns/`.
3. Restart the game. Open `/polestar guides` and look under **Community guides**.
4. **Solo or Multi-Profession:** Open any solo guide on its own, open multiple profession tabs side-by-side, or pick a combined route under **Paired Routes**.

Credits: chvckmachinegun. License: CC-BY-4.0 (see `LICENSE`).
Report issues on [GitHub Issues](https://github.com/doubletap-dave/forever-professions/issues) — this pack is supported by its author, not by Polestar.

## For Authors & Contributors

### Prerequisites

- **Node.js 24+** (`winget install OpenJS.NodeJS.LTS`)
- **Lua 5.1 or LuaJIT** (`winget install DEVCOM.LuaJIT`)
- **Rust / Cargo** (to build `forge`)
- A sibling clone of [`fitzforge/polestar`](https://github.com/fitzforge/polestar) with `npm link`, `forge` built in release mode, and `forge data build` completed.

### Daily Commands

```powershell
polestar doctor                  # Verify Node, Lua, Forever reference data, AddOns dir, pack.json
polestar find "<query>" --json   # Look up verified Forever IDs & coordinates
forge guide lint                 # Validate all .guide files (must be 0 errors, 0 warnings)
polestar playtest                # Simulate every guide end-to-end in Lua
polestar play                    # Preview the pack in the browser playground
polestar dev --play              # Watch, lint, install to WoW: Forever, and live-reload playground
polestar install                 # Build and copy into _forever_/Interface/AddOns/
forge release pack               # Build reproducible release zip into dist/
```

See [docs/MULTI_PROFESSION_RULES.md](docs/MULTI_PROFESSION_RULES.md) and [docs/GITHUB_ACTIONS.md](docs/GITHUB_ACTIONS.md) for authoring conventions and release procedures.
