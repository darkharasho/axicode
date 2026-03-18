# @mks.haro/axicode

Compact binary build code encoder/decoder for Guild Wars 2 builds. Produces shareable build strings in the **AxiCode** format used by [AxiForge](https://axiforge.com).

## Installation

```bash
npm install @mks.haro/axicode
```

## Usage

```js
const { encodeShareCode, decodeShareCode, isValidShareCode } = require("@mks.haro/axicode");
```

### Encoding a build

Pass a build object to `encodeShareCode` to get a share code string:

```js
const code = encodeShareCode({
  profession: "Warrior",
  gameMode: "pve",
  specializations: [
    {
      id: 4, name: "Strength", elite: false,
      majorChoices: { 1: 1444, 2: 1449, 3: 1437 },
      majorTraitsByTier: {
        1: [{ id: 1444 }, { id: 1447 }, { id: 2000 }],
        2: [{ id: 1449 }, { id: 1448 }, { id: 1453 }],
        3: [{ id: 1437 }, { id: 1440 }, { id: 1454 }],
      },
    },
    // ... two more specializations
  ],
  skills: {
    heal: { id: 14402 },
    utility: [{ id: 14404 }, { id: 14410 }, { id: 14405 }],
    elite: { id: 14355 },
  },
  equipment: {
    statPackage: "Berserker's",
    weapons: { mainhand1: "greatsword", offhand1: "", mainhand2: "axe", offhand2: "" },
    runes: { head: "24836", shoulders: "24836", chest: "24836", hands: "24836", legs: "24836", feet: "24836" },
    sigils: { mainhand1: ["24615", "24868"], offhand1: [], mainhand2: ["24615"], offhand2: [] },
    relic: "Relic of the Thief",
    food: "Bowl of Sweet and Spicy Butternut Squash Soup",
    utility: "Superior Sharpening Stone",
    enrichment: "",
    infusions: {
      head: "49432", shoulders: "49432", chest: "49432", hands: "49432", legs: "49432", feet: "49432",
      back: ["49432", "49432"], ring1: ["49432", "49432", "49432"], ring2: ["49432", "49432", "49432"],
      accessory1: "49432", accessory2: "49432",
      mainhand1: ["49432", "49432"], offhand1: [], mainhand2: ["49432", "49432"], offhand2: [],
    },
  },
  // Profession-specific fields (include as needed)
  selectedLegends: ["", ""],           // Revenant
  selectedPets: { terrestrial1: 0, terrestrial2: 0 }, // Ranger
  activeAttunement: "",                // Elementalist
  activeKit: 0,                        // Engineer
  activeWeaponSet: 1,                  // Warrior
  antiquaryArtifacts: { f2: 0, f3: 0, f4: 0 }, // Thief (Antiquary)
});

console.log(code);
// => "<AxiForge:Berserker:x1y2z3...>"
```

### Decoding a build

```js
const build = decodeShareCode("<AxiForge:Berserker:x1y2z3...>");

console.log(build.profession);              // "Warrior"
console.log(build.gameMode);                // "pve"
console.log(build.equipment.statPackage);   // "Berserker's"
console.log(build.skills.healId);           // 14402
console.log(build.equipment.weapons);       // { mainhand1: "greatsword", ... }
```

### Validating a share code

```js
isValidShareCode("<AxiForge:Berserker:x1y2z3...>"); // true
isValidShareCode("not a share code");                // false
isValidShareCode("[&DQYlPSkvMBc=]");                 // false (GW2 chat link, not AxiCode)
```

## API

### `encodeShareCode(build) → string`

Encodes a build object into an AxiCode share string. The output format is `<AxiForge:{label}:{payload}>`, where the label is the elite specialization name (or profession name for core builds).

### `decodeShareCode(code) → object`

Decodes an AxiCode share string back into a build object. Throws if the format is invalid or the payload is corrupted.

**Returned object shape:**

| Field | Type | Description |
|-------|------|-------------|
| `profession` | `string` | Profession name (e.g. `"Warrior"`) |
| `gameMode` | `string` | `"pve"`, `"pvp"`, or `"wvw"` |
| `specializations` | `array` | Three specs, each with `id` and `traitChoices` (1=top, 2=mid, 3=bottom, 0=none) |
| `skills` | `object` | `{ healId, utilityIds: [3], eliteId }` |
| `underwaterSkills` | `object` | Same shape as `skills`, zeroed if unused |
| `equipment` | `object` | Stat package, weapons, runes, sigils, relic, food, utility, enrichment, infusions |
| `selectedLegends` | `array` | Revenant legend strings |
| `selectedPets` | `object` | Ranger pet IDs |
| `activeAttunement` | `string` | Elementalist attunement |
| `activeKit` | `number` | Engineer active kit skill ID |
| `activeWeaponSet` | `number` | Warrior active weapon set (1 or 2) |
| `antiquaryArtifacts` | `object` | Thief Antiquary artifact skill IDs |

### `isValidShareCode(text) → boolean`

Returns `true` if the string matches the AxiCode wrapper format.

## Supported Values

**Professions:** Guardian, Warrior, Engineer, Ranger, Thief, Elementalist, Mesmer, Necromancer, Revenant

**Weapons:** axe, dagger, mace, pistol, sword, scepter, focus, shield, torch, warhorn, greatsword, hammer, longbow, rifle, shortbow, staff, harpoon, spear, trident

**Stat combos:** Berserker's, Marauder's, Assassin's, Valkyrie, Dragon's, Viper's, Grieving, Sinister, Dire, Rabid, Carrion, Trailblazer's, Knight's, Soldier's, Cleric's, Minstrel's, Harrier's, Ritualist's, Seraph, Zealot's, Celestial

**Game modes:** pve, pvp, wvw

## Format Details

AxiCode uses a compact binary encoding with bit-level packing and [Z85](https://rfc.zeromq.org/spec/32/) encoding for the text payload. The format uses flags to conditionally include optional sections (offhands, second weapon set, underwater, profession-specific data, per-slot runes/infusions), keeping simple builds small while supporting full equipment detail.

## License

MIT
