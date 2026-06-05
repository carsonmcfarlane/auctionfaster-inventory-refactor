# AuctionFaster - Inventory Module Tooltip Refactor

**Fix: Replace unreliable tooltip scanning dependency in WoW inventory parsing pipeline**

---

## Overview

This repository contains a refactored version of the `Inventory.lua` module from the AuctionFaster addon.

The module is responsible for scanning player inventory and classifying items for auction-house use, including bind-state detection and basic item metadata extraction.

After a client patch, tooltip parsing via the `LibGratuity-3.0` third-party library began returning incomplete or `nil` tooltip data during inventory scan cycles.

This resulted in inconsistent or `nil` item classification under scan conditions.

---

## Problem

Original implementation relied on `LibGratuity-3.0` for tooltip inspection:

```lua
local Gratuity = LibStub('LibGratuity-3.0')

-- Inside AddItemToInventory:
Gratuity:SetBagItem(bag, slot)

local n = Gratuity:NumLines()
local firstLine = Gratuity:GetLine(1)

for i = 1, n do
    local line = Gratuity:GetLine(i)
    ...
end
```
## Observed behavior
- Empty or incomplete tooltip lines
- `nil` values during scan cycles

## Solution

Replaced external dependency with a module-owned `GameTooltip` instance.

```lua
local tt = CreateFrame("GameTooltip", "CustomAuctionFasterTooltip", nil, "GameTooltipTemplate")
tt:SetOwner(UIParent, "ANCHOR_NONE")
```
## Updated parsing logic
```lua
-- Inside AddItemToInventory:
tt:ClearLines()
tt:SetBagItem(bag, slot)

local tt_name = "CustomAuctionFasterTooltip"
local n = tt:NumLines()
local firstLine = _G[tt_name.."TextLeft1"]:GetText()

for i = 1, n do
    local line = _G[tt_name.."TextLeft"..i]:GetText()
    ...
end
```

## Key Changes
- Removed LibGratuity-3.0 dependency
- Introduced module-scoped `GameTooltip` instance
- Added `tt:ClearLines()` to prevent stale tooltip state
- Replaced library calls with direct fontstring reads via `_G[]`
- Added defensive `if itemId then` guard in scan loop

## Behavior

No functional gameplay changes were introduced.
- Inventory scan flow unchanged
- Item classification rules unchanged
- Event handling unchanged (`AUCTION_HOUSE_SHOW`, `BAG_UPDATE_DELAYED`)

This is a **parsing-layer dependency refactor only**, improving reliability under modern client timing conditions.

## Outcome
Improved stability of inventory scanning under rapid bag update events by removing reliance on an external tooltip parsing library and using a deterministic tooltip source instead.
