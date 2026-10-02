Create a COMPLETE Minecraft Bedrock Edition (Android/mobile) add-on "ENDLESS MOB BATTLE" as one .mcaddon (behavior pack + resource pack, fresh UUIDs). Target: Bedrock 1.26.52 (v26.50 line). Use ONLY stable Script API: "@minecraft/server" version "2.8.0" in the manifest dependencies. No beta/preview APIs, no experiments toggles. Do NOT use world.beforeEvents.chatSend (beta only).

MANIFEST RULES
- format_version 2, min_engine_version [1,26,30], valid unique UUIDs.
- BP modules: "data" + "script" (language javascript, entry scripts/main.js). BP depends on the RP uuid.
- RP: "resources" module. pack_icon.png at each pack root.
- Package the .mcaddon so each pack is a folder/.mcpack with manifest.json at its root (no extra nested folder).

SCRIPT RULES
- ES modules, small files: main, game_manager, round_manager, mob_manager, kit_manager, arena_manager, ui_manager, config.
- main.js: do not touch the world at top level. Subscribe to world.afterEvents.worldLoad and start everything there.
- Register custom slash commands with system.beforeEvents.startup + customCommandRegistry: /endless:startbattle and /endless:stopbattle (permissionLevel Any, cheatsRequired false). Do the real work inside system.run().
- Use ONE system.runInterval (10 ticks) for all players. Per-player state machine; no uncontrolled intervals.
- Victory via world.afterEvents.entityDie (once only). Handle playerSpawn, playerLeave, and player death with safe cleanup. Wrap risky calls in try/catch.

GAMEPLAY
1. /endless:startbattle starts a 02:00 timer. No mob and no teleport during it.
2. At 0: pick random mob (with its difficulty and matching kit), give the kit, start 10-second BATTLE PREPARATION (show 10..1). No mob yet.
3. After 10s: teleport player to the fight spot, then spawn the mob at the mob spawn.
4. Mob dies: restore full health, remove ONLY kit items (mark them with lore text, keepOnDeath, lockMode inventory), reset effects, round++, return to the lobby, then start a 01:00 timer. The next timer starts ONLY after cleanup.
5. Repeat forever. Each player has independent state.
- Difficulties: EASY (zombie, skeleton, spider), NORMAL (husk, stray, witch, cave_spider, bogged), HARD (vindicator, ravager, enderman, blaze, wither_skeleton), VERY_HARD (evoker, warden). Do NOT include piglin_brute.
- Kits scale with difficulty. Kit armor must only fill EMPTY armor slots. Never delete unrelated items.
- Arena: lobby, playerSpawn, mobSpawn saved in world dynamic properties. Commands: /scriptevent endless:setlobby, endless:setplayer, endless:setmob (set at the player's position). Refuse to start until all three are set.
- UI: actionbar + title only, update once per second, mobile friendly.

BEFORE FINISHING: validate all JSON, UUID uniqueness, import paths, and JS syntax. Clearly separate "static tested" from "tested in Minecraft".
