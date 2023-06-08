# A Faster Dead by Daylight&trade;
**This is *only* a proof of concept. The goal is get a 5 - 10s load time for a game (10 including offerings animation).**

## How the hell do you plan on doing that?
Easy, DbD has 3 simple advantages (and an uneccessary disadvantage):

1. All players have the same game data and version
  - All skins and map data are already stored on the users' systems, they've essentially been cached
  - Updates are required, so each player's system knows what everything looks like and their rules of engagement
2. Characters are static
  - Characters do **not** change skins in-game
    - Killer's stance (if applicable)
        + Killer's power status
    - Survivors' status (`Healty` `Injured` `Dying`)
        + Survivors' carried item
3. The game has a *ready-up* feature (the 5s countdown)
  - Once the 5s timer starts nothing can be changed: the map offerings (if used) can be read, decided, and produce a viable map
4. Additionally, there is no need to "load" players into a lobby
  - The player's screen is the same as the lobby screen... Why are they being moved?
  - The sound notification can still be played, just load the other players onto the screen and change the angle
    - This can be accomplished by setting a variable (`GAME_TOKEN` for example) to the connected session's ID

After the 5s timer has started, the server could send back JSON data (for example) of what the map looks like, and where everything important is: players' location (5), killer's item locations (if applicable), generators' location (7), totems' location (5), chests' location (1 - 11), hooks' location (6 - 10, excluding basement), basement location, hatch location, exit gate locations (2), crows' locations, windows' locations, pallets' locations, lockers' locations, and mist level

**Example**

> Game Setup (by Server) - 1578598078584_0123456789abcedf.setup.json `{game time stamp}_{game session hash}.setup.json`
```javascript
{
    "players": {
        "player_0": {
            "character": 0x02, // "WRAITH",
            "skin": {
                "head": 0xE0, // Event head, "Putrid Cavity"
                "body": 0xE0, // Event body, "Withering Vines"
                "item": 0xE0, // Event item, "Oozing Spine"
            },
            "addons": [0x08, 0x0F], // ["coxcombed_clapper", "the_ghost_soot"]
            "perks": [0x20, 0x31, 0x25, 0x19], // ["bloodhound", "infectious_fright", "make_your_choice", "hex_devour_hope"],
            "offering": 0x05, // ["ebony_mori*"]
            "location": {
                x: 'x coordinate',
                y: 'y coordinate',
                z: 'z coordinate',
                r: 'xy rotation'
            },
            "obsession": 0x04 // "player_4"
        },

        "player_1": {
            "character": "MEG_THOMAS",
            "skin": {
                "head": "meg",
                "body": "jogging_hoodie",
                "item": "sport_leggings"
            },
            "item": "medkit",
            "addons": ["rubber_gloves", "bandages"],
            "perks": ["sprint_burst", "urban_evasion", "fixated", "self_care"],
            "offering": "jar_of_salty_lips",
            "location": {
                x: 'x coordinate',
                y: 'y coordinate',
                z: 'z coordinate',
                r: 'xy rotation'
            }
        },
        "player_2": {
            "character": "DAVID_KING",
            "skin": {
                "head": "battle_mullet",
                "body": "foggy_day",
                "item": "cargo_trousers"
            },
            "item": "flashlight",
            "addons": ["odd_bulb", "highend_sapphire_lens"],
            "perks": ["adrenaline", "borrowed_time", "unbreakable", "dance_with_me"],
            "offering": "shiny_coin",
            "location": {
                x: 'x coordinate',
                y: 'y coordinate',
                z: 'z coordinate',
                r: 'xy rotation'
            }
        },
        "player_3": {
            "character": "CLAUDETTE_MOREL",
            "skin": {
                "head": "claudette",
                "body": "teamsxy_gear",
                "item": "producer_jeans"
            },
            "item": "medkit",
            "addons": ["antihaemorrhagic_syringe", ""],
            "perks": ["adrenaline", "borrowed_time", "unbreakable", "dance_with_me"],
            "offering": null,
            "location": {
                x: 'x coordinate',
                y: 'y coordinate',
                z: 'z coordinate',
                r: 'xy rotation'
            }
        },
        "player_4": {
            "character": "YUI_KIMURA",
            "skin": {
                "head": "yui",
                "body": "sakura_7_jacket",
                "item": "racing_pants"
            },
            "item": "key",
            "addons": ["unique_wedding_ring", "weaved_ring"],
            "perks": ["adrenaline", "boil_over", "calm_spirit", "decisive_strike"],
            "offering": "shroud_of_binding*",
            "location": {
                x: 'x coordinate',
                y: 'y coordinate',
                z: 'z coordinate',
                r: 'xy rotation'
            }
        }
    },

    /* main attractions of the game, gets their own special data placement */
    "generators": [
        /* filled in by the server */
    ],

    "basement": {
        x: 'x coordinate',
        y: 'y coordinate',
        z: 'z coordinate',
        r: 'xy rotation'
    },

    "hooks": [
        /* filled in by the server */
    ],

    "gates": [
        /* filled in by the server */
    ],

    /* affects game-play, needs to be read fast */
    "totems": [
        /* filled in by the server */
    ],

    "offerings": [
        "ebony_mori*",
        "jar_of_salty_lips",
        "shroud_of_binding*",
        "shiny_coin",
        null
    ],

    "props": {
        "lockers": [ /* filled in by the server */ ],
        "chests": [ /* filled in by the server */ ],
        "crows": [ /* filled in by the server */ ],
        "walls": [ /* filled in by the server */ ],
        "mist": 0
    }
}
```

> Gameplay (by Server) - **1578598078584_0123456789abcedf.gameplay.json** `{game time stamp}_{game session hash}.gameplay.json`
```javascript
{
    "generators": [
        "generator_0": {
            "status": 0,
                // 0 - not changing
                // 1 - progressing
                // -1 - regressing
            "modifiers": [
                // such as "hex_ruin", "overcharge", "brand_new_part", etc.
            ],
            "progress": 0
                // 0 - no progression
                // 100 - full progression
        },

        // ...
    ],

    "hooks": [
        "hook_1": {
            "status": 0,
                // 0 - not broken
                // -1 - broken
            "modifiers": [
                // such as "hangmans_trick"
            ],
            "progress": 0,
                // 0 - not sabotaged/broken
                // -1 - sabotaged/broken
            "timer": 180
                // 0 - hook is resetting
                // 180 - hook will reset in 180s (3m)
        },

        // ...
    ],

    "totems": [
        "totem_0": {
            "status": 0,
                // 0 - not changing
                // 1 - progressing
            "modifiers": [
                // such as "hex_thrill_of_the_hunt"
            ]
        },

        // ...
    ],

    "pallets": [
        "pallet_0": {
            "status": 0,
                // 0 - up
                // 1 - down
                // -1 - broken
        },

        // ...
    ],

    "lockers": [
        "locker_0": {
            "status": 0,
                // 0 - empty
                // 1 - in use
            "modifiers": [
                // such as "iron_maiden"
            ]
        },

        // ...
    ],

    "walls": {
        "wall_0": {
            "status": 0,
                // 0 - open
                // -1 - blocked
            "modifiers": [
                // such as "bamboozle"
            ],
            "timer": 30
                // 0 - window is opening
                // 30 - window will reset in 30s
        }
    },

    "crows": [
        "crow_0": {
            "status": 0,
                // 0 - undisturbed
                // 1 - disturbed
            "modifiers": [
                // such as "calm_spirit"
            ]
        },

        // ...
    ],

    "gates": [
        "gate_0": {
            "status": 0,
                // 0 - unable to open (locked)
                // 1 - able to open
            "modifiers": [
                // such as "blood_warden"
            ],
            "progress": 0
                // 0 - closed
                // 100 - open
        },

        // ...
    ]
}
```

> Player Action (by Players) - **1578598078584_0123456789abcedf.player_1.json** `{game time stamp}_{game session hash}.player_{player index}.json`
```javascript
{
    "location": {
        x: 'x coordinate',
        y: 'y coordinate',
        z: 'z coordinate',
        r: 'xy rotation'
    },

    "action": 0,
        // 0 - not attacking
            // not running
        // 1 - attacking (not lunge)
            // running
        // 2 - lunging
            // crouching
        // 3 - picking up survivor
            // working (generator, totem, healing, etc.)

    "status": null
        // "primary" - primary power in use
            // item in use
        // "secondary" - secondary power in use
            // item secondary action (such as syringe)
}
```
