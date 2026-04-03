# Maintaining Your FrogPilot Fork

This repo (`ksclark-projects/openpilot`) is a fork of [FrogPilot](https://github.com/FrogAi/FrogPilot) with custom SDGM/SASCM hardware support on top.

## Branch Structure

- **Branch**: `frogpilot-sdgm-sascm-v2`
- **Base**: FrogPilot release branch (pre-compile commit, with full source)
- **Custom commits** (applied on top of the latest FrogPilot base):
  1. `Fix Sport Plus` — GasRegenCmd DBC fix
  2. `Add 2019 Volt` — 2019 Volt fingerprint and support
  3. `New SDGM Harness` — Removes old SDGM hardware type, simplifies panda safety and car code for the new harness
  4. `SASCM Support` — Adds SASCM CAN message whitelist and interface changes
  5. `Use 0xC9 brake signal for SDGM/SASCM` — Adds `GM_PARAM_FORCE_BRAKE_C9` for brake signal routing

## Prerequisites

Make sure you have the FrogPilot upstream remote configured:

```bash
cd /Users/kclark/Development/openpilot-kclark
git remote add upstream https://github.com/FrogAi/FrogPilot.git
```

Verify your remotes:

```bash
git remote -v
# origin    https://github.com/ksclark-projects/openpilot.git (fetch/push)
# upstream  https://github.com/FrogAi/FrogPilot.git (fetch/push)
```

## Updating to the Latest FrogPilot

### Step 1: Fetch the latest from FrogPilot

```bash
git fetch upstream
```

### Step 2: Find the pre-compile commit

FrogPilot's release branch has a final "Compile FrogPilot" commit that strips all source code and ships only precompiled binaries. You need the commit **before** that one, which still has the full source (including `panda/board/safety/safety_gm.h`).

```bash
git log --oneline upstream/FrogPilot -5
```

You'll see something like:

```
aaaaaaa Compile FrogPilot            <-- SKIP this one
bbbbbbb March 15th, 2026 Patch       <-- USE this one (new base)
bd1c9f1 February 28th, 2026 Patch    <-- this is the current base
```

### Step 3: Rebase your custom commits onto the new base

```bash
git rebase --onto <new-base> <old-base> frogpilot-sdgm-sascm-v2
```

For the example above:

```bash
git rebase --onto bbbbbbb bd1c9f1 frogpilot-sdgm-sascm-v2
```

This takes your 5 custom commits, removes them from the old base, and replays them on top of the new base.

### Step 4: Resolve conflicts (if any)

If there are conflicts, git will pause and tell you which files need attention. For each conflicted file:

```bash
# Open the file and resolve the conflict markers (<<<<<<, =======, >>>>>>>)
# Then stage the resolved file:
git add <resolved-file>

# Continue the rebase:
git rebase --continue
```

Common conflict areas:
- `panda/board/safety/safety_gm.h` — if upstream changes the GM safety code
- `selfdrive/car/gm/values.py` — if upstream changes gas/regen values
- `opendbc/gm_global_a_powertrain_generated.dbc` — if upstream changes the GasRegenCmd signal

When resolving, keep your custom changes (the incoming side) unless the upstream change is clearly a fix you need.

If a conflict is too messy, you can abort and start over:

```bash
git rebase --abort
```

### Step 5: Push to your GitHub repo

Since rebase rewrites history, you need a force push:

```bash
git push origin frogpilot-sdgm-sascm-v2 --force-with-lease
```

## Deploying to Your Comma Device

### Fresh Install

On the comma device, open the browser and navigate to:

```
https://installer.comma.ai/ksclark-projects/openpilot/frogpilot-sdgm-sascm-v2
```

### Updating an Existing Install via SSH

```bash
ssh comma@<device-ip>
cd /data/openpilot
git fetch origin
git reset --hard origin/frogpilot-sdgm-sascm-v2
reboot
```

### Updating via Device UI

If running FrogPilot, go to **Settings > Software** and enter:
- **Fork URL**: `ksclark-projects/openpilot`
- **Branch**: `frogpilot-sdgm-sascm-v2`

### First Boot Note

Since this branch has full source code (not precompiled), the first boot after a fresh install or update will take longer than usual as the device compiles panda and other components. Subsequent boots will be normal.

## Quick Reference

| What | Value |
|------|-------|
| Your repo | `https://github.com/ksclark-projects/openpilot.git` |
| Branch | `frogpilot-sdgm-sascm-v2` |
| Upstream | `https://github.com/FrogAi/FrogPilot.git` |
| Current base commit | `bd1c9f1` (February 28th, 2026 Patch) |
| Custom commits | 5 (Fix Sport Plus → 0xC9 brake signal) |
| Installer URL | `https://installer.comma.ai/ksclark-projects/openpilot/frogpilot-sdgm-sascm-v2` |
