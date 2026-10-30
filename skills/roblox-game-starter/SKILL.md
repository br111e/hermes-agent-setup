---
name: roblox-game-starter
category: creative
description: Use when building a Roblox (Luau) game for the user.
---

# Roblox Game Starter

User (Bagus) builds Roblox games in Studio, starts from scratch, wants to publish. Workflow that worked (Genshin-style open world, 29 Okt 2026):

## Deliverable format
- One zip per iteration: full `WorldCore.lua` (Script/ServerScriptService) + `PlayerUI.lua` (LocalScript/StarterPlayerScripts) + optional `GraphicsConfig.lua` (Script/ServerScriptService) + README.md (install steps) + ASSETS.md (if user generates assets) + PUBLISH.md.
- Scripts must be COMPLETE & runnable — no placeholders except a `WORLD` config table at the top where the user pastes generated asset IDs (`texId = "rbxassetid://..."`).
- Zip via Python `zipfile` (zip CLI includes the zip itself recursively if not careful).

## Hard-won pitfalls
- Luau: local functions are NOT forward-referenced — define helper functions BEFORE the function that calls them.
- Roblox HD visuals: use real Lighting effects (`BloomEffect`, `SunRaysEffect`, `Atmosphere`, `VignetteEffect`, `Clouds`, `Sky`). `Lighting.Technology = Enum.Technology.Future` is the biggest single upgrade. There is no `PostProcess` part+attributes system — don't invent it.
- Verified working skybox IDs (DevForum "Blue-ish Simulator Type Lighting"): 528480826/528481890/528481964/528482047/528482365/528484371 (Bk/Dn/Ft/Lf/Rt/Up).
- DataStore in Studio Play needs Game Settings → "Enable Studio Access to API Services" checked — always include in setup steps.
- Asset pipeline for user-generated art: textures via roblox.com/creatormarketplace/texture/insert (prompt: seamless/tileable/top-down, 1024–2048px); 3D via Meshy/Tripo3D → .fbx → models/insert; skybox = 6 images.
- Teleport/multi-map: store per-player lastMap, CFrame-teleport HumanoidRootPart, killfloor at y=-40 + respawn check.
- NPC dialog: Touched + attribute cooldown, RemoteEvent open/choice, client renders buttons.

## User context
- Assets: user generates his own (AI image/3D). Keep all asset IDs in ONE config table.
- Language: Indonesian, casual "bro".
