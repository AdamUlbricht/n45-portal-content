---
{}
---

# Overview
As a server operator in **All the Mods 9**, you can trigger sounds at a specific player's location using a combination of `/execute` and `/playsound`.
# Core Syntax
The following command executes the sound at the target's position, ensuring it follows them if they are moving.
Code snippet
```
/execute at <target_player> run playsound <sound_id> <audience> ~ ~ ~ [volume] [pitch]
```
# Parameter Definitions
- **`<target_player>`**: The player where the sound originates (e.g., `PlayerName`).
- **`<sound_id>`**: The internal name of the sound (e.g., `minecraft:entity.creeper.primed`).
- **`<audience>`**: Who hears the sound:
    - `@s`: Only the target player hears it.
    - `@a`: Everyone nearby hears it coming from the target.
- **`~ ~ ~`**: Relative coordinates (centered on the target).
- **`[volume]`**: Range of audibility (1.0 is default).
- **`[pitch]`**: Speed/Frequency (0.5 to 2.0).
# Common Examples

|**Effect**|**Command**|
|---|---|
|**Silent Scare**|`/execute at <name> run playsound minecraft:entity.creeper.primed @s ~ ~ ~ 1.0 1.0`|
|**Public Alert**|`/execute at <name> run playsound minecraft:entity.player.levelup @a ~ ~ ~ 1.0 1.0`|
|**High Pitch**|`/execute at <name> run playsound minecraft:block.anvil.land @s ~ ~ ~ 1.0 2.0`|

# Technical Notes
- **Permissions:** Requires OP Level 2+.
- **Modded Sounds:** Mod-specific sounds follow the `modid:sound_path` format. Use **Tab** to auto-complete available sound IDs in the console.