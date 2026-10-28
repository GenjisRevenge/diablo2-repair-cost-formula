Diablo 2 repair cost calculations explained
===========================================

Note: This document explains superior and non-superior runeword body armor, helm, and shield repair costs. Magic/set/unique repair costs aren't discussed.


TL;DR:
------

Non-superior runeword items have lower repair costs because runeword bonuses are excluded from the repair cost calculation. Quite often, you pay only for the base item and the inserted runes/gems/jewels. Superior items factor all bonuses into their repair cost.

Automods on non-superior bases also cost less to repair (e.g. +skills on assassin claws) or may incur no repair cost at all (e.g. paladin shield resist).


Details
-------

- The game computes a generic cost value for the item and derives the vendor buy/sell/repair costs from it using separate multipliers defined in the npc.txt data file.
- The number of sockets does not affect the cost.
- The max durability does not affect the full repair cost (where "full" means repairing from zero durability).
- The repair cost of an armor base item is scaled by its base defense roll. In this context "armor" refers to protective equipment like body armor, shield, helm, gloves, boots, ...
- The repair bill is always computed as if the item were fully damaged (zero durability) and then scaled according to current and max durability.
- The repair bill is the sum of the base item repair cost (scaled by armor defense), the costs of socketed items, the costs associated with item bonuses (including runeword bonuses), and the costs of spent charges, if any.
- The cost of an item bonus is derived from the base item's repair cost, the rolled bonus value, and bonus-specific constants defined in the game data files. The influence of the base item's cost explains why superior runeword items can cost orders of magnitude less to repair when made in inexpensive low-level bases (e.g. superior Enigma in Ring Mail vs Sacred Armor).
- Non-superior items completely ignore runeword bonus costs and some automods like paladin shield resist, but they do include skill automod costs (e.g. assassin claw skills, necro wand skills), though the skill bonus costs are still significantly lower on non-superior items. This explains the low repair costs of non-superior items versus the extreme repair costs of superior items.


Example calculations
--------------------

We will calculate the repair costs of the following items:

- Non-superior 524 def Archon Plate (empty, 0/60 durability)
- Superior 15/0 Archon Plate (empty, 0/60 durability)
- Non-superior 249 def Mage Plate (empty, 0/60 durability)
- Non-superior 249 def Mage Plate with Jah+Ith (0/60 durability)
- Non-superior 249 def Mage Plate Enigma (+750 def)
    - 0/60 durability
    - 59/60 durability


| body armor runeword   | superior 15/15 base (0 durability) |
|:----------------------|:-----------------------------------|
| Enigma                | ds, ap, mp, fpm, bp                |
| Chains of Honor (CoH) | ds, ap                             |
| Dragon                | ds, ap                             |
| Fortitude             | ds, ap                             |
| Duress                | ds, ap                             |
| Hustle                | ds, ap                             |
| Treachery             | ds, ap                             |
| Smoke                 | ds, ap, mp                         |
| Stealth               | ds, mp, bp                         |


| shield runeword  | superior 15/15 base with (0 durability) |
|:-----------------|:----------------------------------------|
| Phoenix          | st, mon                                 |
| Dream            | st                                      |
| Dragon           | st                                      |
| Spirit           | st, protector, mon                      |
| Ancient's Pledge | st, protector, large                    |


| helm runeword    | superior 15/15 base (0 durability) |
|:-----------------|:-----------------------------------|
| Dream            | bv, dia, great                     |
| Flickering Flame | bv, dia, great                     |
| Hearth           | bv, dia, great                     |
| Lore             | bv, dia, grim, bone, great, helm   |


**The required game data:**

I include all the necessary data in this text, but wanted to explain where it comes from. The extracted game data contains a `data/global/excel` directory with ~90 .txt files that happen to be TSV spreadsheets (here is an [online copy](https://github.com/pinkufairy/D2R-Excel) of them). TSV = Tab Separated Values, just like CSV = Comma Separated Values. These spreadsheets are like the tables of a relational database, and they can be used to comfortably mod the game with a spreadsheet editor like Excel.

There is an official guide next to these .txt files (_diabloiidatafileguide.mht) explaining the spreadsheet structure and the fields. [Improved versions of it](https://locbones.github.io/D2R_DataGuide/) can be found online.

The spreadsheets are huge, so I include only a few cells that we need:

[npc.txt](https://locbones.github.io/D2R_DataGuide/#NPC):

| npc    | rep mult |
|:-------|:---------|
| charsi | 128      |

From [the game data file guide for modders](https://locbones.github.io/D2R_DataGuide/#NPC):

>**rep mult** - **[N]** - Used to calculate the cost to repair an item. This number is a fraction of 1024 in the following formula: [cost] * [rep mult] / 1024. This is then used to influence the repair cost based on the item durability and charges


[armor.txt](https://locbones.github.io/D2R_DataGuide/#AMW):

| name             | cost   | minac | maxac | durability |
|:-----------------|:-------|:------|:------|:-----------|
| Breast Plate     | 10078  | 65    | 68    | 50         |
| Full Plate Mail  | 47192  | 150   | 161   | 70         |
| Mage Plate       | 118407 | 225   | 261   | 60         |
| Dusk Shroud      | 218357 | 361   | 467   | 20         |
| Archon Plate     | 317526 | 410   | 524   | 60         |
| Large Shield     | 1214   | 12    | 14    | 24         |
| Protector Shield | 52947  | 129   | 153   | 40         |
| Sacred Targe     | 72618  | 126   | 158   | 45         |
| Monarch          | 81896  | 133   | 148   | 86         |
| Helm             | 1558   | 15    | 18    | 24         |
| Great Helm       | 6177   | 30    | 35    | 40         |
| Bone Helm        | 6323   | 33    | 36    | 40         |
| Grim Helm        | 37683  | 60    | 125   | 40         |
| Diadem           | 58000  | 50    | 60    | 20         |
| Bone Visage      | 87274  | 100   | 157   | 40         |

The cost field in armor.txt assumes max defense roll (`maxac` = Max Armor Class, the game data often calls it "ac" or "armor class" instead of "defense").

NOTE: The base defense of an armor that drops with Enhanced Defense (even if it's non-superior or not a body armor) is always `maxac+1` [as explained on The Arreat Summit](https://classic.battle.net/diablo2exp/items/basics.shtml#superior). This can be verified with character editors as well. For example: a superior non-eth Archon Plate with Enhanced Defense always has 524+1=525 base defense, though this can be altered in character editors. Ethereal armor with Enhanced defense has `[(maxac+1)*1.5]` base defense.


[misc.txt](https://locbones.github.io/D2R_DataGuide/#AMW):

| name       | cost   |
|:-----------|:-------|
| El Rune    | 560    |
| Eld Rune   | 560    |
| Tir Rune   | 1260   |
| Nef Rune   | 1260   |
| Eth Rune   | 2240   |
| Ith Rune   | 2240   |
| Tal Rune   | 3500   |
| Ral Rune   | 5040   |
| Ort Rune   | 6860   |
| Thul Rune  | 8960   |
| Amn Rune   | 11340  |
| Sol Rune   | 14000  |
| Shael Rune | 16940  |
| Dol Rune   | 20160  |
| Hel Rune   | 1715   |
| Io Rune    | 27440  |
| Lum Rune   | 31500  |
| Ko Rune    | 35840  |
| Fal Rune   | 40460  |
| Lem Rune   | 45360  |
| Pul Rune   | 50540  |
| Um Rune    | 56000  |
| Mal Rune   | 61740  |
| Ist Rune   | 67760  |
| Gul Rune   | 74060  |
| Vex Rune   | 80640  |
| Ohm Rune   | 87500  |
| Lo Rune    | 94640  |
| Sur Rune   | 102060 |
| Ber Rune   | 109760 |
| Jah Rune   | 117740 |
| Cham Rune  | 126000 |
| Zod Rune   | 134540 |


[itemstatcost.txt](https://locbones.github.io/D2R_DataGuide/#ItemStatCost):

Game data to calculate item bonus costs:

| Stat                       | Multiply | Add   | Encode | op | op param |
|:---------------------------|:---------|:------|:-------|:---|:---------|
| strength                   | 55       | 125   |        |    |          |
| energy                     | 55       | 100   |        | 8  |          |
| dexterity                  | 55       | 125   |        |    |          |
| vitality                   | 55       | 100   |        | 9  |          |
| maxhp                      | 20       | 56    |        |    |          |
| maxmana                    | 20       | 81    |        |    |          |
| maxstamina                 | 20       | 75    |        |    |          |
| item_armor_percent         | 20       | 47    |        | 13 |          |
| item_maxdamage_percent     | 20       | 45    |        | 13 |          |
| item_mindamage_percent     | 20       | 45    |        | 13 |          |
| manarecoverybonus          |          |       |        |    |          |
| armorclass                 | 10       | 17    |        |    |          |
| armorclass_vs_missile      | 5        | 11    |        |    |          |
| normal_damage_reduction    | 200      | 188   |        |    |          |
| magic_damage_reduction     | 340      | 397   |        |    |          |
| damageresist               | 68       | 152   |        |    |          |
| fireresist                 | 20       | 43    |        |    |          |
| maxfireresist              | 256      | 584   |        |    |          |
| lightresist                | 20       | 43    |        |    |          |
| maxlightresist             | 256      | 584   |        |    |          |
| coldresist                 | 20       | 43    |        |    |          |
| poisonresist               | 20       | 43    |        |    |          |
| coldmindam                 | 512      | 451   |        |    |          |
| coldmaxdam                 | 340      | 128   |        |    |          |
| lifedrainmindam            | 341      | 1044  |        |    |          |
| hpregen                    | 410      | 451   |        |    |          |
| item_maxdurability_percent | 10       | 117   |        | 13 |          |
| item_maxhp_percent         | 204      | 32093 |        | 11 |          |
| item_maxmana_percent       | 204      | 56452 |        | 11 |          |
| item_attackertakesdamage   | 128      | 112   |        |    |          |
| item_goldbonus             | 34       | 187   |        |    |          |
| item_magicbonus            | 102      | 577   |        |    |          |
| item_addclassskills        | 1560     | 49523 |        |    |          |
| item_healafterkill         | 101      | 30    |        |    |          |
| item_lightradius           | 51       | 15    |        |    |          |
| item_fasterattackrate      | 156      | 1042  |        |    |          |
| item_fastermovevelocity    | 156      | 4083  |        |    |          |
| item_nonclassskill         | 327      | 181   | 1      |    |          |
| item_fastergethitrate      | 72       | 1065  |        |    |          |
| item_fastercastrate        | 156      | 3876  |        |    |          |
| item_poisonlengthresist    | 10       | 27    |        |    |          |
| item_damagetomana          | 20       | 43    |        |    |          |
| item_halffreezeduration    | 988      | 5096  |        |    |          |
| item_demondamage_percent   | 12       | 19    |        |    |          |
| item_undeaddamage_percent  | 12       | 13    |        |    |          |
| item_elemskill             | 1024     | 76    |        |    |          |
| item_allskills             | 4096     | 15123 |        |    |          |
| item_openwounds            | 10       | 23    |        |    |          |
| item_crushingblow          | 40       | 98    |        |    |          |
| item_manaafterkill         | 102      | 17    |        |    |          |
| item_absorbfire            | 204      | 1739  |        |    |          |
| item_absorbmagic           | 204      | 1739  |        |    |          |
| item_absorbcold_percent    | 102      | 5486  |        |    |          |
| item_aura                  |          |       |        |    |          |
| item_cannotbefrozen        | 2048     | 15011 |        |    |          |
| item_staminadrainpct       | 20       | 102   |        |    |          |
| item_skillonhit            | 256      | 190   | 2      |    |          |
| item_skillonlevelup        | 6        | 7     | 2      |    |          |
| item_skillongethit         | 256      | 190   | 2      |    |          |
| item_charged_skill         | 256      | 401   | 3      |    |          |
| item_hp_perlevel           | 64       | 92    |        | 2  | 3        |
| item_mana_perlevel         | 128      | 90    |        | 2  | 3        |
| item_strength_perlevel     | 128      | 132   |        | 2  | 3        |
| item_find_magic_perlevel   | 1024     | 814   |        | 2  | 3        |
| passive_fire_pierce        | 2578     |       |        |    |          |


Assume zero in case of empty fields. Our calculations will be largely unaffected by the `Encode` and `op` columns, but I included them to make you aware of their existence in case you decide to dig deeper into the documentation of the cost calculation.

The stats with op=2 ("by level operator" according to Blizzard's docs) and "op param"=3 in the table are expressed in units of 1/8. "op param"=3 causes the stat value to be right-shifted (>>) by 3 bits, which is equivalent to dividing by 8 (2^3). A good example to this is Enigma's item_find_magic_perlevel=8, you get 8/8=1 MF per level. Fortitude's item_hp_perlevel works the same way: it rolls between 8 and 12, resulting in 8/8=1 to 12/8=1.5 life per level. Blizzard's documentation is incorrect and instead uses the left-shift (<<) operator, which would multiply by 8 (2^3). Character editors usually expect these stats in units of 1/8, sometimes without an explanation, which can be very confusing to casual users.

Runewords are defined in [runes.txt](https://locbones.github.io/D2R_DataGuide/#Runes) that specifies runeword bonuses by referencing the rows of the [properties.txt](https://locbones.github.io/D2R_DataGuide/#Properties) spreadsheet, and properties.txt has references to the rows of the [itemstatcost.txt](https://locbones.github.io/D2R_DataGuide/#ItemStatCost) included above. For example, the Stealth runeword in runes.txt adds 25 fcr by referencing the "cast2" row of properties.txt, and that row references the "item_fastercastrate" row of itemstatcost.txt. Similarly, gem/rune bonuses can be looked up in gems.txt that references the rows of properties.txt.

Enhanced Defense (item_armor_percent) and Enhanced Damage (item_mindamage_percent, item_maxdamage_percent) are calculated differently for armors and weapons. Fortitude has both of these bonuses, and can be rolled into both body armors and weapons.

Certain things have their cost-related fields (multiply, add) in other data files (skills.txt, uniqueitems.txt, setitems.txt).


[skills.txt](https://locbones.github.io/D2R_DataGuide/#Skills):

| skill          | cost mult | cost add |
|:---------------|:----------|:---------|
| Evade          | 768       | 32000    |
| Blaze          | 512       | 8000     |
| Teleport       | 640       | 16000    |
| Chilling Armor | 768       | 32000    |
| Hydra          | 896       | 64000    |
| Weaken         | 384       | 3000     |
| Confuse        | 640       | 16000    |
| Firestorm      | 256       | 1000     |
| Fade           | 640       | 8000     |
| Venom          | 896       | 64000    |

When the bonus is an oskill (+1 Teleport, +6 Evade), charges, or "X% chance to cast SKILL when Y", you have to use the skill level as the bonus value and the mult/add values of the skill from the skills.txt file instead of the values defined in itemstatcost.txt.


**The repair cost formula:**

This formula does not include recharge costs or non-superior automod costs because I haven't analysed things deeply enough to account for every case. However, the examples below are sufficient to illustrate the most important aspects of repair cost calculation.

Square brackets are used to indicate rounding down (after division). The game uses integer arithmetic so fractions are dropped at every step during the calculations.

```
Armor (Chest, Shield, Helm) repair cost formula
===============================================

scaled_armor_cost = [cost_from_armor_txt * defense_roll / maxac_from_armor_txt]
socketed_items_cost = sum of [jewel_gem_rune_cost_from_misc_txt / 2] for each socketed item
sum_of_bonus_costs = for each bonus (ignore runeword bonuses on non-superior bases):
    if bonus_value * scaled_armor_cost < 65536 then
        bonus_cost = [ bonus_value * scaled_armor_cost * multiply_from_itemstatcost_txt / 1024 ] + add_from_itemstatcost_txt
    else
        bonus_cost = [ bonus_value * scaled_armor_cost / 1024 ] * multiply_from_itemstatcost_txt + add_from_itemstatcost_txt
total = scaled_armor_cost + socketed_items_cost + sum_of_bonus_costs

scaled_repair_cost = [total * (max_durability - current_durability) / max_durability]
if scaled_repair_cost < 65536 then
    repair_cost = [ (scaled_repair_cost * 128) / 1024 ]
else
    repair_cost = [ scaled_repair_cost / 1024 ] * 128
```

The above formula can be used to calculate the repair cost of an empty or partially socketed base as well (e.g. half-finished runewords, or 1-2 gems inserted). The repair costs of other item types (e.g: weapons) are calculated roughly the same way.

There will be two bonuses in our example calculations that use slightly different cost formulas: "Enhanced Defense", and "Increase Maximum Durability".

```
Enhanced Defense (item_armor_percent) cost formula
==================================================

if the item type is armor
    extra_defense_points = [defense_roll * enhanced_defense / 100]
    if extra_defense_points = 0 then
        bonus_cost = 0
    else
        if extra_defense_points * scaled_armor_cost < 65536 then
            bonus_cost = [ extra_defense_points * scaled_armor_cost * 20 / 1024 ] + 47
        else
            bonus_cost = [ extra_defense_points * scaled_armor_cost / 1024 ] * 20 + 47
        bonus_cost = [bonus_cost / 2]
else
    if base_item_cost * enhanced_defense < 65536 then
        bonus_cost = [ base_item_cost * enhanced_defense * 20 / 1024 ] + 47
    else
        bonus_cost = [ base_item_cost * enhanced_defense / 1024 ] * 20 + 47


Increase Maximum Durability (item_maxdurability_percent) cost formula
=====================================================================

extra_durability_points = [max_durability * increase_max_durability / 100]
if extra_durability_points = 0 then
    bonus_cost = 0
else
    if extra_durability_points * scaled_armor_cost < 65536 then
        bonus_cost = [ extra_durability_points * scaled_armor_cost * 10 / 1024 ] + 117
    else
        bonus_cost = [ extra_durability_points * scaled_armor_cost / 1024 ] * 10 + 117
    bonus_cost = [bonus_cost * 10 / 25]
```

Why must the bonus costs from "Enhanced Defense" be divided by 2, and those from "Increase Maximum Durability" be divided by 2.5? I have no definitive explanation, but empirical testing provides strong evidence. By observing Charsi's repair costs on character-edited armors with widely varying defense, durability, ED, and IMD values, these divisors consistently emerge. Using a character editor to assign extreme values (like ED ranging from 1 to ~500 and IMD from 1 to ~100) makes the underlying relationships even clearer and allows these conclusions to be drawn with confidence.


**Calculations:**

```
Non-superior 524 def Archon Plate (empty, 0 durability)
=======================================================

scaled_armor_cost = [317526 * 524 / 524] = 317526
socketed_items_cost                      = 0

total                                    = 317526

scaled_repair_cost = [317526 * (60-0) / 60] = 317526
repair_cost = [ 317526 / 1024 ] * 128 = 39680  (accurate in-game value: 39680)


Superior 15/0 Archon Plate (empty, 0 durability)
================================================

scaled_armor_cost = [317526 * (524+1) / 524]                        = 318131
socketed_items_cost                                                 = 0

bonus_costs:
  15  item_armor_percent  [ (524+1) * 15 / 100 ] = 78
                          [ 318131 * 78 / 1024 ] * 20 + 47 = 484687
                          [ 484687 / 2 ]                            = 242343

total = 318131 + 242343                                             = 560474

scaled_repair_cost = [560474 * (60-0) / 60] = 560474
repair_cost = [ 560474 / 1024 ] * 128 = 70016  (accurate in-game value: 69632)


Non-superior 249 def Mage Plate (empty, 0 durability)
=====================================================

scaled_armor_cost = [118407 * 249 / 261] = 112963
socketed_items_cost                      = 0

total = 112963 + 0                       = 112963

scaled_repair_cost = [112963 * (60-0) / 60] = 112963
repair_cost = [ 112963 / 1024 ] * 128 = 14080  (accurate in-game value: 14080)


Non-superior 249 def Mage Plate with Jah+Ith (0 durability)
===========================================================

scaled_armor_cost = [118407 * 249 / 261]        = 112963
socketed_items_cost = [117740 / 2] + [2240 / 2] = 59990

total = 112963 + 59990                          = 172953

scaled_repair_cost = [172953 * (60-0) / 60] = 172953
repair_cost = [ 172953 / 1024 ] * 128 = 21504  (accurate in-game value: 21504)


Non-superior 249 def Mage Plate Enigma (+750 def roll, 0 and 59 durability)
===========================================================================

scaled_armor_cost = [118407 * 249 / 261]                       = 112963
socketed_items_cost = [117740 / 2] + [2240 / 2] + [109760 / 2] = 114870

total = 112963 + 114870                                        = 227833

- 0/60 durability:
    scaled_repair_cost = [227833 * (60-0) / 60] = 227833
    repair_cost = [ 227833 / 1024 ] * 128 = 28416  (accurate in-game value: 28416)

- 59/60 durability:
    scaled_repair_cost = [227833 * (60-59) / 60] = 3797
    repair_cost = [ 3797 * 128 / 1024 ] = 474  (accurate in-game value: 474)


Superior 15/15 Dusk Shroud Enigma (0 durability)
================================================

scaled_armor_cost = [218357 * (467+1) / 467]                             = 218824
socketed_items_cost = [117740 / 2] + [2240 / 2] + [109760 / 2]           = 114870

bonus_costs:
  15  item_armor_percent          [ (467+1) * 15 / 100 ] = 70
                                  [ 218824 * 70 / 1024 ] * 20 + 47 = 299207
                                  [ 299207 / 2 ]                         = 149603
  15  item_maxdurability_percent  [ 20 * 15 / 100 ] = 3
                                  [ 218824 * 3 / 1024 ] * 10 + 117 = 6527
                                  [ 6527 * 10 / 25 ]                     = 2610
  775 armorclass                  [ 218824 * 775 / 1024 ] * 10   + 17    = 1656147
  14  item_healafterkill          [ 218824 * 14  / 1024 ] * 101  + 30    = 302121
  45  item_fastermovevelocity     [ 218824 * 45  / 1024 ] * 156  + 4083  = 1504179
  6   item_strength_perlevel      [ 218824 * 6   / 1024 ] * 128  + 132   = 164228
  2   item_allskills              [ 218824 * 2   / 1024 ] * 4096 + 15123 = 1764115
  8   item_find_magic_perlevel    [ 218824 * 8   / 1024 ] * 1024 + 814   = 1750830
  1   item_nonclassskill          [ 218824 * 1   / 1024 ] * 640  + 16000 = 152320
      (Teleport, skills.txt: mul=640 add=16000)
  5   item_maxhp_percent (Jah)    [ 218824 * 5   / 1024 ] * 204  + 32093 = 249965
  15  item_damagetomana (Ith)     [ 218824 * 15  / 1024 ] * 20   + 43    = 64143
  8   damageresist (Ber)          [ 218824 * 8   / 1024 ] * 68   + 152   = 116364

total = 218824 + 114870 + 149603 + 2610 + 1656147 + 302121 + 1504179 +
        164228 + 1764115 + 1750830 + 152320 + 249965 + 64143 + 116364    = 8210319

scaled_repair_cost = [8210319 * (23-0) / 23] = 8210319
repair_cost = [ 8210319 / 1024 ] * 128 = 1026176  (accurate in-game value: 1026048)


Superior 15/15 Archon Plate Enigma (0 durability)
=================================================

scaled_armor_cost = [317526 * (524+1) / 524]                             = 318131
socketed_items_cost = [117740 / 2] + [2240 / 2] + [109760 / 2]           = 114870

bonus_costs:
  15  item_armor_percent          [ (524+1) * 15 / 100 ] = 78
                                  [ 318131 * 78 / 1024 ] * 20 + 47 = 484687
                                  [ 484687 / 2 ]                         = 242343
  15  item_maxdurability_percent  [ 60 * 15 / 100 ] = 9
                                  [ 318131 * 9 / 1024 ] * 10 + 117 = 28077
                                  [ 28077 * 10 / 25 ]                    = 11230
  775 armorclass                  [ 318131 * 775 / 1024 ] * 10   + 17    = 2407737
  14  item_healafterkill          [ 318131 * 14  / 1024 ] * 101  + 30    = 439279
  45  item_fastermovevelocity     [ 318131 * 45  / 1024 ] * 156  + 4083  = 2184963
  6   item_strength_perlevel      [ 318131 * 6   / 1024 ] * 128  + 132   = 238724
  2   item_allskills              [ 318131 * 2   / 1024 ] * 4096 + 15123 = 2558739
  8   item_find_magic_perlevel    [ 318131 * 8   / 1024 ] * 1024 + 814   = 2545454
  1   item_nonclassskill          [ 318131 * 1   / 1024 ] * 640  + 16000 = 214400
      (Teleport, skills.txt: mul=640 add=16000)
  5   item_maxhp_percent (Jah)    [ 318131 * 5   / 1024 ] * 204  + 32093 = 348905
  15  item_damagetomana (Ith)     [ 318131 * 15  / 1024 ] * 20   + 43    = 93243
  8   damageresist (Ber)          [ 318131 * 8   / 1024 ] * 68   + 152   = 169132

total = 318131 + 114870 + 242343 + 11230 + 2407737 + 439279 + 2184963 +
        238724 + 2558739 + 2545454 + 214400 + 348905 + 93243 + 169132    = 11887150

scaled_repair_cost = [11887150 * (69-0) / 69] = 11887150
repair_cost = [ 11887150 / 1024 ] * 128 = 1485824  (accurate in-game value: 1485824)


Superior 15/15 Mage Plate Enigma (0 durability)
===============================================

scaled_armor_cost = [118407 * (261+1) / 261]                             = 118860
socketed_items_cost = [117740 / 2] + [2240 / 2] + [109760 / 2]           = 114870

bonus_costs:
  15  item_armor_percent          [ (261+1) * 15 / 100 ] = 39
                                  [ 118860 * 39 / 1024 ] * 20 + 47 = 90567
                                  [ 90567 / 2 ]                          = 45283
  15  item_maxdurability_percent  [ 60 * 15 / 100 ] = 9
                                  [ 118860 * 9 / 1024 ] * 10 + 117 = 10557
                                  [ 10557 * 10 / 25 ]                    = 4222
  775 armorclass                  [ 118860 * 775 / 1024 ] * 10   + 17    = 899587
  14  item_healafterkill          [ 118860 * 14  / 1024 ] * 101  + 30    = 164155
  45  item_fastermovevelocity     [ 118860 * 45  / 1024 ] * 156  + 4083  = 818871
  6   item_strength_perlevel      [ 118860 * 6   / 1024 ] * 128  + 132   = 89220
  2   item_allskills              [ 118860 * 2   / 1024 ] * 4096 + 15123 = 965395
  8   item_find_magic_perlevel    [ 118860 * 8   / 1024 ] * 1024 + 814   = 951086
  1   item_nonclassskill          [ 118860 * 1   / 1024 ] * 640  + 16000 = 90240
      (Teleport, skills.txt: mul=640 add=16000)
  5   item_maxhp_percent (Jah)    [ 118860 * 5   / 1024 ] * 204  + 32093 = 150413
  15  item_damagetomana (Ith)     [ 118860 * 15  / 1024 ] * 20   + 43    = 34863
  8   damageresist (Ber)          [ 118860 * 8   / 1024 ] * 68   + 152   = 63256

total = 118860 + 114870 + 45283 + 4222 + 899587 + 164155 + 818871 +
        89220 + 965395 + 951086 + 90240 + 150413 + 34863 + 63256         = 4510321

scaled_repair_cost = [4510321 * (69-0) / 69] = 4510321
repair_cost = [ 4510321 / 1024 ] * 128 = 563712  (accurate in-game value: 563200)


Superior 15/15 Full Plate Mail Enigma (0 durability)
====================================================

scaled_armor_cost = [47192 * (161+1) / 161]                             = 47485
socketed_items_cost = [117740 / 2] + [2240 / 2] + [109760 / 2]          = 114870

bonus_costs:
  15  item_armor_percent          [ (161+1) * 15 / 100 ] = 24
                                  [ 47485 * 24 / 1024 ] * 20 + 47 = 22287
                                  [ 22287 / 2 ]                         = 11143
  15  item_maxdurability_percent  [ 70 * 15 / 100 ] = 10
                                  [ 47485 * 10 / 1024 ] * 10 + 117 = 4747
                                  [ 4747 * 10 / 25 ]                    = 1898
  775 armorclass                  [ 47485 * 775 / 1024 ] * 10   + 17    = 359397
  14  item_healafterkill          [ 47485 * 14  / 1024 ] * 101  + 30    = 65579
  45  item_fastermovevelocity     [ 47485 * 45  / 1024 ] * 156  + 4083  = 329499
  6   item_strength_perlevel      [ 47485 * 6   / 1024 ] * 128  + 132   = 35716
  2   item_allskills              [ 47485 * 2   / 1024 ] * 4096 + 15123 = 391955
  8   item_find_magic_perlevel    [ 47485 * 8   / 1024 ] * 1024 + 814   = 379694
  1   item_nonclassskill          [ 47485 * 1 * 640 / 1024 ] + 16000    = 45678
      (Teleport, skills.txt: mul=640 add=16000)
  5   item_maxhp_percent (Jah)    [ 47485 * 5   / 1024 ] * 204  + 32093 = 79217
  15  item_damagetomana (Ith)     [ 47485 * 15  / 1024 ] * 20   + 43    = 13943
  8   damageresist (Ber)          [ 47485 * 8   / 1024 ] * 68   + 152   = 25312

total = 47485 + 114870 + 11143 + 1898 + 359397 + 65579 + 329499 + 35716 +
        391955 + 379694 + 45678 + 79217 + 13943 + 25312                 = 1901386

scaled_repair_cost = [1901386 * (80-0) / 80] = 1901386
repair_cost = [ 1901386 / 1024 ] * 128 = 237568  (accurate in-game value: 237568)


Superior 15/15 Breast Plate Enigma (0 durability)
=================================================

scaled_armor_cost = [10078 * (68+1) / 68]                              = 10226
socketed_items_cost = [117740 / 2] + [2240 / 2] + [109760 / 2]         = 114870

bonus_costs:
  15  item_armor_percent          [ (68+1) * 15 / 100 ] = 10
                                  [ 10226 * 10 / 1024 ] * 20 + 47 = 2027
                                  [ 2027 / 2 ]                         = 1013
  15  item_maxdurability_percent  [ 50 * 15 / 100 ] = 7
                                  [ 10226 * 7 / 1024 ] * 10 + 117 = 807
                                  [ 807 * 10 / 25 ]                    = 322
  775 armorclass                  [ 10226 * 775 / 1024 ] * 10   + 17   = 77407
  14  item_healafterkill          [ 10226 * 14  / 1024 ] * 101  + 30   = 14069
  45  item_fastermovevelocity     [ 10226 * 45  / 1024 ] * 156  + 4083 = 74127
  6   item_strength_perlevel      [ 10226 * 6 * 128  / 1024 ] + 132    = 7801
  2   item_allskills              [ 10226 * 2 * 4096 / 1024 ] + 15123  = 96931
  8   item_find_magic_perlevel    [ 10226 * 8   / 1024 ] * 1024 + 814  = 81710
  1   item_nonclassskill          [ 10226 * 1 * 640  / 1024 ] + 16000  = 22391
      (Teleport, skills.txt: mul=640 add=16000)
  5   item_maxhp_percent (Jah)    [ 10226 * 5 * 204  / 1024 ] + 32093  = 42279
  15  item_damagetomana (Ith)     [ 10226 * 15  / 1024 ] * 20   + 43   = 3023
  8   damageresist (Ber)          [ 10226 * 8   / 1024 ] * 68   + 152  = 5524

total = 10226 + 114870 + 1013 + 322 + 77407 + 14069 + 74127 + 7801 +
        96931 + 81710 + 22391 + 42279 + 3023 + 5524                    = 551693

scaled_repair_cost = [551693 * (57-0) / 57] = 551693
repair_cost = [ 551693 / 1024 ] * 128 = 68864  (accurate in-game value: 68608)


Superior 15/15 Dusk Shroud Chains of Honor (0 durability)
=========================================================

scaled_armor_cost = [218357 * (467+1) / 467]                                 = 218824
socketed_items_cost = [20160 / 2] + [56000 / 2] + [109760 / 2] + [67760 / 2] = 126840

bonus_costs:
  15  item_armor_percent          [ (467+1) * 15 / 100 ] = 70
                                  [ 218824 * 70 / 1024 ] * 20 + 47 = 299207
                                  [ 299207 / 2 ]                             = 149603
  15  item_maxdurability_percent  [ 20 * 15 / 100 ] = 3
                                  [ 218824 * 3 / 1024 ] * 10 + 117 = 6527
                                  [ 6527 * 10 / 25 ]                         = 2610
  50  fireresist                  [ 218824 * 50  / 1024 ] * 20   + 43        = 213723
  50  lightresist                 [ 218824 * 50  / 1024 ] * 20   + 43        = 213723
  50  coldresist                  [ 218824 * 50  / 1024 ] * 20   + 43        = 213723
  50  poisonresist                [ 218824 * 50  / 1024 ] * 20   + 43        = 213723
  70  item_armor_percent          [ (467+1) * 70 / 100 ] = 327
                                  [ 218824 * 327 / 1024 ] * 20 + 47 = 1397607
                                  [ 1397607 / 2 ]                            = 698803
  200 item_demondamage_percent    [ 218824 * 200 / 1024 ] * 12   + 19        = 512887
  100 item_undeaddamage_percent   [ 218824 * 100 / 1024 ] * 12   + 13        = 256441
  8   lifedrainmindam             [ 218824 * 8   / 1024 ] * 341  + 1044      = 583813
  2   item_allskills              [ 218824 * 2   / 1024 ] * 4096 + 15123     = 1764115
  20  strength                    [ 218824 * 20  / 1024 ] * 55   + 125       = 235140
  7   hpregen (Dol)               [ 218824 * 7   / 1024 ] * 410  + 451       = 613401
  15  fireresist (Um)             [ 218824 * 15  / 1024 ] * 20   + 43        = 64143
  15  lightresist (Um)            [ 218824 * 15  / 1024 ] * 20   + 43        = 64143
  15  coldresist (Um)             [ 218824 * 15  / 1024 ] * 20   + 43        = 64143
  15  poisonresist (Um)           [ 218824 * 15  / 1024 ] * 20   + 43        = 64143
  8   damageresist (Ber)          [ 218824 * 8   / 1024 ] * 68   + 152       = 116364
  25  item_magicbonus (Ist)       [ 218824 * 25  / 1024 ] * 102  + 577       = 545461

total = 218824 + 126840 + 149603 + 2610 + 4 * 213723 + 698803 + 512887 +
        256441 + 583813 + 1764115 + 235140 + 613401 + 4 * 64143 +
        116364 + 545461                                                      = 6935766

scaled_repair_cost = [6935766 * (23-0) / 23] = 6935766
repair_cost = [ 6935766 / 1024 ] * 128 = 866944  (accurate in-game value: 866304)


Superior 15/15 Archon Plate Chains of Honor (0 durability)
==========================================================

scaled_armor_cost = [317526 * (524+1) / 524]                                 = 318131
socketed_items_cost = [20160 / 2] + [56000 / 2] + [109760 / 2] + [67760 / 2] = 126840

bonus_costs:
  15  item_armor_percent          [ (524+1) * 15 / 100 ] = 78
                                  [ 318131 * 78 / 1024 ] * 20 + 47 = 484687
                                  [ 484687 / 2 ]                             = 242343
  15  item_maxdurability_percent  [ 60 * 15 / 100 ] = 9
                                  [ 318131 * 9 / 1024 ] * 10 + 117 = 28077
                                  [ 28077 * 10 / 25 ]                        = 11230
  50  fireresist                  [ 318131 * 50  / 1024 ] * 20   + 43        = 310703
  50  lightresist                 [ 318131 * 50  / 1024 ] * 20   + 43        = 310703
  50  coldresist                  [ 318131 * 50  / 1024 ] * 20   + 43        = 310703
  50  poisonresist                [ 318131 * 50  / 1024 ] * 20   + 43        = 310703
  70  item_armor_percent          [ (524+1) * 70 / 100 ] = 367
                                  [ 318131 * 367 / 1024 ] * 20 + 47 = 2280387
                                  [ 2280387 / 2 ]                            = 1140193
  200 item_demondamage_percent    [ 318131 * 200 / 1024 ] * 12   + 19        = 745627
  100 item_undeaddamage_percent   [ 318131 * 100 / 1024 ] * 12   + 13        = 372817
  8   lifedrainmindam             [ 318131 * 8   / 1024 ] * 341  + 1044      = 848429
  2   item_allskills              [ 318131 * 2   / 1024 ] * 4096 + 15123     = 2558739
  20  strength                    [ 318131 * 20  / 1024 ] * 55   + 125       = 341840
  7   hpregen (Dol)               [ 318131 * 7   / 1024 ] * 410  + 451       = 891791
  15  fireresist (Um)             [ 318131 * 15  / 1024 ] * 20   + 43        = 93243
  15  lightresist (Um)            [ 318131 * 15  / 1024 ] * 20   + 43        = 93243
  15  coldresist (Um)             [ 318131 * 15  / 1024 ] * 20   + 43        = 93243
  15  poisonresist (Um)           [ 318131 * 15  / 1024 ] * 20   + 43        = 93243
  8   damageresist (Ber)          [ 318131 * 8   / 1024 ] * 68   + 152       = 169132
  25  item_magicbonus (Ist)       [ 318131 * 25  / 1024 ] * 102  + 577       = 792709

total = 318131 + 126840 + 242343 + 11230 + 4 * 310703 + 1140193 +
        745627 + 372817 + 848429 + 2558739 + 341840 + 891791 +
        4 * 93243 + 169132 + 792709                                          = 10175605

scaled_repair_cost = [10175605 * (69-0) / 69] = 10175605
repair_cost = [ 10175605 / 1024 ] * 128 = 1271936  (accurate in-game value: 1271808)


Superior 15/15 Dusk Shroud Dragon (0 durability)
================================================

scaled_armor_cost = [218357 * (467+1) / 467]                               = 218824
socketed_items_cost = [102060 / 2] + [94640 / 2] + [14000 / 2]             = 105350

bonus_costs:
  15  item_armor_percent             [ (467+1) * 15 / 100 ] = 70
                                     [ 218824 * 70 / 1024 ] * 20 + 47 = 299207
                                     [ 299207 / 2 ]                        = 149603
  15  item_maxdurability_percent     [ 20 * 15 / 100 ] = 3
                                     [ 218824 * 3 / 1024 ] * 10 + 117 = 6527
                                     [ 6527 * 10 / 25 ]                    = 2610
  360 armorclass                     [ 218824 * 360 / 1024 ] * 10  + 17    = 769317
  230 armorclass_vs_missile          [ 218824 * 230 / 1024 ] * 5   + 11    = 245756
  3   item_strength_perlevel         [ 218824 * 3   / 1024 ] * 128 + 132   = 82180
  15  item_skillonhit                [ 218824 * 15  / 1024 ] * 896 + 64000 = 2935680
      (Hydra, skills.txt: mul=896 add=64000)
  18  item_skillongethit             [ 218824 * 18  / 1024 ] * 896 + 64000 = 3510016
      (Venom, skills.txt: mul=896 add=64000)
  14  item_aura (Level 14 Holy Fire Aura When Equipped)                    = 0
  5   strength                       [ 218824 * 5   / 1024 ] * 55  + 125   = 58865
  5   dexterity                      [ 218824 * 5   / 1024 ] * 55  + 125   = 58865
  5   vitality                       [ 218824 * 5   / 1024 ] * 55  + 100   = 58840
  5   energy                         [ 218824 * 5   / 1024 ] * 55  + 100   = 58840
  5   item_maxmana_percent (Sur)     [ 218824 * 5   / 1024 ] * 204 + 56452 = 274324
  5   maxlightresist (Lo)            [ 218824 * 5   / 1024 ] * 256 + 584   = 273992
  7   normal_damage_reduction (Sol)  [ 218824 * 7   / 1024 ] * 200 + 188   = 299188

total = 218824 + 105350 + 149603 + 2610 + 769317 + 245756 + 82180 +
        2935680 + 3510016 + 2 * 58865 + 2 * 58840 + 274324 + 273992 +
        299188                                                             = 9102250

scaled_repair_cost = [9102250 * (23-0) / 23] = 9102250
repair_cost = [ 9102250 / 1024 ] * 128 = 1137664  (accurate in-game value: 1137664)


Superior 15/15 Archon Plate Dragon (0 durability)
=================================================

scaled_armor_cost = [317526 * (524+1) / 524]                               = 318131
socketed_items_cost = [102060 / 2] + [94640 / 2] + [14000 / 2]             = 105350

bonus_costs:
  15  item_armor_percent             [ (524+1) * 15 / 100 ] = 78
                                     [ 318131 * 78 / 1024 ] * 20 + 47 = 484687
                                     [ 484687 / 2 ]                        = 242343
  15  item_maxdurability_percent     [ 60 * 15 / 100 ] = 9
                                     [ 318131 * 9 / 1024 ] * 10 + 117 = 28077
                                     [ 28077 * 10 / 25 ]                   = 11230
  360 armorclass                     [ 318131 * 360 / 1024 ] * 10  + 17    = 1118437
  230 armorclass_vs_missile          [ 318131 * 230 / 1024 ] * 5   + 11    = 357286
  3   item_strength_perlevel         [ 318131 * 3   / 1024 ] * 128 + 132   = 119428
  15  item_skillonhit                [ 318131 * 15  / 1024 ] * 896 + 64000 = 4239360
      (Hydra, skills.txt: mul=896 add=64000)
  18  item_skillongethit             [ 318131 * 18  / 1024 ] * 896 + 64000 = 5074432
      (Venom, skills.txt: mul=896 add=64000)
  14  item_aura (Level 14 Holy Fire Aura When Equipped)                    = 0
  5   strength                       [ 318131 * 5   / 1024 ] * 55  + 125   = 85540
  5   dexterity                      [ 318131 * 5   / 1024 ] * 55  + 125   = 85540
  5   vitality                       [ 318131 * 5   / 1024 ] * 55  + 100   = 85515
  5   energy                         [ 318131 * 5   / 1024 ] * 55  + 100   = 85515
  5   item_maxmana_percent (Sur)     [ 318131 * 5   / 1024 ] * 204 + 56452 = 373264
  5   maxlightresist (Lo)            [ 318131 * 5   / 1024 ] * 256 + 584   = 398152
  7   normal_damage_reduction (Sol)  [ 318131 * 7   / 1024 ] * 200 + 188   = 434988

total = 318131 + 105350 + 242343 + 11230 + 1118437 + 357286 + 119428 +
        4239360 + 5074432 + 2 * 85540 + 2 * 85515 + 373264 + 398152 +
        434988                                                             = 13134511

scaled_repair_cost = [13134511 * (69-0) / 69] = 13134511
repair_cost = [ 13134511 / 1024 ] * 128 = 1641728  (accurate in-game value: 1641472)


Superior 15/15 Dusk Shroud Fortitude (0 durability)
===================================================

scaled_armor_cost = [218357 * (467+1) / 467]                               = 218824
socketed_items_cost = [560 / 2] + [14000 / 2] + [20160 / 2] + [94640 / 2]  = 64680

bonus_costs:
  15  item_armor_percent             [ (467+1) * 15 / 100 ] = 70
                                     [ 218824 * 70 / 1024 ] * 20 + 47 = 299207
                                     [ 299207 / 2 ]                        = 149603
  15  item_maxdurability_percent     [ 20 * 15 / 100 ] = 3
                                     [ 218824 * 3 / 1024 ] * 10 + 117 = 6527
                                     [ 6527 * 10 / 25 ]                    = 2610
  200 item_armor_percent             [ (467+1) * 200 / 100 ] = 936
                                     [ 218824 * 936 / 1024 ] * 20 + 47 = 4000407
                                     [ 4000407 / 2 ]                       = 2000203
  300 item_mindamage_percent         [ 218824 * 300 / 1024 ] * 20  + 45    = 1282205
  300 item_maxdamage_percent         [ 218824 * 300 / 1024 ] * 20  + 45    = 1282205
  25  item_fastercastrate            [ 218824 * 25  / 1024 ] * 156 + 3876  = 837228
  15  item_skillongethit             [ 218824 * 15  / 1024 ] * 768 + 32000 = 2493440
      (Chilling Armor, skills.txt: mul=768 add=32000)
  12  item_damagetomana              [ 218824 * 12  / 1024 ] * 20  + 43    = 51323
  12  item_hp_perlevel               [ 218824 * 12  / 1024 ] * 64  + 92    = 164188
  30  fireresist                     [ 218824 * 30  / 1024 ] * 20  + 43    = 128243
  30  lightresist                    [ 218824 * 30  / 1024 ] * 20  + 43    = 128243
  30  coldresist                     [ 218824 * 30  / 1024 ] * 20  + 43    = 128243
  30  poisonresist                   [ 218824 * 30  / 1024 ] * 20  + 43    = 128243
  1   item_lightradius (El)          [ 218824 * 1   / 1024 ] * 51  + 15    = 10878
  15  armorclass (El)                [ 218824 * 15  / 1024 ] * 10  + 17    = 32067
  7   normal_damage_reduction (Sol)  [ 218824 * 7   / 1024 ] * 200 + 188   = 299188
  7   hpregen (Dol)                  [ 218824 * 7   / 1024 ] * 410 + 451   = 613401
  5   maxlightresist (Lo)            [ 218824 * 5   / 1024 ] * 256 + 584   = 273992

total = 218824 + 64680 + 149603 + 2610 + 2000203 + 2 * 1282205 + 837228 +
        2493440 + 51323 + 164188 + 4 * 128243 + 10878 + 32067 + 299188 +
        613401 + 273992                                                    = 10289007

scaled_repair_cost = [10289007 * (23-0) / 23] = 10289007
repair_cost = [ 10289007 / 1024 ] * 128 = 1286016  (accurate in-game value: 1285120)


Superior 15/15 Archon Plate Fortitude (0 durability)
====================================================

scaled_armor_cost = [317526 * (524+1) / 524]                               = 318131
socketed_items_cost = [560 / 2] + [14000 / 2] + [20160 / 2] + [94640 / 2]  = 64680

bonus_costs:
  15  item_armor_percent             [ (524+1) * 15 / 100 ] = 78
                                     [ 318131 * 78 / 1024 ] * 20 + 47 = 484687
                                     [ 484687 / 2 ]                        = 242343
  15  item_maxdurability_percent     [ 60 * 15 / 100 ] = 9
                                     [ 318131 * 9 / 1024 ] * 10 + 117 = 28077
                                     [ 28077 * 10 / 25 ]                   = 11230
  200 item_armor_percent             [ (524+1) * 200 / 100 ] = 1050
                                     [ 318131 * 1050 / 1024 ] * 20 + 47 = 6524207
                                     [ 6524207 / 2 ]                       = 3262103
  300 item_mindamage_percent         [ 318131 * 300 / 1024 ] * 20  + 45    = 1864085
  300 item_maxdamage_percent         [ 318131 * 300 / 1024 ] * 20  + 45    = 1864085
  25  item_fastercastrate            [ 318131 * 25  / 1024 ] * 156 + 3876  = 1215372
  15  item_skillongethit             [ 318131 * 15  / 1024 ] * 768 + 32000 = 3610880
      (Chilling Armor, skills.txt: mul=768 add=32000)
  12  item_damagetomana              [ 318131 * 12  / 1024 ] * 20  + 43    = 74603
  12  item_hp_perlevel               [ 318131 * 12  / 1024 ] * 64  + 92    = 238684
  30  fireresist                     [ 318131 * 30  / 1024 ] * 20  + 43    = 186443
  30  lightresist                    [ 318131 * 30  / 1024 ] * 20  + 43    = 186443
  30  coldresist                     [ 318131 * 30  / 1024 ] * 20  + 43    = 186443
  30  poisonresist                   [ 318131 * 30  / 1024 ] * 20  + 43    = 186443
  1   item_lightradius (El)          [ 318131 * 1   / 1024 ] * 51  + 15    = 15825
  15  armorclass (El)                [ 318131 * 15  / 1024 ] * 10  + 17    = 46617
  7   normal_damage_reduction (Sol)  [ 318131 * 7   / 1024 ] * 200 + 188   = 434988
  7   hpregen (Dol)                  [ 318131 * 7   / 1024 ] * 410 + 451   = 891791
  5   maxlightresist (Lo)            [ 318131 * 5   / 1024 ] * 256 + 584   = 398152

total = 318131 + 64680 + 242343 + 11230 + 3262103 + 2 * 1864085 +
        1215372 + 3610880 + 74603 + 238684 + 4 * 186443 + 15825 +
        46617 + 434988 + 891791 + 398152                                   = 15299341

scaled_repair_cost = [15299341 * (69-0) / 69] = 15299341
repair_cost = [ 15299341 / 1024 ] * 128 = 1912320  (accurate in-game value: 1911808)


Superior 15/15 Dusk Shroud Duress (0 durability)
================================================

scaled_armor_cost = [218357 * (467+1) / 467]                              = 218824
socketed_items_cost = [16940 / 2] + [56000 / 2] + [8960 / 2]              = 40950

bonus_costs:
  15  item_armor_percent             [ (467+1) * 15 / 100 ] = 70
                                     [ 218824 * 70 / 1024 ] * 20 + 47 = 299207
                                     [ 299207 / 2 ]                       = 149603
  15  item_maxdurability_percent     [ 20 * 15 / 100 ] = 3
                                     [ 218824 * 3 / 1024 ] * 10 + 117 = 6527
                                     [ 6527 * 10 / 25 ]                   = 2610
  37  coldmindam                     [ 218824 * 37  / 1024 ] * 512 + 451  = 4048323
  133 coldmaxdam                     [ 218824 * 133 / 1024 ] * 340 + 128  = 9663268
  20  item_mindamage_percent         [ 218824 * 20  / 1024 ] * 20  + 45   = 85505
  20  item_maxdamage_percent         [ 218824 * 20  / 1024 ] * 20  + 45   = 85505
  200 item_armor_percent             [ (467+1) * 200 / 100 ] = 936
                                     [ 218824 * 936 / 1024 ] * 20 + 47 = 4000407
                                     [ 4000407 / 2 ]                      = 2000203
  20  item_fastergethitrate          [ 218824 * 20  / 1024 ] * 72  + 1065 = 308721
  33  item_openwounds                [ 218824 * 33  / 1024 ] * 10  + 23   = 70533
  15  item_crushingblow              [ 218824 * 15  / 1024 ] * 40  + 98   = 128298
  -20 item_staminadrainpct           [ 218824 * -20 * 20 / 1024 ] + 102   = -85376
  20  item_fastergethitrate (Shael)  [ 218824 * 20  / 1024 ] * 72  + 1065 = 308721
  15  fireresist (Um)                [ 218824 * 15  / 1024 ] * 20  + 43   = 64143
  15  lightresist (Um)               [ 218824 * 15  / 1024 ] * 20  + 43   = 64143
  15  coldresist (Um)                [ 218824 * 15  / 1024 ] * 20  + 43   = 64143
  15  poisonresist (Um)              [ 218824 * 15  / 1024 ] * 20  + 43   = 64143
  30  coldresist (Thul)              [ 218824 * 30  / 1024 ] * 20  + 43   = 128243

total = 218824 + 40950 + 149603 + 2610 + 4048323 + 9663268 + 2 * 85505 +
        2000203 + 308721 + 70533 + 128298 + -85376 + 308721 + 4 * 64143 +
        128243                                                            = 17410503

scaled_repair_cost = [17410503 * (23-0) / 23] = 17410503
repair_cost = [ 17410503 / 1024 ] * 128 = 2176256  (accurate in-game value: 2181120)


Superior 15/15 Archon Plate Duress (0 durability)
=================================================

scaled_armor_cost = [317526 * (524+1) / 524]                              = 318131
socketed_items_cost = [16940 / 2] + [56000 / 2] + [8960 / 2]              = 40950

bonus_costs:
  15  item_armor_percent             [ (524+1) * 15 / 100 ] = 78
                                     [ 318131 * 78 / 1024 ] * 20 + 47 = 484687
                                     [ 484687 / 2 ]                       = 242343
  15  item_maxdurability_percent     [ 60 * 15 / 100 ] = 9
                                     [ 318131 * 9 / 1024 ] * 10 + 117 = 28077
                                     [ 28077 * 10 / 25 ]                  = 11230
  37  coldmindam                     [ 318131 * 37  / 1024 ] * 512 + 451  = 5885379
  133 coldmaxdam                     [ 318131 * 133 / 1024 ] * 340 + 128  = 14048588
  20  item_mindamage_percent         [ 318131 * 20  / 1024 ] * 20  + 45   = 124305
  20  item_maxdamage_percent         [ 318131 * 20  / 1024 ] * 20  + 45   = 124305
  200 item_armor_percent             [ (524+1) * 200 / 100 ] = 1050
                                     [ 318131 * 1050 / 1024 ] * 20 + 47 = 6524207
                                     [ 6524207 / 2 ]                      = 3262103
  20  item_fastergethitrate          [ 318131 * 20  / 1024 ] * 72  + 1065 = 448401
  33  item_openwounds                [ 318131 * 33  / 1024 ] * 10  + 23   = 102543
  15  item_crushingblow              [ 318131 * 15  / 1024 ] * 40  + 98   = 186498
  -20 item_staminadrainpct           [ 318131 * -20 * 20 / 1024 ] + 102   = -124167
  20  item_fastergethitrate (Shael)  [ 318131 * 20  / 1024 ] * 72  + 1065 = 448401
  15  fireresist (Um)                [ 318131 * 15  / 1024 ] * 20  + 43   = 93243
  15  lightresist (Um)               [ 318131 * 15  / 1024 ] * 20  + 43   = 93243
  15  coldresist (Um)                [ 318131 * 15  / 1024 ] * 20  + 43   = 93243
  15  poisonresist (Um)              [ 318131 * 15  / 1024 ] * 20  + 43   = 93243
  30  coldresist (Thul)              [ 318131 * 30  / 1024 ] * 20  + 43   = 186443

total = 318131 + 40950 + 242343 + 11230 + 5885379 + 14048588 +
        2 * 124305 + 3262103 + 448401 + 102543 + 186498 + -124167 +
        448401 + 4 * 93243 + 186443                                       = 25678425

scaled_repair_cost = [25678425 * (69-0) / 69] = 25678425
repair_cost = [ 25678425 / 1024 ] * 128 = 3209728  (accurate in-game value: 3217408)


Superior 15/15 Dusk Shroud Hustle (0 durability)
================================================

scaled_armor_cost = [218357 * (467+1) / 467]                              = 218824
socketed_items_cost = [16940 / 2] + [35840 / 2] + [560 / 2]               = 26670

bonus_costs:
  15  item_armor_percent             [ (467+1) * 15 / 100 ] = 70
                                     [ 218824 * 70 / 1024 ] * 20 + 47 = 299207
                                     [ 299207 / 2 ]                       = 149603
  15  item_maxdurability_percent     [ 20 * 15 / 100 ] = 3
                                     [ 218824 * 3 / 1024 ] * 10 + 117 = 6527
                                     [ 6527 * 10 / 25 ]                   = 2610
  65  item_fastermovevelocity        [ 218824 * 65 / 1024 ] * 156 + 4083  = 2170923
  35  item_staminadrainpct           [ 218824 * 35 / 1024 ] * 20  + 102   = 149682
  40  item_fasterattackrate          [ 218824 * 40 / 1024 ] * 156 + 1042  = 1334374
  10  fireresist                     [ 218824 * 10 / 1024 ] * 20  + 43    = 42763
  10  lightresist                    [ 218824 * 10 / 1024 ] * 20  + 43    = 42763
  10  coldresist                     [ 218824 * 10 / 1024 ] * 20  + 43    = 42763
  10  poisonresist                   [ 218824 * 10 / 1024 ] * 20  + 43    = 42763
  6   item_nonclassskill             [ 218824 * 6  / 1024 ] * 768 + 32000 = 1016576
      (Evade, skills.txt: mul=768 add=32000)
  20  item_fastergethitrate (Shael)  [ 218824 * 20 / 1024 ] * 72  + 1065  = 308721
  10  dexterity (Ko)                 [ 218824 * 10 / 1024 ] * 55  + 125   = 117605
  15  item_staminadrainpct (Eld)     [ 218824 * 15 / 1024 ] * 20  + 102   = 64202

total = 218824 + 26670 + 149603 + 2610 + 2170923 + 149682 + 1334374 +
        4 * 42763 + 1016576 + 308721 + 117605 + 64202                     = 5730842

scaled_repair_cost = [5730842 * (23-0) / 23] = 5730842
repair_cost = [ 5730842 / 1024 ] * 128 = 716288  (accurate in-game value: 715776)


Superior 15/15 Archon Plate Hustle (0 durability)
=================================================

scaled_armor_cost = [317526 * (524+1) / 524]                              = 318131
socketed_items_cost = [16940 / 2] + [35840 / 2] + [560 / 2]               = 26670

bonus_costs:
  15  item_armor_percent             [ (524+1) * 15 / 100 ] = 78
                                     [ 318131 * 78 / 1024 ] * 20 + 47 = 484687
                                     [ 484687 / 2 ]                       = 242343
  15  item_maxdurability_percent     [ 60 * 15 / 100 ] = 9
                                     [ 318131 * 9 / 1024 ] * 10 + 117 = 28077
                                     [ 28077 * 10 / 25 ]                  = 11230
  65  item_fastermovevelocity        [ 318131 * 65 / 1024 ] * 156 + 4083  = 3154191
  35  item_staminadrainpct           [ 318131 * 35 / 1024 ] * 20  + 102   = 217562
  40  item_fasterattackrate          [ 318131 * 40 / 1024 ] * 156 + 1042  = 1939498
  10  fireresist                     [ 318131 * 10 / 1024 ] * 20  + 43    = 62163
  10  lightresist                    [ 318131 * 10 / 1024 ] * 20  + 43    = 62163
  10  coldresist                     [ 318131 * 10 / 1024 ] * 20  + 43    = 62163
  10  poisonresist                   [ 318131 * 10 / 1024 ] * 20  + 43    = 62163
  6   item_nonclassskill             [ 318131 * 6  / 1024 ] * 768 + 32000 = 1463552
      (Evade, skills.txt: mul=768 add=32000)
  20  item_fastergethitrate (Shael)  [ 318131 * 20 / 1024 ] * 72  + 1065  = 448401
  10  dexterity (Ko)                 [ 318131 * 10 / 1024 ] * 55  + 125   = 170955
  15  item_staminadrainpct (Eld)     [ 318131 * 15 / 1024 ] * 20  + 102   = 93302

total = 318131 + 26670 + 242343 + 11230 + 3154191 + 217562 + 1939498 +
        4 * 62163 + 1463552 + 448401 + 170955 + 93302                     = 8334487

scaled_repair_cost = [8334487 * (69-0) / 69] = 8334487
repair_cost = [ 8334487 / 1024 ] * 128 = 1041792  (accurate in-game value: 1041408)


Superior 15/15 Dusk Shroud Treachery (0 durability)
===================================================

scaled_armor_cost = [218357 * (467+1) / 467]                               = 218824
socketed_items_cost = [16940 / 2] + [8960 / 2] + [45360 / 2]               = 35630

bonus_costs:
  15  item_armor_percent             [ (467+1) * 15 / 100 ] = 70
                                     [ 218824 * 70 / 1024 ] * 20 + 47 = 299207
                                     [ 299207 / 2 ]                        = 149603
  15  item_maxdurability_percent     [ 20 * 15 / 100 ] = 3
                                     [ 218824 * 3 / 1024 ] * 10 + 117 = 6527
                                     [ 6527 * 10 / 25 ]                    = 2610
  15  item_skillonhit                [ 218824 * 15 / 1024 ] * 896  + 64000 = 2935680
      (Venom, skills.txt: mul=896 add=64000)
  15  item_skillongethit             [ 218824 * 15 / 1024 ] * 640  + 8000  = 2059200
      (Fade, skills.txt: mul=640 add=8000)
  2   item_addclassskills            [ 218824 * 2  / 1024 ] * 1560 + 49523 = 715643
  45  item_fasterattackrate          [ 218824 * 45 / 1024 ] * 156  + 1042  = 1501138
  20  item_fastergethitrate (Shael)  [ 218824 * 20 / 1024 ] * 72   + 1065  = 308721
  30  coldresist (Thul)              [ 218824 * 30 / 1024 ] * 20   + 43    = 128243
  50  item_goldbonus (Lem)           [ 218824 * 50 / 1024 ] * 34   + 187   = 363443

total = 218824 + 35630 + 149603 + 2610 + 2935680 + 2059200 + 715643 +
        1501138 + 308721 + 128243 + 363443                                 = 8418735

scaled_repair_cost = [8418735 * (23-0) / 23] = 8418735
repair_cost = [ 8418735 / 1024 ] * 128 = 1052288  (accurate in-game value: 1051648)


Superior 15/15 Archon Plate Treachery (0 durability)
====================================================

scaled_armor_cost = [317526 * (524+1) / 524]                               = 318131
socketed_items_cost = [16940 / 2] + [8960 / 2] + [45360 / 2]               = 35630

bonus_costs:
  15  item_armor_percent             [ (524+1) * 15 / 100 ] = 78
                                     [ 318131 * 78 / 1024 ] * 20 + 47 = 484687
                                     [ 484687 / 2 ]                        = 242343
  15  item_maxdurability_percent     [ 60 * 15 / 100 ] = 9
                                     [ 318131 * 9 / 1024 ] * 10 + 117 = 28077
                                     [ 28077 * 10 / 25 ]                   = 11230
  15  item_skillonhit                [ 318131 * 15 / 1024 ] * 896  + 64000 = 4239360
      (Venom, skills.txt: mul=896 add=64000)
  15  item_skillongethit             [ 318131 * 15 / 1024 ] * 640  + 8000  = 2990400
      (Fade, skills.txt: mul=640 add=8000)
  2   item_addclassskills            [ 318131 * 2  / 1024 ] * 1560 + 49523 = 1018283
  45  item_fasterattackrate          [ 318131 * 45 / 1024 ] * 156  + 1042  = 2181922
  20  item_fastergethitrate (Shael)  [ 318131 * 20 / 1024 ] * 72   + 1065  = 448401
  30  coldresist (Thul)              [ 318131 * 30 / 1024 ] * 20   + 43    = 186443
  50  item_goldbonus (Lem)           [ 318131 * 50 / 1024 ] * 34   + 187   = 528309

total = 318131 + 35630 + 242343 + 11230 + 4239360 + 2990400 + 1018283 +
        2181922 + 448401 + 186443 + 528309                                 = 12200452

scaled_repair_cost = [12200452 * (69-0) / 69] = 12200452
repair_cost = [ 12200452 / 1024 ] * 128 = 1524992  (accurate in-game value: 1524736)


Superior 15/15 Dusk Shroud Smoke (0 durability)
===============================================

scaled_armor_cost = [218357 * (467+1) / 467]                            = 218824
socketed_items_cost = [1260 / 2] + [31500 / 2]                          = 16380

bonus_costs:
  15  item_armor_percent           [ (467+1) * 15 / 100 ] = 70
                                   [ 218824 * 70 / 1024 ] * 20 + 47 = 299207
                                   [ 299207 / 2 ]                       = 149603
  15  item_maxdurability_percent   [ 20 * 15 / 100 ] = 3
                                   [ 218824 * 3 / 1024 ] * 10 + 117 = 6527
                                   [ 6527 * 10 / 25 ]                   = 2610
  250 armorclass_vs_missile        [ 218824 * 250 / 1024 ] * 5   + 11   = 267126
  75  item_armor_percent           [ (467+1) * 75 / 100 ] = 351
                                   [ 218824 * 351 / 1024 ] * 20 + 47 = 1500187
                                   [ 1500187 / 2 ]                      = 750093
  50  fireresist                   [ 218824 * 50  / 1024 ] * 20  + 43   = 213723
  50  lightresist                  [ 218824 * 50  / 1024 ] * 20  + 43   = 213723
  50  coldresist                   [ 218824 * 50  / 1024 ] * 20  + 43   = 213723
  50  poisonresist                 [ 218824 * 50  / 1024 ] * 20  + 43   = 213723
  20  item_fastergethitrate        [ 218824 * 20  / 1024 ] * 72  + 1065 = 308721
  -1  item_lightradius             [ 218824 * -1 * 51 / 1024 ] + 15     = -10883
  6   item_charged_skill           [ 218824 * 6   / 1024 ] * 384 + 3000 = 495288
      (Weaken, skills.txt: mul=384 add=3000)
  30  armorclass_vs_missile (Nef)  [ 218824 * 30  / 1024 ] * 5   + 11   = 32061
  10  energy (Lum)                 [ 218824 * 10  / 1024 ] * 55  + 100  = 117580

total = 218824 + 16380 + 149603 + 2610 + 267126 + 750093 +
        4 * 213723 + 308721 + -10883 + 495288 + 32061 + 117580          = 3202295

scaled_repair_cost = [3202295 * (23-0) / 23] = 3202295
repair_cost = [ 3202295 / 1024 ] * 128 = 400256  (accurate in-game value: 399360)


Superior 15/15 Archon Plate Smoke (0 durability)
================================================

scaled_armor_cost = [317526 * (524+1) / 524]                            = 318131
socketed_items_cost = [1260 / 2] + [31500 / 2]                          = 16380

bonus_costs:
  15  item_armor_percent           [ (524+1) * 15 / 100 ] = 78
                                   [ 318131 * 78 / 1024 ] * 20 + 47 = 484687
                                   [ 484687 / 2 ]                       = 242343
  15  item_maxdurability_percent   [ 60 * 15 / 100 ] = 9
                                   [ 318131 * 9 / 1024 ] * 10 + 117 = 28077
                                   [ 28077 * 10 / 25 ]                  = 11230
  250 armorclass_vs_missile        [ 318131 * 250 / 1024 ] * 5   + 11   = 388351
  75  item_armor_percent           [ (524+1) * 75 / 100 ] = 393
                                   [ 318131 * 393 / 1024 ] * 20 + 47 = 2441947
                                   [ 2441947 / 2 ]                      = 1220973
  50  fireresist                   [ 318131 * 50  / 1024 ] * 20  + 43   = 310703
  50  lightresist                  [ 318131 * 50  / 1024 ] * 20  + 43   = 310703
  50  coldresist                   [ 318131 * 50  / 1024 ] * 20  + 43   = 310703
  50  poisonresist                 [ 318131 * 50  / 1024 ] * 20  + 43   = 310703
  20  item_fastergethitrate        [ 318131 * 20  / 1024 ] * 72  + 1065 = 448401
  -1  item_lightradius             [ 318131 * -1 * 51 / 1024 ] + 15     = -15829
  6   item_charged_skill           [ 318131 * 6   / 1024 ] * 384 + 3000 = 718776
      (Weaken, skills.txt: mul=384 add=3000)
  30  armorclass_vs_missile (Nef)  [ 318131 * 30  / 1024 ] * 5   + 11   = 46611
  10  energy (Lum)                 [ 318131 * 10  / 1024 ] * 55  + 100  = 170930

total = 318131 + 16380 + 242343 + 11230 + 388351 + 1220973 +
        4 * 310703 + 448401 + -15829 + 718776 + 46611 + 170930          = 4809109

scaled_repair_cost = [4809109 * (69-0) / 69] = 4809109
repair_cost = [ 4809109 / 1024 ] * 128 = 601088  (accurate in-game value: 601088)


Superior 15/15 Mage Plate Smoke (0 durability)
==============================================

scaled_armor_cost = [118407 * (261+1) / 261]                            = 118860
socketed_items_cost = [1260 / 2] + [31500 / 2]                          = 16380

bonus_costs:
  15  item_armor_percent           [ (261+1) * 15 / 100 ] = 39
                                   [ 118860 * 39 / 1024 ] * 20 + 47 = 90567
                                   [ 90567 / 2 ]                        = 45283
  15  item_maxdurability_percent   [ 60 * 15 / 100 ] = 9
                                   [ 118860 * 9 / 1024 ] * 10 + 117 = 10557
                                   [ 10557 * 10 / 25 ]                  = 4222
  250 armorclass_vs_missile        [ 118860 * 250 / 1024 ] * 5   + 11   = 145101
  75  item_armor_percent           [ (261+1) * 75 / 100 ] = 196
                                   [ 118860 * 196 / 1024 ] * 20 + 47 = 455047
                                   [ 455047 / 2 ]                       = 227523
  50  fireresist                   [ 118860 * 50  / 1024 ] * 20  + 43   = 116103
  50  lightresist                  [ 118860 * 50  / 1024 ] * 20  + 43   = 116103
  50  coldresist                   [ 118860 * 50  / 1024 ] * 20  + 43   = 116103
  50  poisonresist                 [ 118860 * 50  / 1024 ] * 20  + 43   = 116103
  20  item_fastergethitrate        [ 118860 * 20  / 1024 ] * 72  + 1065 = 168177
  -1  item_lightradius             [ 118860 * -1 * 51 / 1024 ] + 15     = -5904
  6   item_charged_skill           [ 118860 * 6   / 1024 ] * 384 + 3000 = 270264
      (Weaken, skills.txt: mul=384 add=3000)
  30  armorclass_vs_missile (Nef)  [ 118860 * 30  / 1024 ] * 5   + 11   = 17421
  10  energy (Lum)                 [ 118860 * 10  / 1024 ] * 55  + 100  = 63900

total = 118860 + 16380 + 45283 + 4222 + 145101 + 227523 +
        4 * 116103 + 168177 + -5904 + 270264 + 17421 + 63900            = 1535639

scaled_repair_cost = [1535639 * (69-0) / 69] = 1535639
repair_cost = [ 1535639 / 1024 ] * 128 = 191872  (accurate in-game value: 191488)


Superior 15/15 Dusk Shroud Stealth (0 durability)
=================================================

scaled_armor_cost = [218357 * (467+1) / 467]                          = 218824
socketed_items_cost = [3500 / 2] + [2240 / 2]                         = 2870

bonus_costs:
  15  item_armor_percent          [ (467+1) * 15 / 100 ] = 70
                                  [ 218824 * 70 / 1024 ] * 20 + 47 = 299207
                                  [ 299207 / 2 ]                      = 149603
  15  item_maxdurability_percent  [ 20 * 15 / 100 ] = 3
                                  [ 218824 * 3 / 1024 ] * 10 + 117 = 6527
                                  [ 6527 * 10 / 25 ]                  = 2610
  3   magic_damage_reduction      [ 218824 * 3  / 1024 ] * 340 + 397  = 218337
  6   dexterity                   [ 218824 * 6  / 1024 ] * 55  + 125  = 70635
  15  maxstamina                  [ 218824 * 15 / 1024 ] * 20  + 75   = 64175
  25  item_fastermovevelocity     [ 218824 * 25 / 1024 ] * 156 + 4083 = 837435
  25  item_fastercastrate         [ 218824 * 25 / 1024 ] * 156 + 3876 = 837228
  25  item_fastergethitrate       [ 218824 * 25 / 1024 ] * 72  + 1065 = 385689
  30  poisonresist (Tal)          [ 218824 * 30 / 1024 ] * 20  + 43   = 128243
  15  manarecoverybonus (Eth)                                         = 0

total = 218824 + 2870 + 149603 + 2610 + 218337 + 70635 + 64175 +
        837435 + 837228 + 385689 + 128243                             = 2915649

scaled_repair_cost = [2915649 * (23-0) / 23] = 2915649
repair_cost = [ 2915649 / 1024 ] * 128 = 364416  (accurate in-game value: 363520)


Superior 15/15 Mage Plate Stealth (0 durability)
================================================

scaled_armor_cost = [118407 * (261+1) / 261]                          = 118860
socketed_items_cost = [3500 / 2] + [2240 / 2]                         = 2870

bonus_costs:
  15  item_armor_percent          [ (261+1) * 15 / 100 ] = 39
                                  [ 118860 * 39 / 1024 ] * 20 + 47 = 90567
                                  [ 90567 / 2 ]                       = 45283
  15  item_maxdurability_percent  [ 60 * 15 / 100 ] = 9
                                  [ 118860 * 9 / 1024 ] * 10 + 117 = 10557
                                  [ 10557 * 10 / 25 ]                 = 4222
  3   magic_damage_reduction      [ 118860 * 3  / 1024 ] * 340 + 397  = 118717
  6   dexterity                   [ 118860 * 6  / 1024 ] * 55  + 125  = 38405
  15  maxstamina                  [ 118860 * 15 / 1024 ] * 20  + 75   = 34895
  25  item_fastermovevelocity     [ 118860 * 25 / 1024 ] * 156 + 4083 = 456639
  25  item_fastercastrate         [ 118860 * 25 / 1024 ] * 156 + 3876 = 456432
  25  item_fastergethitrate       [ 118860 * 25 / 1024 ] * 72  + 1065 = 209937
  30  poisonresist (Tal)          [ 118860 * 30 / 1024 ] * 20  + 43   = 69683
  15  manarecoverybonus (Eth)                                         = 0

total = 118860 + 2870 + 45283 + 4222 + 118717 + 38405 + 34895 +
        456639 + 456432 + 209937 + 69683                              = 1555943

scaled_repair_cost = [1555943 * (69-0) / 69] = 1555943
repair_cost = [ 1555943 / 1024 ] * 128 = 194432  (accurate in-game value: 193536)


Superior 15/15 Breast Plate Stealth (0 durability)
=================================================

scaled_armor_cost = [10078 * (68+1) / 68]                            = 10226
socketed_items_cost = [3500 / 2] + [2240 / 2]                        = 2870

bonus_costs:
  15  item_armor_percent          [ (68+1) * 15 / 100 ] = 10
                                  [ 10226 * 10 / 1024 ] * 20 + 47 = 2027
                                  [ 2027 / 2 ]                       = 1013
  15  item_maxdurability_percent  [ 50 * 15 / 100 ] = 7
                                  [ 10226 * 7 / 1024 ] * 10 + 117 = 807
                                  [ 807 * 10 / 25 ]                  = 322
  3   magic_damage_reduction      [ 10226 * 3 * 340 / 1024 ] + 397   = 10583
  6   dexterity                   [ 10226 * 6 * 55  / 1024 ] + 125   = 3420
  15  maxstamina                  [ 10226 * 15 / 1024 ] * 20  + 75   = 3055
  25  item_fastermovevelocity     [ 10226 * 25 / 1024 ] * 156 + 4083 = 42927
  25  item_fastercastrate         [ 10226 * 25 / 1024 ] * 156 + 3876 = 42720
  25  item_fastergethitrate       [ 10226 * 25 / 1024 ] * 72  + 1065 = 18993
  30  poisonresist (Tal)          [ 10226 * 30 / 1024 ] * 20  + 43   = 6023
  15  manarecoverybonus (Eth)                                        = 0

total = 10226 + 2870 + 1013 + 322 + 10583 + 3420 + 3055 + 42927 +
        42720 + 18993 + 6023                                         = 142152

scaled_repair_cost = [142152 * (57-0) / 57] = 142152
repair_cost = [ 142152 / 1024 ] * 128 = 17664  (accurate in-game value: 17664)


Superior 15/15 45@ Sacred Targe Phoenix (0 durability)
======================================================

scaled_armor_cost = [72618 * (158+1) / 158]                                  = 73077
socketed_items_cost = [80640 / 2] + [80640 / 2] + [94640 / 2] + [117740 / 2] = 186830

bonus_costs:
  15  item_armor_percent          [ (158+1) * 15 / 100 ] = 23
                                  [ 73077 * 23 / 1024 ] * 20 + 47 = 32867
                                  [ 32867 / 2 ]                              = 16433
  15  item_maxdurability_percent  [ 45 * 15 / 100 ] = 6
                                  [ 73077 * 6 / 1024 ] * 10 + 117 = 4397
                                  [ 4397 * 10 / 25 ]                         = 1758
  45  fireresist (automod)        [ 73077 * 45  / 1024 ] * 20   + 43         = 64263
  45  lightresist (automod)       [ 73077 * 45  / 1024 ] * 20   + 43         = 64263
  45  coldresist (automod)        [ 73077 * 45  / 1024 ] * 20   + 43         = 64263
  45  poisonresist (automod)      [ 73077 * 45  / 1024 ] * 20   + 43         = 64263
  400 item_mindamage_percent      [ 73077 * 400 / 1024 ] * 20   + 45         = 570945
  400 item_maxdamage_percent      [ 73077 * 400 / 1024 ] * 20   + 45         = 570945
  400 armorclass_vs_missile       [ 73077 * 400 / 1024 ] * 5    + 11         = 142736
  22  item_skillonhit             [ 73077 * 22  / 1024 ] * 256  + 1000       = 402920
      (Firestorm, skills.txt: mul=256 add=1000)
  40  item_skillonlevelup         [ 73077 * 40  / 1024 ] * 512  + 8000       = 1469248
      (Blaze, skills.txt: mul=512 add=8000)
  28  passive_fire_pierce         [ 73077 * 28  / 1024 ] * 2578 + 0          = 5150844
  15  item_aura (Level 10-15 Redemption Aura When Equipped)                  = 0
  21  item_absorbfire             [ 73077 * 21  / 1024 ] * 204  + 1739       = 307331
  5   maxfireresist (Vex)         [ 73077 * 5   / 1024 ] * 256  + 584        = 91720
  5   maxfireresist (Vex)         [ 73077 * 5   / 1024 ] * 256  + 584        = 91720
  5   maxlightresist (Lo)         [ 73077 * 5   / 1024 ] * 256  + 584        = 91720
  50  maxhp (Jah)                 [ 73077 * 50  / 1024 ] * 20   + 56         = 71416

total = 73077 + 186830 + 16433 + 1758 + 4 * 64263 + 2 * 570945 +
        142736 + 402920 + 1469248 + 5150844 + 307331 + 3 * 91720 + 71416     = 9496695

scaled_repair_cost = [9496695 * (51-0) / 51] = 9496695
repair_cost = [ 9496695 / 1024 ] * 128 = 1187072  (accurate in-game value: 1186816)


Superior 15/15 Monarch Phoenix (0 durability)
=============================================

scaled_armor_cost = [81896 * (148+1) / 148]                                  = 82449
socketed_items_cost = [80640 / 2] + [80640 / 2] + [94640 / 2] + [117740 / 2] = 186830

bonus_costs:
  15  item_armor_percent          [ (148+1) * 15 / 100 ] = 22
                                  [ 82449 * 22 / 1024 ] * 20 + 47 = 35467
                                  [ 35467 / 2 ]                              = 17733
  15  item_maxdurability_percent  [ 86 * 15 / 100 ] = 12
                                  [ 82449 * 12 / 1024 ] * 10 + 117 = 9777
                                  [ 9777 * 10 / 25 ]                         = 3910
  400 item_mindamage_percent      [ 82449 * 400 / 1024 ] * 20   + 45         = 644165
  400 item_maxdamage_percent      [ 82449 * 400 / 1024 ] * 20   + 45         = 644165
  400 armorclass_vs_missile       [ 82449 * 400 / 1024 ] * 5    + 11         = 161041
  22  item_skillonhit             [ 82449 * 22  / 1024 ] * 256  + 1000       = 454376
      (Firestorm, skills.txt: mul=256 add=1000)
  40  item_skillonlevelup         [ 82449 * 40  / 1024 ] * 512  + 8000       = 1656640
      (Blaze, skills.txt: mul=512 add=8000)
  28  passive_fire_pierce         [ 82449 * 28  / 1024 ] * 2578 + 0          = 5810812
  15  item_aura (Level 10-15 Redemption Aura When Equipped)                  = 0
  21  item_absorbfire             [ 82449 * 21  / 1024 ] * 204  + 1739       = 346499
  5   maxfireresist (Vex)         [ 82449 * 5   / 1024 ] * 256  + 584        = 103496
  5   maxfireresist (Vex)         [ 82449 * 5   / 1024 ] * 256  + 584        = 103496
  5   maxlightresist (Lo)         [ 82449 * 5   / 1024 ] * 256  + 584        = 103496
  50  maxhp (Jah)                 [ 82449 * 50  / 1024 ] * 20   + 56         = 80556

total = 82449 + 186830 + 17733 + 3910 + 2 * 644165 + 161041 + 454376 +
        1656640 + 5810812 + 346499 + 3 * 103496 + 80556                      = 10399664

scaled_repair_cost = [10399664 * (98-0) / 98] = 10399664
repair_cost = [ 10399664 / 1024 ] * 128 = 1299840  (accurate in-game value: 1299456)


Superior 15/15 45@ Sacred Targe Dream (0 durability)
====================================================

scaled_armor_cost = [72618 * (158+1) / 158]                            = 73077
socketed_items_cost = [27440 / 2] + [117740 / 2] + [50540 / 2]         = 97860

bonus_costs:
  15  item_armor_percent          [ (158+1) * 15 / 100 ] = 23
                                  [ 73077 * 23 / 1024 ] * 20 + 47 = 32867
                                  [ 32867 / 2 ]                        = 16433
  15  item_maxdurability_percent  [ 45 * 15 / 100 ] = 6
                                  [ 73077 * 6 / 1024 ] * 10 + 117 = 4397
                                  [ 4397 * 10 / 25 ]                   = 1758
  45  fireresist (automod)        [ 73077 * 45  / 1024 ] * 20  + 43    = 64263
  45  lightresist (automod)       [ 73077 * 45  / 1024 ] * 20  + 43    = 64263
  45  coldresist (automod)        [ 73077 * 45  / 1024 ] * 20  + 43    = 64263
  45  poisonresist (automod)      [ 73077 * 45  / 1024 ] * 20  + 43    = 64263
  220 armorclass                  [ 73077 * 220 / 1024 ] * 10  + 17    = 157017
  15  item_skillongethit          [ 73077 * 15  / 1024 ] * 640 + 16000 = 700800
      (Confuse, skills.txt: mul=640 add=16000)
  5   item_mana_perlevel          [ 73077 * 5   / 1024 ] * 128 + 90    = 45658
  20  fireresist                  [ 73077 * 20  / 1024 ] * 20  + 43    = 28583
  20  lightresist                 [ 73077 * 20  / 1024 ] * 20  + 43    = 28583
  20  coldresist                  [ 73077 * 20  / 1024 ] * 20  + 43    = 28583
  20  poisonresist                [ 73077 * 20  / 1024 ] * 20  + 43    = 28583
  30  item_fastergethitrate       [ 73077 * 30  / 1024 ] * 72  + 1065  = 155145
  15  item_aura (Level 15 Holy Shock Aura When Equipped)               = 0
  25  item_magicbonus             [ 73077 * 25  / 1024 ] * 102 + 577   = 182545
  10  vitality (Io)               [ 73077 * 10  / 1024 ] * 55  + 100   = 39315
  50  maxhp (Jah)                 [ 73077 * 50  / 1024 ] * 20  + 56    = 71416
  30  item_armor_percent (Pul)    [ (158+1) * 30 / 100 ] = 47
                                  [ 73077 * 47 / 1024 ] * 20 + 47 = 67127
                                  [ 67127 / 2 ]                        = 33563

total = 73077 + 97860 + 16433 + 1758 + 4 * 64263 + 157017 + 700800 +
        45658 + 4 * 28583 + 155145 + 182545 + 39315 + 71416 + 33563    = 1945971

scaled_repair_cost = [1945971 * (51-0) / 51] = 1945971
repair_cost = [ 1945971 / 1024 ] * 128 = 243200  (accurate in-game value: 242688)


Superior 15/15 45@ Sacred Targe Dragon (0 durability)
=====================================================

scaled_armor_cost = [72618 * (158+1) / 158]                               = 73077
socketed_items_cost = [102060 / 2] + [94640 / 2] + [14000 / 2]            = 105350

bonus_costs:
  15  item_armor_percent             [ (158+1) * 15 / 100 ] = 23
                                     [ 73077 * 23 / 1024 ] * 20 + 47 = 32867
                                     [ 32867 / 2 ]                        = 16433
  15  item_maxdurability_percent     [ 45 * 15 / 100 ] = 6
                                     [ 73077 * 6 / 1024 ] * 10 + 117 = 4397
                                     [ 4397 * 10 / 25 ]                   = 1758
  45  fireresist (automod)           [ 73077 * 45  / 1024 ] * 20  + 43    = 64263
  45  lightresist (automod)          [ 73077 * 45  / 1024 ] * 20  + 43    = 64263
  45  coldresist (automod)           [ 73077 * 45  / 1024 ] * 20  + 43    = 64263
  45  poisonresist (automod)         [ 73077 * 45  / 1024 ] * 20  + 43    = 64263
  360 armorclass                     [ 73077 * 360 / 1024 ] * 10  + 17    = 256927
  230 armorclass_vs_missile          [ 73077 * 230 / 1024 ] * 5   + 11    = 82076
  3   item_strength_perlevel         [ 73077 * 3   / 1024 ] * 128 + 132   = 27524
  15  item_skillonhit                [ 73077 * 15  / 1024 ] * 896 + 64000 = 1022720
      (Hydra, skills.txt: mul=896 add=64000)
  18  item_skillongethit             [ 73077 * 18  / 1024 ] * 896 + 64000 = 1214464
      (Venom, skills.txt: mul=896 add=64000)
  14  item_aura (Level 14 Holy Fire Aura When Equipped)                   = 0
  5   strength                       [ 73077 * 5   / 1024 ] * 55  + 125   = 19705
  5   dexterity                      [ 73077 * 5   / 1024 ] * 55  + 125   = 19705
  5   vitality                       [ 73077 * 5   / 1024 ] * 55  + 100   = 19680
  5   energy                         [ 73077 * 5   / 1024 ] * 55  + 100   = 19680
  50  maxmana (Sur)                  [ 73077 * 50  / 1024 ] * 20  + 81    = 71441
  5   maxlightresist (Lo)            [ 73077 * 5   / 1024 ] * 256 + 584   = 91720
  7   normal_damage_reduction (Sol)  [ 73077 * 7   / 1024 ] * 200 + 188   = 99988

total = 73077 + 105350 + 16433 + 1758 + 4 * 64263 + 256927 + 82076 +
        27524 + 1022720 + 1214464 + 2 * 19705 + 2 * 19680 + 71441 +
        91720 + 99988                                                     = 3399300

scaled_repair_cost = [3399300 * (51-0) / 51] = 3399300
repair_cost = [ 3399300 / 1024 ] * 128 = 424832  (accurate in-game value: 423936)


Superior 15/15 45@ Sacred Targe Spirit (0 durability)
=====================================================

scaled_armor_cost = [72618 * (158+1) / 158]                                 = 73077
socketed_items_cost = [3500 / 2] + [8960 / 2] + [6860 / 2] + [11340 / 2]    = 15330

bonus_costs:
  15  item_armor_percent              [ (158+1) * 15 / 100 ] = 23
                                      [ 73077 * 23 / 1024 ] * 20 + 47 = 32867
                                      [ 32867 / 2 ]                         = 16433
  15  item_maxdurability_percent      [ 45 * 15 / 100 ] = 6
                                      [ 73077 * 6 / 1024 ] * 10 + 117 = 4397
                                      [ 4397 * 10 / 25 ]                    = 1758
  45  fireresist (automod)            [ 73077 * 45  / 1024 ] * 20   + 43    = 64263
  45  lightresist (automod)           [ 73077 * 45  / 1024 ] * 20   + 43    = 64263
  45  coldresist (automod)            [ 73077 * 45  / 1024 ] * 20   + 43    = 64263
  45  poisonresist (automod)          [ 73077 * 45  / 1024 ] * 20   + 43    = 64263
  55  item_fastergethitrate           [ 73077 * 55  / 1024 ] * 72   + 1065  = 283665
  112 maxmana                         [ 73077 * 112 / 1024 ] * 20   + 81    = 159921
  250 armorclass_vs_missile           [ 73077 * 250 / 1024 ] * 5    + 11    = 89216
  22  vitality                        [ 73077 * 22  / 1024 ] * 55   + 100   = 86450
  35  item_fastercastrate             [ 73077 * 35  / 1024 ] * 156  + 3876  = 393408
  8   item_absorbmagic                [ 73077 * 8   / 1024 ] * 204  + 1739  = 118019
  2   item_allskills                  [ 73077 * 2   / 1024 ] * 4096 + 15123 = 596755
  35  poisonresist (Tal)              [ 73077 * 35  / 1024 ] * 20   + 43    = 49983
  35  coldresist (Thul)               [ 73077 * 35  / 1024 ] * 20   + 43    = 49983
  35  lightresist (Ort)               [ 73077 * 35  / 1024 ] * 20   + 43    = 49983
  14  item_attackertakesdamage (Amn)  [ 73077 * 14  / 1024 ] * 128  + 112   = 127984

total = 73077 + 15330 + 16433 + 1758 + 4 * 64263 + 283665 + 159921 +
        89216 + 86450 + 393408 + 118019 + 596755 + 3 * 49983 + 127984       = 2369017

scaled_repair_cost = [2369017 * (51-0) / 51] = 2369017
repair_cost = [ 2369017 / 1024 ] * 128 = 296064  (accurate in-game value: 295936)


Superior 15/15 45@ Protector Shield Spirit (0 durability)
=========================================================

scaled_armor_cost = [52947 * (153+1) / 153]                                 = 53293
socketed_items_cost = [3500 / 2] + [8960 / 2] + [6860 / 2] + [11340 / 2]    = 15330

bonus_costs:
  15  item_armor_percent              [ (153+1) * 15 / 100 ] = 23
                                      [ 53293 * 23 / 1024 ] * 20 + 47 = 23987
                                      [ 23987 / 2 ]                         = 11993
  15  item_maxdurability_percent      [ 40 * 15 / 100 ] = 6
                                      [ 53293 * 6 / 1024 ] * 10 + 117 = 3237
                                      [ 3237 * 10 / 25 ]                    = 1294
  45  fireresist (automod)            [ 53293 * 45  / 1024 ] * 20   + 43    = 46863
  45  lightresist (automod)           [ 53293 * 45  / 1024 ] * 20   + 43    = 46863
  45  coldresist (automod)            [ 53293 * 45  / 1024 ] * 20   + 43    = 46863
  45  poisonresist (automod)          [ 53293 * 45  / 1024 ] * 20   + 43    = 46863
  55  item_fastergethitrate           [ 53293 * 55  / 1024 ] * 72   + 1065  = 207129
  112 maxmana                         [ 53293 * 112 / 1024 ] * 20   + 81    = 116641
  250 armorclass_vs_missile           [ 53293 * 250 / 1024 ] * 5    + 11    = 65061
  22  vitality                        [ 53293 * 22  / 1024 ] * 55   + 100   = 63020
  35  item_fastercastrate             [ 53293 * 35  / 1024 ] * 156  + 3876  = 287952
  8   item_absorbmagic                [ 53293 * 8   / 1024 ] * 204  + 1739  = 86603
  2   item_allskills                  [ 53293 * 2   / 1024 ] * 4096 + 15123 = 441107
  35  poisonresist (Tal)              [ 53293 * 35  / 1024 ] * 20   + 43    = 36463
  35  coldresist (Thul)               [ 53293 * 35  / 1024 ] * 20   + 43    = 36463
  35  lightresist (Ort)               [ 53293 * 35  / 1024 ] * 20   + 43    = 36463
  14  item_attackertakesdamage (Amn)  [ 53293 * 14  / 1024 ] * 128  + 112   = 93296

total = 53293 + 15330 + 11993 + 1294 + 4 * 46863 + 207129 + 116641 +
        65061 + 63020 + 287952 + 86603 + 441107 + 3 * 36463 + 93296         = 1739560

scaled_repair_cost = [1739560 * (46-0) / 46] = 1739560
repair_cost = [ 1739560 / 1024 ] * 128 = 217344  (accurate in-game value: 217088)


Superior 15/15 Monarch Spirit (0 durability)
============================================

scaled_armor_cost = [81896 * (148+1) / 148]                                 = 82449
socketed_items_cost = [3500 / 2] + [8960 / 2] + [6860 / 2] + [11340 / 2]    = 15330

bonus_costs:
  15  item_armor_percent              [ (148+1) * 15 / 100 ] = 22
                                      [ 82449 * 22 / 1024 ] * 20 + 47 = 35467
                                      [ 35467 / 2 ]                         = 17733
  15  item_maxdurability_percent      [ 86 * 15 / 100 ] = 12
                                      [ 82449 * 12 / 1024 ] * 10 + 117 = 9777
                                      [ 9777 * 10 / 25 ]                    = 3910
  55  item_fastergethitrate           [ 82449 * 55  / 1024 ] * 72   + 1065  = 319881
  112 maxmana                         [ 82449 * 112 / 1024 ] * 20   + 81    = 180421
  250 armorclass_vs_missile           [ 82449 * 250 / 1024 ] * 5    + 11    = 100656
  22  vitality                        [ 82449 * 22  / 1024 ] * 55   + 100   = 97505
  35  item_fastercastrate             [ 82449 * 35  / 1024 ] * 156  + 3876  = 443484
  8   item_absorbmagic                [ 82449 * 8   / 1024 ] * 204  + 1739  = 133115
  2   item_allskills                  [ 82449 * 2   / 1024 ] * 4096 + 15123 = 674579
  35  poisonresist (Tal)              [ 82449 * 35  / 1024 ] * 20   + 43    = 56403
  35  coldresist (Thul)               [ 82449 * 35  / 1024 ] * 20   + 43    = 56403
  35  lightresist (Ort)               [ 82449 * 35  / 1024 ] * 20   + 43    = 56403
  14  item_attackertakesdamage (Amn)  [ 82449 * 14  / 1024 ] * 128  + 112   = 144368

total = 82449 + 15330 + 17733 + 3910 + 319881 + 180421 + 100656 +
        97505 + 443484 + 133115 + 674579 + 3 * 56403 + 144368               = 2382640

scaled_repair_cost = [2382640 * (98-0) / 98] = 2382640
repair_cost = [ 2382640 / 1024 ] * 128 = 297728  (accurate in-game value: 296960)


Superior 15/15 45@ Sacred Targe Ancient's Pledge (0 durability)
===============================================================

scaled_armor_cost = [72618 * (158+1) / 158]                       = 73077
socketed_items_cost = [5040 / 2] + [6860 / 2] + [3500 / 2]        = 7700

bonus_costs:
  15  item_armor_percent          [ (158+1) * 15 / 100 ] = 23
                                  [ 73077 * 23 / 1024 ] * 20 + 47 = 32867
                                  [ 32867 / 2 ]                   = 16433
  15  item_maxdurability_percent  [ 45 * 15 / 100 ] = 6
                                  [ 73077 * 6 / 1024 ] * 10 + 117 = 4397
                                  [ 4397 * 10 / 25 ]              = 1758
  45  fireresist (automod)        [ 73077 * 45 / 1024 ] * 20 + 43 = 64263
  45  lightresist (automod)       [ 73077 * 45 / 1024 ] * 20 + 43 = 64263
  45  coldresist (automod)        [ 73077 * 45 / 1024 ] * 20 + 43 = 64263
  45  poisonresist (automod)      [ 73077 * 45 / 1024 ] * 20 + 43 = 64263
  30  coldresist                  [ 73077 * 30 / 1024 ] * 20 + 43 = 42843
  13  fireresist                  [ 73077 * 13 / 1024 ] * 20 + 43 = 18583
  13  lightresist                 [ 73077 * 13 / 1024 ] * 20 + 43 = 18583
  13  coldresist                  [ 73077 * 13 / 1024 ] * 20 + 43 = 18583
  13  poisonresist                [ 73077 * 13 / 1024 ] * 20 + 43 = 18583
  50  item_armor_percent          [ (158+1) * 50 / 100 ] = 79
                                  [ 73077 * 79 / 1024 ] * 20 + 47 = 112787
                                  [ 112787 / 2 ]                  = 56393
  10  item_damagetomana           [ 73077 * 10 / 1024 ] * 20 + 43 = 14303
  35  fireresist (Ral)            [ 73077 * 35 / 1024 ] * 20 + 43 = 49983
  35  lightresist (Ort)           [ 73077 * 35 / 1024 ] * 20 + 43 = 49983
  35  poisonresist (Tal)          [ 73077 * 35 / 1024 ] * 20 + 43 = 49983

total = 73077 + 7700 + 16433 + 1758 + 4 * 64263 + 42843 + 4 * 18583 +
        56393 + 14303 + 3 * 49983                                 = 693840

scaled_repair_cost = [693840 * (51-0) / 51] = 693840
repair_cost = [ 693840 / 1024 ] * 128 = 86656  (accurate in-game value: 86016)


Superior 15/15 45@ Protector Shield Ancient's Pledge (0 durability)
===================================================================

scaled_armor_cost = [52947 * (153+1) / 153]                       = 53293
socketed_items_cost = [5040 / 2] + [6860 / 2] + [3500 / 2]        = 7700

bonus_costs:
  15  item_armor_percent          [ (153+1) * 15 / 100 ] = 23
                                  [ 53293 * 23 / 1024 ] * 20 + 47 = 23987
                                  [ 23987 / 2 ]                   = 11993
  15  item_maxdurability_percent  [ 40 * 15 / 100 ] = 6
                                  [ 53293 * 6 / 1024 ] * 10 + 117 = 3237
                                  [ 3237 * 10 / 25 ]              = 1294
  45  fireresist (automod)        [ 53293 * 45 / 1024 ] * 20 + 43 = 46863
  45  lightresist (automod)       [ 53293 * 45 / 1024 ] * 20 + 43 = 46863
  45  coldresist (automod)        [ 53293 * 45 / 1024 ] * 20 + 43 = 46863
  45  poisonresist (automod)      [ 53293 * 45 / 1024 ] * 20 + 43 = 46863
  30  coldresist                  [ 53293 * 30 / 1024 ] * 20 + 43 = 31263
  13  fireresist                  [ 53293 * 13 / 1024 ] * 20 + 43 = 13563
  13  lightresist                 [ 53293 * 13 / 1024 ] * 20 + 43 = 13563
  13  coldresist                  [ 53293 * 13 / 1024 ] * 20 + 43 = 13563
  13  poisonresist                [ 53293 * 13 / 1024 ] * 20 + 43 = 13563
  50  item_armor_percent          [ (153+1) * 50 / 100 ] = 77
                                  [ 53293 * 77 / 1024 ] * 20 + 47 = 80187
                                  [ 80187 / 2 ]                   = 40093
  10  item_damagetomana           [ 53293 * 10 / 1024 ] * 20 + 43 = 10443
  35  fireresist (Ral)            [ 53293 * 35 / 1024 ] * 20 + 43 = 36463
  35  lightresist (Ort)           [ 53293 * 35 / 1024 ] * 20 + 43 = 36463
  35  poisonresist (Tal)          [ 53293 * 35 / 1024 ] * 20 + 43 = 36463

total = 53293 + 7700 + 11993 + 1294 + 4 * 46863 + 31263 + 4 * 13563 +
        40093 + 10443 + 3 * 36463                                 = 507172

scaled_repair_cost = [507172 * (46-0) / 46] = 507172
repair_cost = [ 507172 / 1024 ] * 128 = 63360  (accurate in-game value: 63360)


Superior 15/15 Large Shield Ancient's Pledge (0 durability)
===========================================================

scaled_armor_cost = [1214 * (14+1) / 14]                            = 1300
socketed_items_cost = [5040 / 2] + [6860 / 2] + [3500 / 2]          = 7700

bonus_costs:
  15  item_armor_percent          [ (14+1) * 15 / 100 ] = 2
                                  [ 1300 * 2 * 20 / 1024 ] + 47 = 97
                                  [ 97 / 2 ]                        = 48
  15  item_maxdurability_percent  [ 24 * 15 / 100 ] = 3
                                  [ 1300 * 3 * 10 / 1024 ] + 117 = 155
                                  [ 155 * 10 / 25 ]                 = 62
  30  coldresist                  [ 1300 * 30 * 20 / 1024 ] + 43    = 804
  13  fireresist                  [ 1300 * 13 * 20 / 1024 ] + 43    = 373
  13  lightresist                 [ 1300 * 13 * 20 / 1024 ] + 43    = 373
  13  coldresist                  [ 1300 * 13 * 20 / 1024 ] + 43    = 373
  13  poisonresist                [ 1300 * 13 * 20 / 1024 ] + 43    = 373
  50  item_armor_percent          [ (14+1) * 50 / 100 ] = 7
                                  [ 1300 * 7 * 20 / 1024 ] + 47 = 224
                                  [ 224 / 2 ]                       = 112
  10  item_damagetomana           [ 1300 * 10 * 20 / 1024 ] + 43    = 296
  35  fireresist (Ral)            [ 1300 * 35 * 20 / 1024 ] + 43    = 931
  35  lightresist (Ort)           [ 1300 * 35 * 20 / 1024 ] + 43    = 931
  35  poisonresist (Tal)          [ 1300 * 35 * 20 / 1024 ] + 43    = 931

total = 1300 + 7700 + 48 + 62 + 804 + 4 * 373 + 112 + 296 + 3 * 931 = 14607

scaled_repair_cost = [14607 * (27-0) / 27] = 14607
repair_cost = [ 14607 * 128 / 1024 ] = 1825  (accurate in-game value: 1796)


Superior 15/15 Bone Visage Dream (0 durability)
===============================================

scaled_armor_cost = [87274 * (157+1) / 157]                            = 87829
socketed_items_cost = [27440 / 2] + [117740 / 2] + [50540 / 2]         = 97860

bonus_costs:
  15  item_armor_percent          [ (157+1) * 15 / 100 ] = 23
                                  [ 87829 * 23 / 1024 ] * 20 + 47 = 39487
                                  [ 39487 / 2 ]                        = 19743
  15  item_maxdurability_percent  [ 40 * 15 / 100 ] = 6
                                  [ 87829 * 6 / 1024 ] * 10 + 117 = 5257
                                  [ 5257 * 10 / 25 ]                   = 2102
  220 armorclass                  [ 87829 * 220 / 1024 ] * 10  + 17    = 188707
  15  item_skillongethit          [ 87829 * 15  / 1024 ] * 640 + 16000 = 839040
      (Confuse, skills.txt: mul=640 add=16000)
  5   item_mana_perlevel          [ 87829 * 5   / 1024 ] * 128 + 90    = 54874
  20  fireresist                  [ 87829 * 20  / 1024 ] * 20  + 43    = 34343
  20  lightresist                 [ 87829 * 20  / 1024 ] * 20  + 43    = 34343
  20  coldresist                  [ 87829 * 20  / 1024 ] * 20  + 43    = 34343
  20  poisonresist                [ 87829 * 20  / 1024 ] * 20  + 43    = 34343
  30  item_fastergethitrate       [ 87829 * 30  / 1024 ] * 72  + 1065  = 186321
  15  item_aura (Level 15 Holy Shock Aura When Equipped)               = 0
  25  item_magicbonus             [ 87829 * 25  / 1024 ] * 102 + 577   = 219265
  10  vitality (Io)               [ 87829 * 10  / 1024 ] * 55  + 100   = 47235
  5   item_maxhp_percent (Jah)    [ 87829 * 5   / 1024 ] * 204 + 32093 = 119405
  30  item_armor_percent (Pul)    [ (157+1) * 30 / 100 ] = 47
                                  [ 87829 * 47 / 1024 ] * 20 + 47 = 80667
                                  [ 80667 / 2 ]                        = 40333

total = 87829 + 97860 + 19743 + 2102 + 188707 + 839040 + 54874 +
        4 * 34343 + 186321 + 219265 + 47235 + 119405 + 40333           = 2040086

scaled_repair_cost = [2040086 * (46-0) / 46] = 2040086
repair_cost = [ 2040086 / 1024 ] * 128 = 254976  (accurate in-game value: 254976)


Superior 15/15 Diadem Dream (0 durability)
==========================================

scaled_armor_cost = [58000 * (60+1) / 60]                              = 58966
socketed_items_cost = [27440 / 2] + [117740 / 2] + [50540 / 2]         = 97860

bonus_costs:
  15  item_armor_percent          [ (60+1) * 15 / 100 ] = 9
                                  [ 58966 * 9 / 1024 ] * 20 + 47 = 10407
                                  [ 10407 / 2 ]                        = 5203
  15  item_maxdurability_percent  [ 20 * 15 / 100 ] = 3
                                  [ 58966 * 3 / 1024 ] * 10 + 117 = 1837
                                  [ 1837 * 10 / 25 ]                   = 734
  220 armorclass                  [ 58966 * 220 / 1024 ] * 10  + 17    = 126697
  15  item_skillongethit          [ 58966 * 15  / 1024 ] * 640 + 16000 = 568320
      (Confuse, skills.txt: mul=640 add=16000)
  5   item_mana_perlevel          [ 58966 * 5   / 1024 ] * 128 + 90    = 36826
  20  fireresist                  [ 58966 * 20  / 1024 ] * 20  + 43    = 23063
  20  lightresist                 [ 58966 * 20  / 1024 ] * 20  + 43    = 23063
  20  coldresist                  [ 58966 * 20  / 1024 ] * 20  + 43    = 23063
  20  poisonresist                [ 58966 * 20  / 1024 ] * 20  + 43    = 23063
  30  item_fastergethitrate       [ 58966 * 30  / 1024 ] * 72  + 1065  = 125409
  15  item_aura (Level 15 Holy Shock Aura When Equipped)               = 0
  25  item_magicbonus             [ 58966 * 25  / 1024 ] * 102 + 577   = 147355
  10  vitality (Io)               [ 58966 * 10  / 1024 ] * 55  + 100   = 31725
  5   item_maxhp_percent (Jah)    [ 58966 * 5   / 1024 ] * 204 + 32093 = 90641
  30  item_armor_percent (Pul)    [ (60+1) * 30 / 100 ] = 18
                                  [ 58966 * 18 / 1024 ] * 20 + 47 = 20767
                                  [ 20767 / 2 ]                        = 10383

total = 58966 + 97860 + 5203 + 734 + 126697 + 568320 + 36826 +
        4 * 23063 + 125409 + 147355 + 31725 + 90641 + 10383            = 1392371

scaled_repair_cost = [1392371 * (23-0) / 23] = 1392371
repair_cost = [ 1392371 / 1024 ] * 128 = 173952  (accurate in-game value: 173056)


Superior 15/15 Great Helm Dream (0 durability)
==============================================

scaled_armor_cost = [6177 * (35+1) / 35]                              = 6353
socketed_items_cost = [27440 / 2] + [117740 / 2] + [50540 / 2]        = 97860

bonus_costs:
  15  item_armor_percent          [ (35+1) * 15 / 100 ] = 5
                                  [ 6353 * 5 * 20 / 1024 ] + 47 = 667
                                  [ 667 / 2 ]                         = 333
  15  item_maxdurability_percent  [ 40 * 15 / 100 ] = 6
                                  [ 6353 * 6 * 10 / 1024 ] + 117 = 489
                                  [ 489 * 10 / 25 ]                   = 195
  220 armorclass                  [ 6353 * 220 / 1024 ] * 10  + 17    = 13657
  15  item_skillongethit          [ 6353 * 15  / 1024 ] * 640 + 16000 = 75520
      (Confuse, skills.txt: mul=640 add=16000)
  5   item_mana_perlevel          [ 6353 * 5  * 128 / 1024 ] + 90     = 4060
  20  fireresist                  [ 6353 * 20  / 1024 ] * 20  + 43    = 2523
  20  lightresist                 [ 6353 * 20  / 1024 ] * 20  + 43    = 2523
  20  coldresist                  [ 6353 * 20  / 1024 ] * 20  + 43    = 2523
  20  poisonresist                [ 6353 * 20  / 1024 ] * 20  + 43    = 2523
  30  item_fastergethitrate       [ 6353 * 30  / 1024 ] * 72  + 1065  = 14457
  15  item_aura (Level 15 Holy Shock Aura When Equipped)              = 0
  25  item_magicbonus             [ 6353 * 25  / 1024 ] * 102 + 577   = 16387
  10  vitality (Io)               [ 6353 * 10 * 55  / 1024 ] + 100    = 3512
  5   item_maxhp_percent (Jah)    [ 6353 * 5  * 204 / 1024 ] + 32093  = 38421
  30  item_armor_percent (Pul)    [ (35+1) * 30 / 100 ] = 10
                                  [ 6353 * 10 * 20 / 1024 ] + 47 = 1287
                                  [ 1287 / 2 ]                        = 643

total = 6353 + 97860 + 333 + 195 + 13657 + 75520 + 4060 + 4 * 2523 +
        14457 + 16387 + 3512 + 38421 + 643                            = 281490

scaled_repair_cost = [281490 * (46-0) / 46] = 281490
repair_cost = [ 281490 / 1024 ] * 128 = 35072  (accurate in-game value: 35072)


Superior 15/15 Bone Visage Flickering Flame (0 durability)
==========================================================

scaled_armor_cost = [87274 * (157+1) / 157]                            = 87829
socketed_items_cost = [1260 / 2] + [50540 / 2] + [80640 / 2]           = 66220

bonus_costs:
  15  item_armor_percent           [ (157+1) * 15 / 100 ] = 23
                                   [ 87829 * 23 / 1024 ] * 20 + 47 = 39487
                                   [ 39487 / 2 ]                       = 19743
  15  item_maxdurability_percent   [ 40 * 15 / 100 ] = 6
                                   [ 87829 * 6 / 1024 ] * 10 + 117 = 5257
                                   [ 5257 * 10 / 25 ]                  = 2102
  3   item_elemskill               [ 87829 * 3  / 1024 ] * 1024 + 76   = 263244
  8   item_aura (Level 4-8 Resist Fire Aura When Equipped)             = 0
  15  passive_fire_pierce          [ 87829 * 15 / 1024 ] * 2578 + 0    = 3315308
  75  maxmana                      [ 87829 * 75 / 1024 ] * 20   + 81   = 128721
  1   item_halffreezeduration      [ 87829 * 1  / 1024 ] * 988  + 5096 = 89076
  50  item_poisonlengthresist      [ 87829 * 50 / 1024 ] * 10   + 27   = 42907
  30  armorclass_vs_missile (Nef)  [ 87829 * 30 / 1024 ] * 5    + 11   = 12876
  30  item_armor_percent (Pul)     [ (157+1) * 30 / 100 ] = 47
                                   [ 87829 * 47 / 1024 ] * 20 + 47 = 80667
                                   [ 80667 / 2 ]                       = 40333
  5   maxfireresist (Vex)          [ 87829 * 5  / 1024 ] * 256  + 584  = 110152

total = 87829 + 66220 + 19743 + 2102 + 263244 + 3315308 + 128721 +
        89076 + 42907 + 12876 + 40333 + 110152                         = 4178511

scaled_repair_cost = [4178511 * (46-0) / 46] = 4178511
repair_cost = [ 4178511 / 1024 ] * 128 = 522240  (accurate in-game value: 522240)


Superior 15/15 Diadem Flickering Flame (0 durability)
=====================================================

scaled_armor_cost = [58000 * (60+1) / 60]                             = 58966
socketed_items_cost = [1260 / 2] + [50540 / 2] + [80640 / 2]          = 66220

bonus_costs:
  15  item_armor_percent           [ (60+1) * 15 / 100 ] = 9
                                   [ 58966 * 9 / 1024 ] * 20 + 47 = 10407
                                   [ 10407 / 2 ]                      = 5203
  15  item_maxdurability_percent   [ 20 * 15 / 100 ] = 3
                                   [ 58966 * 3 / 1024 ] * 10 + 117 = 1837
                                   [ 1837 * 10 / 25 ]                 = 734
  3   item_elemskill               [ 58966 * 3  / 1024 ] * 1024 + 76  = 176204
  8   item_aura (Level 4-8 Resist Fire Aura When Equipped)            = 0
  15  passive_fire_pierce          [ 58966 * 15 / 1024 ] * 2578 + 0   = 2224814
  75  maxmana                      [ 58966 * 75 / 1024 ] * 20   + 81  = 86441
  1   item_halffreezeduration      [ 58966 * 1 * 988 / 1024 ] + 5096  = 61988
  50  item_poisonlengthresist      [ 58966 * 50 / 1024 ] * 10   + 27  = 28817
  30  armorclass_vs_missile (Nef)  [ 58966 * 30 / 1024 ] * 5    + 11  = 8646
  30  item_armor_percent (Pul)     [ (60+1) * 30 / 100 ] = 18
                                   [ 58966 * 18 / 1024 ] * 20 + 47 = 20767
                                   [ 20767 / 2 ]                      = 10383
  5   maxfireresist (Vex)          [ 58966 * 5  / 1024 ] * 256  + 584 = 74056

total = 58966 + 66220 + 5203 + 734 + 176204 + 2224814 + 86441 + 61988 +
        28817 + 8646 + 10383 + 74056                                  = 2802472

scaled_repair_cost = [2802472 * (23-0) / 23] = 2802472
repair_cost = [ 2802472 / 1024 ] * 128 = 350208  (accurate in-game value: 350208)


Superior 15/15 Great Helm Flickering Flame (0 durability)
=========================================================

scaled_armor_cost = [6177 * (35+1) / 35]                             = 6353
socketed_items_cost = [1260 / 2] + [50540 / 2] + [80640 / 2]         = 66220

bonus_costs:
  15  item_armor_percent           [ (35+1) * 15 / 100 ] = 5
                                   [ 6353 * 5 * 20 / 1024 ] + 47 = 667
                                   [ 667 / 2 ]                       = 333
  15  item_maxdurability_percent   [ 40 * 15 / 100 ] = 6
                                   [ 6353 * 6 * 10 / 1024 ] + 117 = 489
                                   [ 489 * 10 / 25 ]                 = 195
  3   item_elemskill               [ 6353 * 3 * 1024 / 1024 ] + 76   = 19135
  8   item_aura (Level 4-8 Resist Fire Aura When Equipped)           = 0
  15  passive_fire_pierce          [ 6353 * 15 / 1024 ] * 2578 + 0   = 239754
  75  maxmana                      [ 6353 * 75 / 1024 ] * 20   + 81  = 9381
  1   item_halffreezeduration      [ 6353 * 1 * 988  / 1024 ] + 5096 = 11225
  50  item_poisonlengthresist      [ 6353 * 50 / 1024 ] * 10   + 27  = 3127
  30  armorclass_vs_missile (Nef)  [ 6353 * 30 / 1024 ] * 5    + 11  = 941
  30  item_armor_percent (Pul)     [ (35+1) * 30 / 100 ] = 10
                                   [ 6353 * 10 * 20 / 1024 ] + 47 = 1287
                                   [ 1287 / 2 ]                      = 643
  5   maxfireresist (Vex)          [ 6353 * 5 * 256  / 1024 ] + 584  = 8525

total = 6353 + 66220 + 333 + 195 + 19135 + 239754 + 9381 + 11225 +
        3127 + 941 + 643 + 8525                                      = 365832

scaled_repair_cost = [365832 * (46-0) / 46] = 365832
repair_cost = [ 365832 / 1024 ] * 128 = 45696  (accurate in-game value: 45696)


Superior 15/15 Bone Visage Hearth (0 durability)
================================================

scaled_armor_cost = [87274 * (157+1) / 157]                               = 87829
socketed_items_cost = [16940 / 2] + [27440 / 2] + [8960 / 2]              = 26670

bonus_costs:
  15  item_armor_percent             [ (157+1) * 15 / 100 ] = 23
                                     [ 87829 * 23 / 1024 ] * 20 + 47 = 39487
                                     [ 39487 / 2 ]                        = 19743
  15  item_maxdurability_percent     [ 40 * 15 / 100 ] = 6
                                     [ 87829 * 6 / 1024 ] * 10 + 117 = 5257
                                     [ 5257 * 10 / 25 ]                   = 2102
  5   item_maxhp_percent             [ 87829 * 5  / 1024 ] * 204  + 32093 = 119405
  100 item_armor_percent             [ (157+1) * 100 / 100 ] = 158
                                     [ 87829 * 158 / 1024 ] * 20 + 47 = 271067
                                     [ 271067 / 2 ]                       = 135533
  30  coldresist                     [ 87829 * 30 / 1024 ] * 20   + 43    = 51503
  15  item_absorbcold_percent        [ 87829 * 15 / 1024 ] * 102  + 5486  = 136658
  1   item_cannotbefrozen            [ 87829 * 1  / 1024 ] * 2048 + 15011 = 189091
  20  item_fastergethitrate (Shael)  [ 87829 * 20 / 1024 ] * 72   + 1065  = 124545
  10  vitality (Io)                  [ 87829 * 10 / 1024 ] * 55   + 100   = 47235
  30  coldresist (Thul)              [ 87829 * 30 / 1024 ] * 20   + 43    = 51503

total = 87829 + 26670 + 19743 + 2102 + 119405 + 135533 + 51503 +
        136658 + 189091 + 124545 + 47235 + 51503                          = 991817

scaled_repair_cost = [991817 * (46-0) / 46] = 991817
repair_cost = [ 991817 / 1024 ] * 128 = 123904  (accurate in-game value: 123904)


Superior 15/15 Diadem Hearth (0 durability)
===========================================

scaled_armor_cost = [58000 * (60+1) / 60]                                = 58966
socketed_items_cost = [16940 / 2] + [27440 / 2] + [8960 / 2]             = 26670

bonus_costs:
  15  item_armor_percent             [ (60+1) * 15 / 100 ] = 9
                                     [ 58966 * 9 / 1024 ] * 20 + 47 = 10407
                                     [ 10407 / 2 ]                       = 5203
  15  item_maxdurability_percent     [ 20 * 15 / 100 ] = 3
                                     [ 58966 * 3 / 1024 ] * 10 + 117 = 1837
                                     [ 1837 * 10 / 25 ]                  = 734
  5   item_maxhp_percent             [ 58966 * 5  / 1024 ] * 204 + 32093 = 90641
  100 item_armor_percent             [ (60+1) * 100 / 100 ] = 61
                                     [ 58966 * 61 / 1024 ] * 20 + 47 = 70287
                                     [ 70287 / 2 ]                       = 35143
  30  coldresist                     [ 58966 * 30 / 1024 ] * 20  + 43    = 34583
  15  item_absorbcold_percent        [ 58966 * 15 / 1024 ] * 102 + 5486  = 93512
  1   item_cannotbefrozen            [ 58966 * 1 * 2048 / 1024 ] + 15011 = 132943
  20  item_fastergethitrate (Shael)  [ 58966 * 20 / 1024 ] * 72  + 1065  = 83937
  10  vitality (Io)                  [ 58966 * 10 / 1024 ] * 55  + 100   = 31725
  30  coldresist (Thul)              [ 58966 * 30 / 1024 ] * 20  + 43    = 34583

total = 58966 + 26670 + 5203 + 734 + 90641 + 35143 + 34583 + 93512 +
        132943 + 83937 + 31725 + 34583                                   = 628640

scaled_repair_cost = [628640 * (23-0) / 23] = 628640
repair_cost = [ 628640 / 1024 ] * 128 = 78464  (accurate in-game value: 77824)


Superior 15/15 Great Helm Hearth (0 durability)
===============================================

scaled_armor_cost = [6177 * (35+1) / 35]                                 = 6353
socketed_items_cost = [16940 / 2] + [27440 / 2] + [8960 / 2]             = 26670

bonus_costs:
  15  item_armor_percent             [ (35+1) * 15 / 100 ] = 5
                                     [ 6353 * 5 * 20 / 1024 ] + 47 = 667
                                     [ 667 / 2 ]                         = 333
  15  item_maxdurability_percent     [ 40 * 15 / 100 ] = 6
                                     [ 6353 * 6 * 10 / 1024 ] + 117 = 489
                                     [ 489 * 10 / 25 ]                   = 195
  5   item_maxhp_percent             [ 6353 * 5  * 204  / 1024 ] + 32093 = 38421
  100 item_armor_percent             [ (35+1) * 100 / 100 ] = 36
                                     [ 6353 * 36 / 1024 ] * 20 + 47 = 4507
                                     [ 4507 / 2 ]                        = 2253
  30  coldresist                     [ 6353 * 30 / 1024 ] * 20  + 43     = 3763
  15  item_absorbcold_percent        [ 6353 * 15 / 1024 ] * 102 + 5486   = 14972
  1   item_cannotbefrozen            [ 6353 * 1  * 2048 / 1024 ] + 15011 = 27717
  20  item_fastergethitrate (Shael)  [ 6353 * 20 / 1024 ] * 72  + 1065   = 9993
  10  vitality (Io)                  [ 6353 * 10 * 55   / 1024 ] + 100   = 3512
  30  coldresist (Thul)              [ 6353 * 30 / 1024 ] * 20  + 43     = 3763

total = 6353 + 26670 + 333 + 195 + 38421 + 2253 + 3763 + 14972 +
        27717 + 9993 + 3512 + 3763                                       = 137945

scaled_repair_cost = [137945 * (46-0) / 46] = 137945
repair_cost = [ 137945 / 1024 ] * 128 = 17152  (accurate in-game value: 17152)


Superior 15/15 Bone Visage Lore (0 durability)
==============================================

scaled_armor_cost = [87274 * (157+1) / 157]                               = 87829
socketed_items_cost = [6860 / 2] + [14000 / 2]                            = 10430

bonus_costs:
  15  item_armor_percent             [ (157+1) * 15 / 100 ] = 23
                                     [ 87829 * 23 / 1024 ] * 20 + 47 = 39487
                                     [ 39487 / 2 ]                        = 19743
  15  item_maxdurability_percent     [ 40 * 15 / 100 ] = 6
                                     [ 87829 * 6 / 1024 ] * 10 + 117 = 5257
                                     [ 5257 * 10 / 25 ]                   = 2102
  10  energy                         [ 87829 * 10 / 1024 ] * 55   + 100   = 47235
  1   item_allskills                 [ 87829 * 1  / 1024 ] * 4096 + 15123 = 363283
  2   item_lightradius               [ 87829 * 2  / 1024 ] * 51   + 15    = 8736
  2   item_manaafterkill             [ 87829 * 2  / 1024 ] * 102  + 17    = 17459
  30  lightresist (Ort)              [ 87829 * 30 / 1024 ] * 20   + 43    = 51503
  7   normal_damage_reduction (Sol)  [ 87829 * 7  / 1024 ] * 200  + 188   = 120188

total = 87829 + 10430 + 19743 + 2102 + 47235 + 363283 + 8736 +
        17459 + 51503 + 120188                                            = 728508

scaled_repair_cost = [728508 * (46-0) / 46] = 728508
repair_cost = [ 728508 / 1024 ] * 128 = 91008  (accurate in-game value: 90112)


Superior 15/15 Diadem Lore (0 durability)
=========================================

scaled_armor_cost = [58000 * (60+1) / 60]                                = 58966
socketed_items_cost = [6860 / 2] + [14000 / 2]                           = 10430

bonus_costs:
  15  item_armor_percent             [ (60+1) * 15 / 100 ] = 9
                                     [ 58966 * 9 / 1024 ] * 20 + 47 = 10407
                                     [ 10407 / 2 ]                       = 5203
  15  item_maxdurability_percent     [ 20 * 15 / 100 ] = 3
                                     [ 58966 * 3 / 1024 ] * 10 + 117 = 1837
                                     [ 1837 * 10 / 25 ]                  = 734
  10  energy                         [ 58966 * 10 / 1024 ] * 55  + 100   = 31725
  1   item_allskills                 [ 58966 * 1 * 4096 / 1024 ] + 15123 = 250987
  2   item_lightradius               [ 58966 * 2  / 1024 ] * 51  + 15    = 5880
  2   item_manaafterkill             [ 58966 * 2  / 1024 ] * 102 + 17    = 11747
  30  lightresist (Ort)              [ 58966 * 30 / 1024 ] * 20  + 43    = 34583
  7   normal_damage_reduction (Sol)  [ 58966 * 7  / 1024 ] * 200 + 188   = 80788

total = 58966 + 10430 + 5203 + 734 + 31725 + 250987 + 5880 +
        11747 + 34583 + 80788                                            = 491043

scaled_repair_cost = [491043 * (23-0) / 23] = 491043
repair_cost = [ 491043 / 1024 ] * 128 = 61312  (accurate in-game value: 61312)


Superior 15/15 Grim Helm Lore (0 durability)
============================================

scaled_armor_cost = [37683 * (125+1) / 125]                              = 37984
socketed_items_cost = [6860 / 2] + [14000 / 2]                           = 10430

bonus_costs:
  15  item_armor_percent             [ (125+1) * 15 / 100 ] = 18
                                     [ 37984 * 18 / 1024 ] * 20 + 47 = 13387
                                     [ 13387 / 2 ]                       = 6693
  15  item_maxdurability_percent     [ 40 * 15 / 100 ] = 6
                                     [ 37984 * 6 / 1024 ] * 10 + 117 = 2337
                                     [ 2337 * 10 / 25 ]                  = 934
  10  energy                         [ 37984 * 10 / 1024 ] * 55  + 100   = 20450
  1   item_allskills                 [ 37984 * 1 * 4096 / 1024 ] + 15123 = 167059
  2   item_lightradius               [ 37984 * 2  / 1024 ] * 51  + 15    = 3789
  2   item_manaafterkill             [ 37984 * 2  / 1024 ] * 102 + 17    = 7565
  30  lightresist (Ort)              [ 37984 * 30 / 1024 ] * 20  + 43    = 22283
  7   normal_damage_reduction (Sol)  [ 37984 * 7  / 1024 ] * 200 + 188   = 51988

total = 37984 + 10430 + 6693 + 934 + 20450 + 167059 + 3789 + 7565 +
        22283 + 51988                                                    = 329175

scaled_repair_cost = [329175 * (46-0) / 46] = 329175
repair_cost = [ 329175 / 1024 ] * 128 = 41088  (accurate in-game value: 41088)


Superior 15/15 Bone Helm Lore (0 durability)
============================================

scaled_armor_cost = [6323 * (36+1) / 36]                                 = 6498
socketed_items_cost = [6860 / 2] + [14000 / 2]                           = 10430

bonus_costs:
  15  item_armor_percent             [ (36+1) * 15 / 100 ] = 5
                                     [ 6498 * 5 * 20 / 1024 ] + 47 = 681
                                     [ 681 / 2 ]                         = 340
  15  item_maxdurability_percent     [ 40 * 15 / 100 ] = 6
                                     [ 6498 * 6 * 10 / 1024 ] + 117 = 497
                                     [ 497 * 10 / 25 ]                   = 198
  10  energy                         [ 6498 * 10 * 55   / 1024 ] + 100   = 3590
  1   item_allskills                 [ 6498 * 1  * 4096 / 1024 ] + 15123 = 41115
  2   item_lightradius               [ 6498 * 2  * 51   / 1024 ] + 15    = 662
  2   item_manaafterkill             [ 6498 * 2  * 102  / 1024 ] + 17    = 1311
  30  lightresist (Ort)              [ 6498 * 30 / 1024 ] * 20 + 43      = 3843
  7   normal_damage_reduction (Sol)  [ 6498 * 7  * 200  / 1024 ] + 188   = 9071

total = 6498 + 10430 + 340 + 198 + 3590 + 41115 + 662 + 1311 +
        3843 + 9071                                                      = 77058

scaled_repair_cost = [77058 * (46-0) / 46] = 77058
repair_cost = [ 77058 / 1024 ] * 128 = 9600  (accurate in-game value: 9600)


Superior 15/15 Great Helm Lore (0 durability)
=============================================

scaled_armor_cost = [6177 * (35+1) / 35]                                 = 6353
socketed_items_cost = [6860 / 2] + [14000 / 2]                           = 10430

bonus_costs:
  15  item_armor_percent             [ (35+1) * 15 / 100 ] = 5
                                     [ 6353 * 5 * 20 / 1024 ] + 47 = 667
                                     [ 667 / 2 ]                         = 333
  15  item_maxdurability_percent     [ 40 * 15 / 100 ] = 6
                                     [ 6353 * 6 * 10 / 1024 ] + 117 = 489
                                     [ 489 * 10 / 25 ]                   = 195
  10  energy                         [ 6353 * 10 * 55   / 1024 ] + 100   = 3512
  1   item_allskills                 [ 6353 * 1  * 4096 / 1024 ] + 15123 = 40535
  2   item_lightradius               [ 6353 * 2  * 51   / 1024 ] + 15    = 647
  2   item_manaafterkill             [ 6353 * 2  * 102  / 1024 ] + 17    = 1282
  30  lightresist (Ort)              [ 6353 * 30 / 1024 ] * 20 + 43      = 3763
  7   normal_damage_reduction (Sol)  [ 6353 * 7  * 200  / 1024 ] + 188   = 8873

total = 6353 + 10430 + 333 + 195 + 3512 + 40535 + 647 + 1282 +
        3763 + 8873                                                      = 75923

scaled_repair_cost = [75923 * (46-0) / 46] = 75923
repair_cost = [ 75923 / 1024 ] * 128 = 9472  (accurate in-game value: 9472)


Superior 15/15 Helm Lore (0 durability)
=======================================

scaled_armor_cost = [1558 * (18+1) / 18]                                 = 1644
socketed_items_cost = [6860 / 2] + [14000 / 2]                           = 10430

bonus_costs:
  15  item_armor_percent             [ (18+1) * 15 / 100 ] = 2
                                     [ 1644 * 2 * 20 / 1024 ] + 47 = 111
                                     [ 111 / 2 ]                         = 55
  15  item_maxdurability_percent     [ 24 * 15 / 100 ] = 3
                                     [ 1644 * 3 * 10 / 1024 ] + 117 = 165
                                     [ 165 * 10 / 25 ]                   = 66
  10  energy                         [ 1644 * 10 * 55   / 1024 ] + 100   = 983
  1   item_allskills                 [ 1644 * 1  * 4096 / 1024 ] + 15123 = 21699
  2   item_lightradius               [ 1644 * 2  * 51   / 1024 ] + 15    = 178
  2   item_manaafterkill             [ 1644 * 2  * 102  / 1024 ] + 17    = 344
  30  lightresist (Ort)              [ 1644 * 30 * 20   / 1024 ] + 43    = 1006
  7   normal_damage_reduction (Sol)  [ 1644 * 7  * 200  / 1024 ] + 188   = 2435

total = 1644 + 10430 + 55 + 66 + 983 + 21699 + 178 + 344 + 1006 + 2435   = 38840

scaled_repair_cost = [38840 * (27-0) / 27] = 38840
repair_cost = [ 38840 * 128 / 1024 ] = 4855  (accurate in-game value: 4849)
```


Repair cost tables
------------------

Summary of the above calculations:

| runeword         | 15/15 base           | charsi cost | calculated cost |
|:-----------------|:---------------------|:------------|:----------------|
| Enigma           | Archon Plate         | 1,485,824   | 1,485,824       |
|                  | Dusk Shroud          | 1,026,048   | 1,026,176       |
|                  | Mage Plate           | 563,200     | 563,712         |
|                  | Full Plate Mail      | 237,568     | 237,568         |
|                  | Breast Plate         | 68,608      | 68,864          |
| Chains of Honor  | Archon Plate         | 1,271,808   | 1,271,936       |
|                  | Dusk Shroud          | 866,304     | 866,944         |
| Dragon           | Archon Plate         | 1,641,472   | 1,641,728       |
|                  | Dusk Shroud          | 1,137,664   | 1,137,664       |
| Fortitude        | Archon Plate         | 1,911,808   | 1,912,320       |
|                  | Dusk Shroud          | 1,285,120   | 1,286,016       |
| Duress           | Archon Plate         | 3,217,408   | 3,209,728       |
|                  | Dusk Shroud          | 2,181,120   | 2,176,256       |
| Hustle           | Archon Plate         | 1,041,408   | 1,041,792       |
|                  | Dusk Shroud          | 715,776     | 716,288         |
| Treachery        | Archon Plate         | 1,524,736   | 1,524,992       |
|                  | Dusk Shroud          | 1,051,648   | 1,052,288       |
| Smoke            | Archon Plate         | 601,088     | 601,088         |
|                  | Dusk Shroud          | 399,360     | 400,256         |
|                  | Mage Plate           | 191,488     | 191,872         |
| Stealth          | Dusk Shroud          | 363,520     | 364,416         |
|                  | Mage Plate           | 193,536     | 194,432         |
|                  | Breast Plate         | 17,664      | 17,664          |
| Phoenix          | 45@ Sacred Targe     | 1,186,816   | 1,187,072       |
|                  | Monarch              | 1,299,456   | 1,299,840       |
| Dream            | 45@ Sacred Targe     | 242,688     | 243,200         |
| Dragon           | 45@ Sacred Targe     | 423,936     | 424,832         |
| Spirit           | 45@ Sacred Targe     | 295,936     | 296,064         |
|                  | 45@ Protector Shield | 217,088     | 217,344         |
|                  | Monarch              | 296,960     | 297,728         |
| Ancient's Pledge | 45@ Sacred Targe     | 86,016      | 86,656          |
|                  | 45@ Protector Shield | 63,360      | 63,360          |
|                  | Large Shield         | 1,796       | 1,825           |
| Dream            | Bone Visage          | 254,976     | 254,976         |
|                  | Diadem               | 173,056     | 173,952         |
|                  | Great Helm           | 35,072      | 35,072          |
| Flickering Flame | Bone Visage          | 522,240     | 522,240         |
|                  | Diadem               | 350,208     | 350,208         |
|                  | Great Helm           | 45,696      | 45,696          |
| Hearth           | Bone Visage          | 123,904     | 123,904         |
|                  | Diadem               | 77,824      | 78,464          |
|                  | Great Helm           | 17,152      | 17,152          |
| Lore             | Bone Visage          | 90,112      | 91,008          |
|                  | Diadem               | 61,312      | 61,312          |
|                  | Grim Helm            | 41,088      | 41,088          |
|                  | Bone Helm            | 9,600       | 9,600           |
|                  | Great Helm           | 9,472       | 9,472           |
|                  | Helm                 | 4,849       | 4,855           |


Comparing the superior repair costs of runewords in the same base:

| 15/15 base       | runeword         | charsi cost |
|:-----------------|:-----------------|:------------|
| Dusk Shroud      | Duress           | 2,181,120   |
|                  | Fortitude        | 1,285,120   |
|                  | Dragon           | 1,137,664   |
|                  | Treachery        | 1,051,648   |
|                  | Enigma           | 1,026,048   |
|                  | Chains of Honor  | 866,304     |
|                  | Hustle           | 715,776     |
|                  | Smoke            | 399,360     |
|                  | Stealth          | 363,520     |
| 45@ Sacred Targe | Phoenix          | 1,186,816   |
|                  | Dragon           | 423,936     |
|                  | Spirit           | 295,936     |
|                  | Dream            | 242,688     |
|                  | Ancient's Pledge | 86,016      |
| Bone Visage      | Flickering Flame | 522,240     |
|                  | Dream            | 254,976     |
|                  | Hearth           | 123,904     |
|                  | Lore             | 90,112      |


Inaccuracies
------------

The game uses integer arithmetic to perform calculations, and integer division is a lossy operation because the fractions are simply dropped. For this reason the order of operations matters. In an ideal world `(16 * 128) / 1024` and `(16 / 1024) * 128` yield the same result (2), but with integer arithmetic `(16 * 128) / 1024 = 2` and `(16 / 1024) * 128 = 0` because `16 / 1024 = 0`.

This is why the formula I shared performs multiplication first when the starting value is small (below 65536, or 0x1000 in hexadecimal, basically an unsigned 16-bit value). Why don't we always perform multiplication first? Because large numbers and intermediate results (after multiplication) can cause integer overflow, and D2 probably uses 32-bit integers in a lot of places.

Scaling values before or after adding them can also introduce small differences. For example:

```
[83 * 128 / 1024] + [69 * 128 / 1024] = 18
[(83 + 69) * 128 / 1024] = 19

Simplified version:

[83 / 8] + [69 / 8] = 18
[(83 + 69) / 8] = 19
```

Charsi's repair costs seem to be an integer multiple of 1024 for expensive repairs. This means that the least significant 10 bits of the cost are all zero. Zeroing out the least significant 10 bits of my calculated repair cost often results in the same value as Charsi's. For example, for a Dusk Shroud Enigma: `[1026176 / 1024] * 1024 = 1026048`.

I didn't reverse-engineer the game, so my formula is an approximation of what the game does, based on observations and the limited documentation I found. I don't exactly know how the game orders its integer operations to avoid overflows and lossy calculations with small numbers, and this can lead to small differences even if my formula is correct.


Why 65536???
------------

65536 is a round number in base two and also in base 16 (hexadecimal 0x10000). It's the lowest positive number that doesn't fit into an unsigned 16-bit integer, it requires 17 bits.

I was messing with my character editor and asked Charsi about the repair costs of some edited/hacked plain Archon Plates. I noticed that my price calculations were correct with Archon Plate defense < 109 only by using `*128/1024`, and defense >= 109 required `/1024*128` to agree with Charsi's repair cost. Here are the calculations of that experiment:

```
Correct calculations
====================

108 def Archon Plate repair cost from zero durability:

scaled_armor_cost = 317526 * 108 / 524 = 65444  ( < 65536 )
repair_cost = [65444 * 128 / 1024] = 8180  (approved by Charsi)

109 def Archon Plate repair cost from zero durability:

scaled_armor_cost = 317526 * 109 / 524 = 66050  ( >= 65536 )
repair_cost = [66050 / 1024] * 128 = 8192  (approved by Charsi)


Incorrect calculations
======================

108 def Archon Plate repair cost from zero durability:

scaled_armor_cost = 317526 * 108 / 524 = 65444
repair_cost = [65444 / 1024] * 128 = 8064  (inaccurate)

109 def Archon Plate repair cost from zero durability:

scaled_armor_cost = 317526 * 109 / 524 = 66050
repair_cost = [66050 * 128 / 1024] = 8256  (inaccurate)
```

The `*128/1024` factor can be simplified to `/8` (it's actually better because it reduces the risk of a potential integer overflow caused by `*128`), but `/1024*8` isn't equivalent to `/8`, because `/1024` is much more "lossy", it often throws out much larger remainders than `/8`, and that's how Diablo 2's cost calculation seems to work.
