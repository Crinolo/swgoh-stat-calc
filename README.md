# SWGOH Stat Calculator Readme #

Calculates unit stats for EA's Star Wars: Galaxy of Heroes based on player data.
Accepted data formats are those found in [swgoh.help's API](http://api.swgoh.help) endpoints, specifically the 'player.roster' object from their `/player` endpoint.

## Setup ##

```js
const statCalculator = require('statCalculator.js');
statCalculator.setGameData( gameData );
```

## Methods ##

Examples below make use of the [api-swgoh-help](https://github.com/r3volved/api-swgoh-help/tree/node) package (loaded in to variable `swapi`) to collect the raw data.
See it's documentation to learn more about how to use it to gather this data.

### .setGameData(gameData) ###

Tells the Stat Calculator what to use for the base Game Data.
As hinted at in the **Setup** code above, this needs to be called before any stats can actually be calculated.
Can also be used later to update / reassign the game data, if an update is detected and loaded externally.

#### Paramters ####

`gameData` *Object*\
The Obect used by the Stat Calculator to read raw game data.  It requires a specific format.
An example JSON file of the proper `gameData` object can be found [here](https://swgoh-stat-calc.glitch.me/gameData.json).
That link should remain active and updated, and thus can be used directly to create the data object.
To create the object from [swgoh.help's](http://api.swgoh.help) `/data` endpoint, see the code in [dataBuilder.js](https://glitch.com/edit/#!/swgoh-stat-calc?path=statCalc/dataBuilder.js). (A separate package for this code will be created in the future, but for now, it's just hiding in there).

#### Return Value ####

None.

### .calcCharStats(char [, flags, options] ) ###

Calculates stats for a single character.

#### Parameters ####

`char` *Object*\
The character object to calculate stats for.  Only a single character is allowed.  See `Object Formats` below for more info.

`flags` *Object* `| Optional`\
Optional stat format flags.  See `Flags` below for a list of accepted flags.

`options` *Object* `| Optional`\
Optional stat format instructions more complex than `flags`.  See `Options` below for a breakdown.

#### Return Value ####

The `stats` object for the given `char`.  Does not affect `char` itself.

#### Example ####

```js
// get Player roster from api.swgoh.help
let player = (await swapi.fetchPlayer({
  allycode: 231686213,
  language: "ENG_US",
  project: {
    roster: {
      defId: 1,
      nameKey: 1,
      rarity: 1,
      level: 1,
      gear: 1,
      equipped: 1,
      combatType: 1,
      skills: 1,
      mods: 1,
      relic: 1
    }
  }
}) ).result[0];
// pull Darth Sion out of roster as an example
let char = player.roster.find( unit => unit.defId == "DARTHSION" );
// get stats
char.stats = statCalculator.calcCharStats( char );
```

### .calcShipStats(ship, crew [, flags, options] ) ###

Calculates stats for a single ship.

#### Parameters ####

`ship` *Object*\
The ship object to calculate stats for.  Only a single character is allowed.  See `Object Formats` below for more info.

`crew` *Array*\
Array of crew members belonging to the ship.  Each element is regular character object.  See `Object Formats` below for more info.

`flags` *Object* `| Optional`\
Optional stat format flags.  See `Flags` below for a list of accepted flags.

`options` *Object* `| Optional`\
Optional stat format instructions more complex than `flags`.  See `Options` below for a breakdown.

#### Return Value ####

The `stats` object for the given `ship`.  Does not affect `ship` itself.

#### Example ####

```js
// get Player roster from api.swgoh.help
let player = (await swapi.fetchPlayer({
  allycode: 231686213,
  language: "ENG_US",
  project: {
    roster: {
      defId: 1,
      nameKey: 1,
      rarity: 1,
      level: 1,
      gear: 1,
      equipped: 1,
      combatType: 1,
      skills: 1,
      mods: 1,
      relic: 1
    }
  }
}) ).result[0];
// pulls Hound's Tooth out of roster as an example
let ship = player.roster.find( unit => unit.defId == "HOUNDSTOOTH" );
// pulls Bossk out of roster for example crew
let crew = player.roster.find( unit => unit.defId == "BOSSK" );
// get stats
ship.stats = statCalculator.calcCharStats( ship, [crew] );
```

### .calcRosterStats(units [, flags, options] ) ###

Calls `.calcCharStats()` or `.calcShipStats()` depending on each unit's `combatType` in a roster.

#### Parameters ####

`units` *Array*\
Array of unit objects to calculate stats for.  Each element is regular unit object.  See `Object Formats` below for more info.

`flags` *Object* `| Optional`\
Optional stat format flags.  See `Flags` below for a list of accepted flags.

`options` *Object* `| Optional`\
Optional stat format instructions more complex than `flags`.  See `Options` below for a breakdown.

#### Return Value ####

The number of units that had stats calculated.
The original `units` array has been altered such that each element now has a `.stats` property with the calculated stats.

#### Example ####

```js
// get Player roster from api.swgoh.help
let player = (await swapi.fetchPlayer({
  allycode: 231686213,
  language: "ENG_US",
  project: {
    roster: {
      defId: 1,
      nameKey: 1,
      rarity: 1,
      level: 1,
      gear: 1,
      equipped: 1,
      combatType: 1,
      skills: 1,
      mods: 1,
      relic: 1
    }
  }
}) ).result[0];
// get stats for full roster
let count = statCalculator.calcRosterStats( player.roster );
```

### .calcPlayerStats(players [, flags, options] ) ###

Calls `.calcRosterStats()` for each roster object in the player profile(s) submitted.

#### Parameters ####

`players` *Object* or *Array*\
Full player profile(s).  Either a single player or an array of players is accepted.  See `Object Formats` below for more info.

`flags` *Object* `| Optional`\
Optional stat format flags.  See `Flags` below for a list of accepted flags.

`options` *Object* `| Optional`\
Optional stat format instructions more complex than `flags`.  See `Options` below for a breakdown.

#### Return Value ####

The number of units that had stats calculated.
The original `players` object/array has been altered such that each unit in each `player.roster` object now has a `.stats` property with the calculated stats.

#### Example ####

```js
// get Player roster from api.swgoh.help
let player = (await swapi.fetchPlayer({
  allycode: 231686213,
  language: "ENG_US",
  project: {
    roster: {
      defId: 1,
      nameKey: 1,
      rarity: 1,
      level: 1,
      gear: 1,
      equipped: 1,
      combatType: 1,
      skills: 1,
      mods: 1,
      relic: 1
    }
  }
}) ).result[0];
// get stats for full roster
let count = statCalculator.calcPlayerStats( player );
```

## Options ##

The `options` parameter of all calculation methods is an object that can contain any of the following properties.
Any additional properties of the object will be ignored.\
The *Default* explanations below are what is used when the related flag(s) are not used.

#### Example: ####
```js
{ gameStyle: true, unscaled: true }
```

### Calculation control ###

`withoutModCalc: true`\
Speeds up character calculations by ignoring stats from mods.\
*Default* - calculate mods stats for all characters that include them.

`useValues: {Object}`\
Overrides unit parameters with specific values.
Object structure and total options are as defined below.
Parameters provided here can be missing in the original unit.
```js
{
  char: { // used when calculating character stats
    rarity: 1-7,
    level: 1-90,
    gear: 1-12,
    equipped: "all" || "none" || [1,2,3,4,5,6], // See Below
    relic: 1-9 // 1='locked', 2='unlocked', 3=R1, 4=R2, ...9=R7
  },
  ship: { // used when calculating ship stats
    rarity: 1-7,
    level: 1-90
  },
  crew: { // used for characters when calculating ship stats
    rarity: 1-7,
    level: 1-90,
    gear: 1-12,
    equipped: "all" || "none" || [1,2,3,4,5,6] || 1-6, // See Below
    skills: "max" || "maxNoZeta" || 1-8, // See Below
    modRarity: 1-7,
    modLevel: 1-15,
    relic: 1-9 // 1='locked', 2='unlocked', 3=R1, 4=R2, ...9=R7
  }
}
```
>`char.equipped` / `crew.equipped` - gear currently equipped on characters/crew:
>* *"all"* - all possible gear at the current level.
>* *"none"* - no gear at the current level.
>* *Array* - List of filled slots.  Slots are numbered 1-6, left to right, starting at the top by in-game UI.
>* *Integer* - Number, 1-6, of gear pieces equipped at current level.  Only valid for crew.

>`char.relic` / `crew.relic` - Relic 'Tier' to use\
>Used as the `relic.currentTier` property in .help's data format.\
>Values of 1 and 2 are for 'locked' and 'unlocked', while values >2 are 2 more than the actual Relic Level.

>`crew.skills` - skill level to use for all crew members' abilities:
>* *"max"* - Max possible level.
>* *"maxNoZeta"* - Leaves zeta abilities at level 7, but uses max level for all others.
>* *Integer* - Number, 1-8, to use for all abilities, if possible.\
8 or higher is identical to *"max"*.


*Default* - uses the values defined by the unit objects submitted.\
This applies to each individual property of the `useValues` object, not just the option as a whole.


### Value control ###

`percentVals: true`\
Converts internal flat values for Defense (Armor/Resistance) and Crit Chance (Special/Physical) to the percentages displayed in-game.
Uses the decimal form (i.e. 10% is returned as 0.1)\
*Default* - return the flat values for above stats.

`scaled: true` / `unscaled: true`\
Matches scaling status of values used internally to the game (as seen in portions of [swgoh.help's](http://api.swgoh.help) `/data` endpoint).\
>`scaled` - multiplies all values by 10,000.  All non-modded stats should be integers at this scale.\
`unscaled` - multiplies all values by 100,000,000.  All stats (including mods) fit as integers at this scale.\
*Default* - Stats returned at the expected scale as seen in-game.  Non-percent stats (like Speed) should be integers, all percent stats (like Potency) will be decimals

### Stats Object Style ###

*Default*\
The default Stats Object Style has the following properties:
>`base` *all units* - The base value of of the unit's stats without any stats from mods/gear/crew.
For characters, these are the values used in mods with a percent bonus.
>`gear` *characters* - Amount of stat granted by currently equipped gear (and unused within mod calculations).\
>`mods` *characters* - Amount of stat granted by mods.\
>`crew` *ships* - Amount of stat granted by crew rating.

`gameStyle: true`\
Activates the `percentVals` flag above, and also changes the Stats Object to have the following properties:
>`final` *all units* - Sums values from base, gear, mods, and/or crew into the total stat value.\
>`gear` *characters* - Amount of stat granted by currently equipped gear (and unused within mod calculations).\
>`mods` *characters* - Amount of stat granted by mods.\
>`crew` *ships* - Amount of stat granted by crew rating.


### Stat Naming Options ###

`language: {Object}`\
Tells the calculator to rename the stats using the submitted object.
Used mostly for localization.
*Object* must be such that `options.language[ statID ]` is the stat name, i.e. `{"1": "Health",...}`.\
An example English localization can be seen [here](https://swgoh-stat-calc.glitch.me/lang/eng_us.json).\
Note that a large enough array will also work, as can be seen [here](https://swgoh-stat-calc.glitch.me/lang/statEnum.json).

**Note on `language` keys:** The object/array for `options.language` does not need to be as complete as the above examples
(which cover all 60 possible stats in game code).  Some statIDs that exist in game code are not used (such as id 59 - "UnitStat_Taunt"),
and even more are not returned by this API (such as id 57 - "Speed %" - which converted to the flat "Speed" value, id 5).\
Any statIDs that are not in `options.language` will remain indexed as that integer ID in the return object.

`noSpace: true`\
Converts any stat name strings used in the `language` option into standard camelCase with no spaces.
Only affects stat names defined in that parameter.\
I.e. if the `language[6]` is `Physical Damage`, return object will use `physicalDamage` as the name.

## Object Formats ##

### "*Native*" format -- [swgoh.help's](http://api.swgoh.help) `/player` ###

Player profile object.  Contains a `.roster` property with an array of unit objects.

**Full profile:**
```js
{
  roster: [
    {
      defId: <String>,
      rarity: <Integer>,
      level: <Integer>,
      gear: <Integer>,
      equipped: [
        {
          equipmentId: <String>
        },
      ... ],
      skills: [ // skill list only required for crew members when calculating ship stats
        {
          tier: <Integer>
        },
      ... ],
      mods: [ // can be skipped if using `withoutModCalc` flag for characters only
        {
          pips: <Integer>,
          set: <Integer>,
          level: <Integer>,
          primaryStat: {
            unitStat: <Integer>,
            value: <Number>
          },
          secondaryStat: [
            {
              unitStat: <Integer>,
              value: <Number>
            },
          ... ]
        }
      ... ],
      relic: {
        currentTier: <Integer>
      }
    },
  ... ]
}
```
Used directly by `.calcPlayerStats()`, which also accepts an array of these objects (and [swgoh.help's](http://api.swgoh.help) `/player` endpoint always returns an array)

**player.roster**\
Used directly by `.calcRosterStats()`

**Unit**  *single element of* `player.roster`\
Used directly by `.calcCharStats()` and `.calcShipStats()` (for both the ship and the crew members).

### "*Units*" format -- [swgoh.help's](http://api.swgoh.help) `/units` ###

Object indexed by unit's base ID.  Each such property is an array of unit objects.

**Full Object:**
```js
{
  <BaseID>: [
    {
      starLevel: <Integer>,
      level: <Integer>,
      gearLevel: <Integer>,
      gear: [ <String>, ... ],
      mods: [
        {
          set: <Integer>,
          level: <Integer>,
          stat: [
            [ <Integer>, <Number> ],
          ... ]
        }
      ... ]
    },
  ... ],
  <BaseID>: [ ... ],
... }
```
Only allowed by `.calcRosterStats()`.
As skill info is not included in this format, ship stats cannot be processed.
Only characters will have their stats calculated.
Relic levels are not available in this format, so stats granted by relics will not be included.

**Note:** [swgoh.help's](http://api.swgoh.help) `/roster` endpoint is an array of these objects.
While that array is not directly accepted, each element in the array is a "Units" style object that is accepted as stated above.
