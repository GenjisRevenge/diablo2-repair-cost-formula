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
socketed_items_cost = [sum_of_jewel_gem_rune_costs_from_misc_txt / 2]
sum_of_bonus_costs = for each bonus (ignore runeword bonuses on non-superior bases):
    bonus_cost = [scaled_armor_cost * bonus_value * multiply_from_itemstatcost_txt / 1024 ] + add_from_itemstatcost_txt
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
        bonus_cost = [scaled_armor_cost * extra_defense_points * 20 / 1024 ] + 47
        bonus_cost = [bonus_cost / 2]
else
    bonus_cost = [base_item_cost * enhanced_defense * 20 / 1024 ] + 47


Increase Maximum Durability (item_maxdurability_percent) cost formula
=====================================================================

extra_durability_points = [max_durability * increase_max_durability / 100]
if extra_durability_points = 0 then
    bonus_cost = 0
else
    bonus_cost = [scaled_armor_cost * extra_durability_points * 10 / 1024 ] + 117
    bonus_cost = [bonus_cost * 10 / 25]
```

Why must the bonus costs from "Enhanced Defense" be divided by 2, and those from "Increase Maximum Durability" be divided by 2.5? I have no definitive explanation, but empirical testing provides strong evidence. By observing Charsi's repair costs on character-edited armors with widely varying defense, durability, ED, and IMD values, these divisors consistently emerge. Using a character editor to assign extreme values (like ED ranging from 1 to ~500 and IMD from 1 to ~100) makes the underlying relationships even clearer and allows these conclusions to be drawn with confidence.


**Calculations:**

```
Non-superior 524 def Archon Plate (empty, 0 durability)
=======================================================

scaled_armor_cost = [317526 * 524 / 524] = 317526
socketed_items_cost = 0

total = 317526

scaled_repair_cost = [317526 * (60-0) / 60] = 317526
repair_cost = [ 317526 / 1024 ] * 128 = 39680  (accurate in-game value: 39680)


Superior 15/0 Archon Plate (empty, 0 durability)
================================================

scaled_armor_cost = [317526 * (524+1) / 524] = 318131
socketed_items_cost = 0

bonus_costs:
  15  item_armor_percent          [ (524+1) * 15 / 100 ] = 78
                                  [ 318131 * 78 * 20 / 1024 ] + 47 = 484699
                                  [ 484699 / 2 ] = 242349

total = 318131 + 242349 = 560480

scaled_repair_cost = [560480 * (60-0) / 60] = 560480
repair_cost = [ 560480 / 1024 ] * 128 = 70016  (accurate in-game value: 69632)


Non-superior 249 def Mage Plate (empty, 0 durability)
=====================================================

scaled_armor_cost = [118407 * 249 / 261] = 112963
socketed_items_cost = 0

total = 112963 + 0 = 112963

scaled_repair_cost = [112963 * (60-0) / 60] = 112963
repair_cost = [ 112963 / 1024 ] * 128 = 14080  (accurate in-game value: 14080)


Non-superior 249 def Mage Plate with Jah+Ith (0 durability)
===========================================================

scaled_armor_cost = [118407 * 249 / 261] = 112963
socketed_items_cost = [(117740 + 2240) / 2] = 59990

total = 112963 + 59990 = 172953

scaled_repair_cost = [172953 * (60-0) / 60] = 172953
repair_cost = [ 172953 / 1024 ] * 128 = 21504  (accurate in-game value: 21504)


Non-superior 249 def Mage Plate Enigma (+750 def roll, 0 and 59 durability)
===========================================================================

scaled_armor_cost = [118407 * 249 / 261] = 112963
socketed_items_cost = [(117740 + 2240 + 109760) / 2] = 114870

total = 112963 + 114870 = 227833

- 0/60 durability:
    scaled_repair_cost = [227833 * (60-0) / 60] = 227833
    repair_cost = [ 227833 / 1024 ] * 128 = 28416  (accurate in-game value: 28416)

- 59/60 durability:
    scaled_repair_cost = [227833 * (60-59) / 60] = 3797
    repair_cost = [ 3797 * 128 / 1024 ] = 474  (accurate in-game value: 474)


Superior 15/15 Dusk Shroud Enigma (0 durability)
================================================

scaled_armor_cost = [218357 * (467+1) / 467]                             = 218824
socketed_items_cost = [(117740 + 2240 + 109760) / 2]                     = 114870

bonus_costs:
  15  item_armor_percent          [ (467+1) * 15 / 100 ] = 70
                                  [ 218824 * 70 * 20 / 1024 ] + 47 = 299220
                                  [ 299220 / 2 ]                         = 149610
  15  item_maxdurability_percent  [ 20 * 15 / 100 ] = 3
                                  [ 218824 * 3 * 10 / 1024 ] + 117 = 6527
                                  [ 6527 * 10 / 25 ]                     = 2610
  775 armorclass                  [ 218824 * 775 * 10   / 1024 ] + 17    = 1656155
  14  item_healafterkill          [ 218824 * 14  * 101  / 1024 ] + 30    = 302195
  45  item_fastermovevelocity     [ 218824 * 45  * 156  / 1024 ] + 4083  = 1504224
  6   item_strength_perlevel      [ 218824 * 6   * 128  / 1024 ] + 132   = 164250
  2   item_allskills              [ 218824 * 2   * 4096 / 1024 ] + 15123 = 1765715
  8   item_find_magic_perlevel    [ 218824 * 8   * 1024 / 1024 ] + 814   = 1751406
  1   item_nonclassskill          [ 218824 * 1   * 640  / 1024 ] + 16000 = 152765
      (Teleport, skills.txt: mul=640 add=16000)
  5   item_maxhp_percent (Jah)    [ 218824 * 5   * 204  / 1024 ] + 32093 = 250062
  15  item_damagetomana (Ith)     [ 218824 * 15  * 20   / 1024 ] + 43    = 64151
  8   damageresist (Ber)          [ 218824 * 8   * 68   / 1024 ] + 152   = 116402

total = 218824 + 114870 + 149610 + 2610 + 1656155 + 302195 + 1504224 +
        164250 + 1765715 + 1751406 + 152765 + 250062 + 64151 + 116402    = 8213239

scaled_repair_cost = [8213239 * (23-0) / 23] = 8213239
repair_cost = [ 8213239 / 1024 ] * 128 = 1026560  (accurate in-game value: 1026048)


Superior 15/15 Archon Plate Enigma (0 durability)
=================================================

scaled_armor_cost = [317526 * (524+1) / 524]                             = 318131
socketed_items_cost = [(117740 + 2240 + 109760) / 2]                     = 114870

bonus_costs:
  15  item_armor_percent          [ (524+1) * 15 / 100 ] = 78
                                  [ 318131 * 78 * 20 / 1024 ] + 47 = 484699
                                  [ 484699 / 2 ]                         = 242349
  15  item_maxdurability_percent  [ 60 * 15 / 100 ] = 9
                                  [ 318131 * 9 * 10 / 1024 ] + 117 = 28077
                                  [ 28077 * 10 / 25 ]                    = 11230
  775 armorclass                  [ 318131 * 775 * 10   / 1024 ] + 17    = 2407746
  14  item_healafterkill          [ 318131 * 14  * 101  / 1024 ] + 30    = 439324
  45  item_fastermovevelocity     [ 318131 * 45  * 156  / 1024 ] + 4083  = 2185020
  6   item_strength_perlevel      [ 318131 * 6   * 128  / 1024 ] + 132   = 238730
  2   item_allskills              [ 318131 * 2   * 4096 / 1024 ] + 15123 = 2560171
  8   item_find_magic_perlevel    [ 318131 * 8   * 1024 / 1024 ] + 814   = 2545862
  1   item_nonclassskill          [ 318131 * 1   * 640  / 1024 ] + 16000 = 214831
      (Teleport, skills.txt: mul=640 add=16000)
  5   item_maxhp_percent (Jah)    [ 318131 * 5   * 204  / 1024 ] + 32093 = 348981
  15  item_damagetomana (Ith)     [ 318131 * 15  * 20   / 1024 ] + 43    = 93245
  8   damageresist (Ber)          [ 318131 * 8   * 68   / 1024 ] + 152   = 169159

total = 318131 + 114870 + 242349 + 11230 + 2407746 + 439324 + 2185020 +
        238730 + 2560171 + 2545862 + 214831 + 348981 + 93245 + 169159    = 11889649

scaled_repair_cost = [11889649 * (69-0) / 69] = 11889649
repair_cost = [ 11889649 / 1024 ] * 128 = 1486080  (accurate in-game value: 1485824)


Superior 15/15 Mage Plate Enigma (0 durability)
===============================================

scaled_armor_cost = [118407 * (261+1) / 261]                             = 118860
socketed_items_cost = [(117740 + 2240 + 109760) / 2]                     = 114870

bonus_costs:
  15  item_armor_percent          [ (261+1) * 15 / 100 ] = 39
                                  [ 118860 * 39 * 20 / 1024 ] + 47 = 90584
                                  [ 90584 / 2 ]                          = 45292
  15  item_maxdurability_percent  [ 60 * 15 / 100 ] = 9
                                  [ 118860 * 9 * 10 / 1024 ] + 117 = 10563
                                  [ 10563 * 10 / 25 ]                    = 4225
  775 armorclass                  [ 118860 * 775 * 10   / 1024 ] + 17    = 899592
  14  item_healafterkill          [ 118860 * 14  * 101  / 1024 ] + 30    = 164158
  45  item_fastermovevelocity     [ 118860 * 45  * 156  / 1024 ] + 4083  = 818924
  6   item_strength_perlevel      [ 118860 * 6   * 128  / 1024 ] + 132   = 89277
  2   item_allskills              [ 118860 * 2   * 4096 / 1024 ] + 15123 = 966003
  8   item_find_magic_perlevel    [ 118860 * 8   * 1024 / 1024 ] + 814   = 951694
  1   item_nonclassskill          [ 118860 * 1   * 640  / 1024 ] + 16000 = 90287
      (Teleport, skills.txt: mul=640 add=16000)
  5   item_maxhp_percent (Jah)    [ 118860 * 5   * 204  / 1024 ] + 32093 = 150488
  15  item_damagetomana (Ith)     [ 118860 * 15  * 20   / 1024 ] + 43    = 34865
  8   damageresist (Ber)          [ 118860 * 8   * 68   / 1024 ] + 152   = 63296

total = 118860 + 114870 + 45292 + 4225 + 899592 + 164158 + 818924 +
        89277 + 966003 + 951694 + 90287 + 150488 + 34865 + 63296         = 4511831

scaled_repair_cost = [4511831 * (69-0) / 69] = 4511831
repair_cost = [ 4511831 / 1024 ] * 128 = 563968  (accurate in-game value: 563200)


Superior 15/15 Full Plate Mail Enigma (0 durability)
====================================================

scaled_armor_cost = [47192 * (161+1) / 161]                              = 47485
socketed_items_cost = [(117740 + 2240 + 109760) / 2]                     = 114870

bonus_costs:
  15  item_armor_percent          [ (161+1) * 15 / 100 ] = 24
                                  [ 47485 * 24 * 20 / 1024 ] + 47 = 22305
                                  [ 22305 / 2 ]                          = 11152
  15  item_maxdurability_percent  [ 70 * 15 / 100 ] = 10
                                  [ 47485 * 10 * 10 / 1024 ] + 117 = 4754
                                  [ 4754 * 10 / 25 ]                     = 1901
  775 armorclass                  [ 47485 * 775 * 10   / 1024 ] + 17     = 359400
  14  item_healafterkill          [ 47485 * 14  * 101  / 1024 ] + 30     = 65600
  45  item_fastermovevelocity     [ 47485 * 45  * 156  / 1024 ] + 4083   = 329614
  6   item_strength_perlevel      [ 47485 * 6   * 128  / 1024 ] + 132    = 35745
  2   item_allskills              [ 47485 * 2   * 4096 / 1024 ] + 15123  = 395003
  8   item_find_magic_perlevel    [ 47485 * 8   * 1024 / 1024 ] + 814    = 380694
  1   item_nonclassskill          [ 47485 * 1   * 640  / 1024 ] + 16000  = 45678
      (Teleport, skills.txt: mul=640 add=16000)
  5   item_maxhp_percent (Jah)    [ 47485 * 5   * 204  / 1024 ] + 32093  = 79392
  15  item_damagetomana (Ith)     [ 47485 * 15  * 20   / 1024 ] + 43     = 13954
  8   damageresist (Ber)          [ 47485 * 8   * 68   / 1024 ] + 152    = 25378

total = 47485 + 114870 + 11152 + 1901 + 359400 + 65600 + 329614 + 35745 +
        395003 + 380694 + 45678 + 79392 + 13954 + 25378                  = 1905866

scaled_repair_cost = [1905866 * (80-0) / 80] = 1905866
repair_cost = [ 1905866 / 1024 ] * 128 = 238208  (accurate in-game value: 237568)


Superior 15/15 Breast Plate Enigma (0 durability)
=================================================

scaled_armor_cost = [10078 * (68+1) / 68]                                = 10226
socketed_items_cost = [(117740 + 2240 + 109760) / 2]                     = 114870

bonus_costs:
  15  item_armor_percent          [ (68+1) * 15 / 100 ] = 10
                                  [ 10226 * 10 * 20 / 1024 ] + 47 = 2044
                                  [ 2044 / 2 ]                           = 1022
  15  item_maxdurability_percent  [ 50 * 15 / 100 ] = 7
                                  [ 10226 * 7 * 10 / 1024 ] + 117 = 816
                                  [ 816 * 10 / 25 ]                      = 326
  775 armorclass                  [ 10226 * 775 * 10   / 1024 ] + 17     = 77411
  14  item_healafterkill          [ 10226 * 14  * 101  / 1024 ] + 30     = 14150
  45  item_fastermovevelocity     [ 10226 * 45  * 156  / 1024 ] + 4083   = 74187
  6   item_strength_perlevel      [ 10226 * 6   * 128  / 1024 ] + 132    = 7801
  2   item_allskills              [ 10226 * 2   * 4096 / 1024 ] + 15123  = 96931
  8   item_find_magic_perlevel    [ 10226 * 8   * 1024 / 1024 ] + 814    = 82622
  1   item_nonclassskill          [ 10226 * 1   * 640  / 1024 ] + 16000  = 22391
      (Teleport, skills.txt: mul=640 add=16000)
  5   item_maxhp_percent (Jah)    [ 10226 * 5   * 204  / 1024 ] + 32093  = 42279
  15  item_damagetomana (Ith)     [ 10226 * 15  * 20   / 1024 ] + 43     = 3038
  8   damageresist (Ber)          [ 10226 * 8   * 68   / 1024 ] + 152    = 5584

total = 10226 + 114870 + 1022 + 326 + 77411 + 14150 + 74187 + 7801 +
        96931 + 82622 + 22391 + 42279 + 3038 + 5584                      = 552838

scaled_repair_cost = [552838 * (57-0) / 57] = 552838
repair_cost = [ 552838 / 1024 ] * 128 = 68992  (accurate in-game value: 68608)


Superior 15/15 Dusk Shroud Chains of Honor (0 durability)
=========================================================

scaled_armor_cost = [218357 * (467+1) / 467]                             = 218824
socketed_items_cost = [(20160 + 56000 + 109760 + 67760) / 2]             = 126840

bonus_costs:
  15  item_armor_percent          [ (467+1) * 15 / 100 ] = 70
                                  [ 218824 * 70 * 20 / 1024 ] + 47 = 299220
                                  [ 299220 / 2 ]                         = 149610
  15  item_maxdurability_percent  [ 20 * 15 / 100 ] = 3
                                  [ 218824 * 3 * 10 / 1024 ] + 117 = 6527
                                  [ 6527 * 10 / 25 ]                     = 2610
  50  fireresist                  [ 218824 * 50  * 20   / 1024 ] + 43    = 213738
  50  lightresist                 [ 218824 * 50  * 20   / 1024 ] + 43    = 213738
  50  coldresist                  [ 218824 * 50  * 20   / 1024 ] + 43    = 213738
  50  poisonresist                [ 218824 * 50  * 20   / 1024 ] + 43    = 213738
  70  item_armor_percent          [ (467+1) * 70 / 100 ] = 327
                                  [ 218824 * 327 * 20 / 1024 ] + 47 = 1397614
                                  [ 1397614 / 2 ]                        = 698807
  200 item_demondamage_percent    [ 218824 * 200 * 12   / 1024 ] + 19    = 512887
  100 item_undeaddamage_percent   [ 218824 * 100 * 12   / 1024 ] + 13    = 256447
  8   lifedrainmindam             [ 218824 * 8   * 341  / 1024 ] + 1044  = 584004
  2   item_allskills              [ 218824 * 2   * 4096 / 1024 ] + 15123 = 1765715
  20  strength                    [ 218824 * 20  * 55   / 1024 ] + 125   = 235189
  7   hpregen (Dol)               [ 218824 * 7   * 410  / 1024 ] + 451   = 613756
  15  fireresist (Um)             [ 218824 * 15  * 20   / 1024 ] + 43    = 64151
  15  lightresist (Um)            [ 218824 * 15  * 20   / 1024 ] + 43    = 64151
  15  coldresist (Um)             [ 218824 * 15  * 20   / 1024 ] + 43    = 64151
  15  poisonresist (Um)           [ 218824 * 15  * 20   / 1024 ] + 43    = 64151
  8   damageresist (Ber)          [ 218824 * 8   * 68   / 1024 ] + 152   = 116402
  25  item_magicbonus (Ist)       [ 218824 * 25  * 102  / 1024 ] + 577   = 545500

total = 218824 + 126840 + 149610 + 2610 + 4 * 213738 + 698807 + 512887 +
        256447 + 584004 + 1765715 + 235189 + 613756 + 4 * 64151 +
        116402 + 545500                                                  = 6938147

scaled_repair_cost = [6938147 * (23-0) / 23] = 6938147
repair_cost = [ 6938147 / 1024 ] * 128 = 867200  (accurate in-game value: 866304)


Superior 15/15 Archon Plate Chains of Honor (0 durability)
==========================================================

scaled_armor_cost = [317526 * (524+1) / 524]                             = 318131
socketed_items_cost = [(20160 + 56000 + 109760 + 67760) / 2]             = 126840

bonus_costs:
  15  item_armor_percent          [ (524+1) * 15 / 100 ] = 78
                                  [ 318131 * 78 * 20 / 1024 ] + 47 = 484699
                                  [ 484699 / 2 ]                         = 242349
  15  item_maxdurability_percent  [ 60 * 15 / 100 ] = 9
                                  [ 318131 * 9 * 10 / 1024 ] + 117 = 28077
                                  [ 28077 * 10 / 25 ]                    = 11230
  50  fireresist                  [ 318131 * 50  * 20   / 1024 ] + 43    = 310717
  50  lightresist                 [ 318131 * 50  * 20   / 1024 ] + 43    = 310717
  50  coldresist                  [ 318131 * 50  * 20   / 1024 ] + 43    = 310717
  50  poisonresist                [ 318131 * 50  * 20   / 1024 ] + 43    = 310717
  70  item_armor_percent          [ (524+1) * 70 / 100 ] = 367
                                  [ 318131 * 367 * 20 / 1024 ] + 47 = 2280400
                                  [ 2280400 / 2 ]                        = 1140200
  200 item_demondamage_percent    [ 318131 * 200 * 12   / 1024 ] + 19    = 745638
  100 item_undeaddamage_percent   [ 318131 * 100 * 12   / 1024 ] + 13    = 372822
  8   lifedrainmindam             [ 318131 * 8   * 341  / 1024 ] + 1044  = 848564
  2   item_allskills              [ 318131 * 2   * 4096 / 1024 ] + 15123 = 2560171
  20  strength                    [ 318131 * 20  * 55   / 1024 ] + 125   = 341867
  7   hpregen (Dol)               [ 318131 * 7   * 410  / 1024 ] + 451   = 892087
  15  fireresist (Um)             [ 318131 * 15  * 20   / 1024 ] + 43    = 93245
  15  lightresist (Um)            [ 318131 * 15  * 20   / 1024 ] + 43    = 93245
  15  coldresist (Um)             [ 318131 * 15  * 20   / 1024 ] + 43    = 93245
  15  poisonresist (Um)           [ 318131 * 15  * 20   / 1024 ] + 43    = 93245
  8   damageresist (Ber)          [ 318131 * 8   * 68   / 1024 ] + 152   = 169159
  25  item_magicbonus (Ist)       [ 318131 * 25  * 102  / 1024 ] + 577   = 792797

total = 318131 + 126840 + 242349 + 11230 + 4 * 310717 + 1140200 +
        745638 + 372822 + 848564 + 2560171 + 341867 + 892087 +
        4 * 93245 + 169159 + 792797                                      = 10177703

scaled_repair_cost = [10177703 * (69-0) / 69] = 10177703
repair_cost = [ 10177703 / 1024 ] * 128 = 1272192  (accurate in-game value: 1271808)


Superior 15/15 Dusk Shroud Dragon (0 durability)
================================================

scaled_armor_cost = [218357 * (467+1) / 467]                             = 218824
socketed_items_cost = [(102060 + 94640 + 14000) / 2]                     = 105350

bonus_costs:
  15  item_armor_percent          [ (467+1) * 15 / 100 ] = 70
                                  [ 218824 * 70 * 20 / 1024 ] + 47 = 299220
                                  [ 299220 / 2 ]                         = 149610
  15  item_maxdurability_percent  [ 20 * 15 / 100 ] = 3
                                  [ 218824 * 3 * 10 / 1024 ] + 117 = 6527
                                  [ 6527 * 10 / 25 ]                     = 2610
  360 armorclass                  [ 218824 * 360 * 10   / 1024 ] + 17    = 769320
  230 armorclass_vs_missile       [ 218824 * 230 * 5    / 1024 ] + 11    = 245760
  3   item_strength_perlevel      [ 218824 * 3   * 128  / 1024 ] + 132   = 82191
  15  item_skillonhit             [ 218824 * 15  * 896  / 1024 ] + 64000 = 2936065
      (Hydra, skills.txt: mul=896 add=64000)
  18  item_skillongethit          [ 218824 * 18  * 896  / 1024 ] + 64000 = 3510478
      (Venom, skills.txt: mul=896 add=64000)
  14  item_aura (Level 14 Holy Fire Aura When Equipped)                  = 0
  5   strength                    [ 218824 * 5   * 55   / 1024 ] + 125   = 58891
  5   dexterity                   [ 218824 * 5   * 55   / 1024 ] + 125   = 58891
  5   vitality                    [ 218824 * 5   * 55   / 1024 ] + 100   = 58866
  5   energy                      [ 218824 * 5   * 55   / 1024 ] + 100   = 58866
  5   item_maxmana_percent (Sur)  [ 218824 * 5   * 204  / 1024 ] + 56452 = 274421
  5   maxlightresist (Lo)         [ 218824 * 5   * 256  / 1024 ] + 584   = 274114
  7   normal_damage_reduction (Sol) [ 218824 * 7 * 200  / 1024 ] + 188   = 299361

total = 218824 + 105350 + 149610 + 2610 + 769320 + 245760 + 82191 +
        2936065 + 3510478 + 2 * 58891 + 2 * 58866 + 274421 + 274114 +
        299361                                                           = 9103618

scaled_repair_cost = [9103618 * (23-0) / 23] = 9103618
repair_cost = [ 9103618 / 1024 ] * 128 = 1137920  (accurate in-game value: 1137664)


Superior 15/15 Archon Plate Dragon (0 durability)
=================================================

scaled_armor_cost = [317526 * (524+1) / 524]                             = 318131
socketed_items_cost = [(102060 + 94640 + 14000) / 2]                     = 105350

bonus_costs:
  15  item_armor_percent          [ (524+1) * 15 / 100 ] = 78
                                  [ 318131 * 78 * 20 / 1024 ] + 47 = 484699
                                  [ 484699 / 2 ]                         = 242349
  15  item_maxdurability_percent  [ 60 * 15 / 100 ] = 9
                                  [ 318131 * 9 * 10 / 1024 ] + 117 = 28077
                                  [ 28077 * 10 / 25 ]                    = 11230
  360 armorclass                  [ 318131 * 360 * 10   / 1024 ] + 17    = 1118446
  230 armorclass_vs_missile       [ 318131 * 230 * 5    / 1024 ] + 11    = 357287
  3   item_strength_perlevel      [ 318131 * 3   * 128  / 1024 ] + 132   = 119431
  15  item_skillonhit             [ 318131 * 15  * 896  / 1024 ] + 64000 = 4239469
      (Hydra, skills.txt: mul=896 add=64000)
  18  item_skillongethit          [ 318131 * 18  * 896  / 1024 ] + 64000 = 5074563
      (Venom, skills.txt: mul=896 add=64000)
  14  item_aura (Level 14 Holy Fire Aura When Equipped)                  = 0
  5   strength                    [ 318131 * 5   * 55   / 1024 ] + 125   = 85560
  5   dexterity                   [ 318131 * 5   * 55   / 1024 ] + 125   = 85560
  5   vitality                    [ 318131 * 5   * 55   / 1024 ] + 100   = 85535
  5   energy                      [ 318131 * 5   * 55   / 1024 ] + 100   = 85535
  5   item_maxmana_percent (Sur)  [ 318131 * 5   * 204  / 1024 ] + 56452 = 373340
  5   maxlightresist (Lo)         [ 318131 * 5   * 256  / 1024 ] + 584   = 398247
  7   normal_damage_reduction (Sol) [ 318131 * 7 * 200  / 1024 ] + 188   = 435132

total = 318131 + 105350 + 242349 + 11230 + 1118446 + 357287 + 119431 +
        4239469 + 5074563 + 2 * 85560 + 2 * 85535 + 373340 + 398247 +
        435132                                                           = 13135165

scaled_repair_cost = [13135165 * (69-0) / 69] = 13135165
repair_cost = [ 13135165 / 1024 ] * 128 = 1641856  (accurate in-game value: 1641472)


Superior 15/15 Dusk Shroud Fortitude (0 durability)
===================================================

scaled_armor_cost = [218357 * (467+1) / 467]                             = 218824
socketed_items_cost = [(560 + 14000 + 20160 + 94640) / 2]                = 64680

bonus_costs:
  15  item_armor_percent          [ (467+1) * 15 / 100 ] = 70
                                  [ 218824 * 70 * 20 / 1024 ] + 47 = 299220
                                  [ 299220 / 2 ]                         = 149610
  15  item_maxdurability_percent  [ 20 * 15 / 100 ] = 3
                                  [ 218824 * 3 * 10 / 1024 ] + 117 = 6527
                                  [ 6527 * 10 / 25 ]                     = 2610
  200 item_armor_percent          [ (467+1) * 200 / 100 ] = 936
                                  [ 218824 * 936 * 20 / 1024 ] + 47 = 4000423
                                  [ 4000423 / 2 ]                        = 2000211
  300 item_mindamage_percent      [ 218824 * 300 * 20   / 1024 ] + 45    = 1282216
  300 item_maxdamage_percent      [ 218824 * 300 * 20   / 1024 ] + 45    = 1282216
  25  item_fastercastrate         [ 218824 * 25  * 156  / 1024 ] + 3876  = 837287
  15  item_skillongethit          [ 218824 * 15  * 768  / 1024 ] + 32000 = 2493770
      (Chilling Armor, skills.txt: mul=768 add=32000)
  12  item_damagetomana           [ 218824 * 12  * 20   / 1024 ] + 43    = 51329
  12  item_hp_perlevel            [ 218824 * 12  * 64   / 1024 ] + 92    = 164210
  30  fireresist                  [ 218824 * 30  * 20   / 1024 ] + 43    = 128260
  30  lightresist                 [ 218824 * 30  * 20   / 1024 ] + 43    = 128260
  30  coldresist                  [ 218824 * 30  * 20   / 1024 ] + 43    = 128260
  30  poisonresist                [ 218824 * 30  * 20   / 1024 ] + 43    = 128260
  1   item_lightradius (El)       [ 218824 * 1   * 51   / 1024 ] + 15    = 10913
  15  armorclass (El)             [ 218824 * 15  * 10   / 1024 ] + 17    = 32071
  7   normal_damage_reduction (Sol) [ 218824 * 7 * 200  / 1024 ] + 188   = 299361
  7   hpregen (Dol)               [ 218824 * 7   * 410  / 1024 ] + 451   = 613756
  5   maxlightresist (Lo)         [ 218824 * 5   * 256  / 1024 ] + 584   = 274114

total = 218824 + 64680 + 149610 + 2610 + 2000211 + 2 * 1282216 + 837287 +
        2493770 + 51329 + 164210 + 4 * 128260 + 10913 + 32071 + 299361 +
        613756 + 274114                                                  = 10290218

scaled_repair_cost = [10290218 * (23-0) / 23] = 10290218
repair_cost = [ 10290218 / 1024 ] * 128 = 1286272  (accurate in-game value: 1285120)


Superior 15/15 Archon Plate Fortitude (0 durability)
====================================================

scaled_armor_cost = [317526 * (524+1) / 524]                             = 318131
socketed_items_cost = [(560 + 14000 + 20160 + 94640) / 2]                = 64680

bonus_costs:
  15  item_armor_percent          [ (524+1) * 15 / 100 ] = 78
                                  [ 318131 * 78 * 20 / 1024 ] + 47 = 484699
                                  [ 484699 / 2 ]                         = 242349
  15  item_maxdurability_percent  [ 60 * 15 / 100 ] = 9
                                  [ 318131 * 9 * 10 / 1024 ] + 117 = 28077
                                  [ 28077 * 10 / 25 ]                    = 11230
  200 item_armor_percent          [ (524+1) * 200 / 100 ] = 1050
                                  [ 318131 * 1050 * 20 / 1024 ] + 47 = 6524217
                                  [ 6524217 / 2 ]                        = 3262108
  300 item_mindamage_percent      [ 318131 * 300 * 20   / 1024 ] + 45    = 1864093
  300 item_maxdamage_percent      [ 318131 * 300 * 20   / 1024 ] + 45    = 1864093
  25  item_fastercastrate         [ 318131 * 25  * 156  / 1024 ] + 3876  = 1215507
  15  item_skillongethit          [ 318131 * 15  * 768  / 1024 ] + 32000 = 3610973
      (Chilling Armor, skills.txt: mul=768 add=32000)
  12  item_damagetomana           [ 318131 * 12  * 20   / 1024 ] + 43    = 74604
  12  item_hp_perlevel            [ 318131 * 12  * 64   / 1024 ] + 92    = 238690
  30  fireresist                  [ 318131 * 30  * 20   / 1024 ] + 43    = 186447
  30  lightresist                 [ 318131 * 30  * 20   / 1024 ] + 43    = 186447
  30  coldresist                  [ 318131 * 30  * 20   / 1024 ] + 43    = 186447
  30  poisonresist                [ 318131 * 30  * 20   / 1024 ] + 43    = 186447
  1   item_lightradius (El)       [ 318131 * 1   * 51   / 1024 ] + 15    = 15859
  15  armorclass (El)             [ 318131 * 15  * 10   / 1024 ] + 17    = 46618
  7   normal_damage_reduction (Sol) [ 318131 * 7 * 200  / 1024 ] + 188   = 435132
  7   hpregen (Dol)               [ 318131 * 7   * 410  / 1024 ] + 451   = 892087
  5   maxlightresist (Lo)         [ 318131 * 5   * 256  / 1024 ] + 584   = 398247

total = 318131 + 64680 + 242349 + 11230 + 3262108 + 2 * 1864093 +
        1215507 + 3610973 + 74604 + 238690 + 4 * 186447 + 15859 +
        46618 + 435132 + 892087 + 398247                                 = 15300189

scaled_repair_cost = [15300189 * (69-0) / 69] = 15300189
repair_cost = [ 15300189 / 1024 ] * 128 = 1912448  (accurate in-game value: 1911808)


Superior 15/15 Dusk Shroud Duress (0 durability)
================================================

scaled_armor_cost = [218357 * (467+1) / 467]                             = 218824
socketed_items_cost = [(16940 + 56000 + 8960) / 2]                       = 40950

bonus_costs:
  15  item_armor_percent          [ (467+1) * 15 / 100 ] = 70
                                  [ 218824 * 70 * 20 / 1024 ] + 47 = 299220
                                  [ 299220 / 2 ]                         = 149610
  15  item_maxdurability_percent  [ 20 * 15 / 100 ] = 3
                                  [ 218824 * 3 * 10 / 1024 ] + 117 = 6527
                                  [ 6527 * 10 / 25 ]                     = 2610
  37  coldmindam                  [ 218824 * 37  * 512  / 1024 ] + 451   = 4048695
  133 coldmaxdam                  [ 218824 * 133 * 340  / 1024 ] + 128   = 9663430
  20  item_mindamage_percent      [ 218824 * 20  * 20   / 1024 ] + 45    = 85523
  20  item_maxdamage_percent      [ 218824 * 20  * 20   / 1024 ] + 45    = 85523
  200 item_armor_percent          [ (467+1) * 200 / 100 ] = 936
                                  [ 218824 * 936 * 20 / 1024 ] + 47 = 4000423
                                  [ 4000423 / 2 ]                        = 2000211
  20  item_fastergethitrate       [ 218824 * 20  * 72   / 1024 ] + 1065  = 308786
  33  item_openwounds             [ 218824 * 33  * 10   / 1024 ] + 23    = 70542
  15  item_crushingblow           [ 218824 * 15  * 40   / 1024 ] + 98    = 128315
  -20 item_staminadrainpct        [ 218824 * -20 * 20   / 1024 ] + 102   = -85376
  20  item_fastergethitrate (Shael) [ 218824 * 20  * 72 / 1024 ] + 1065  = 308786
  15  fireresist (Um)             [ 218824 * 15  * 20   / 1024 ] + 43    = 64151
  15  lightresist (Um)            [ 218824 * 15  * 20   / 1024 ] + 43    = 64151
  15  coldresist (Um)             [ 218824 * 15  * 20   / 1024 ] + 43    = 64151
  15  poisonresist (Um)           [ 218824 * 15  * 20   / 1024 ] + 43    = 64151
  30  coldresist (Thul)           [ 218824 * 30  * 20   / 1024 ] + 43    = 128260

total = 218824 + 40950 + 149610 + 2610 + 4048695 + 9663430 + 2 * 85523 +
        2000211 + 308786 + 70542 + 128315 + -85376 + 308786 + 4 * 64151 +
        128260                                                           = 17411293

scaled_repair_cost = [17411293 * (23-0) / 23] = 17411293
repair_cost = [ 17411293 / 1024 ] * 128 = 2176384  (accurate in-game value: 2181120)


Superior 15/15 Archon Plate Duress (0 durability)
=================================================

scaled_armor_cost = [317526 * (524+1) / 524]                             = 318131
socketed_items_cost = [(16940 + 56000 + 8960) / 2]                       = 40950

bonus_costs:
  15  item_armor_percent          [ (524+1) * 15 / 100 ] = 78
                                  [ 318131 * 78 * 20 / 1024 ] + 47 = 484699
                                  [ 484699 / 2 ]                         = 242349
  15  item_maxdurability_percent  [ 60 * 15 / 100 ] = 9
                                  [ 318131 * 9 * 10 / 1024 ] + 117 = 28077
                                  [ 28077 * 10 / 25 ]                    = 11230
  37  coldmindam                  [ 318131 * 37  * 512  / 1024 ] + 451   = 5885874
  133 coldmaxdam                  [ 318131 * 133 * 340  / 1024 ] + 128   = 14048842
  20  item_mindamage_percent      [ 318131 * 20  * 20   / 1024 ] + 45    = 124314
  20  item_maxdamage_percent      [ 318131 * 20  * 20   / 1024 ] + 45    = 124314
  200 item_armor_percent          [ (524+1) * 200 / 100 ] = 1050
                                  [ 318131 * 1050 * 20 / 1024 ] + 47 = 6524217
                                  [ 6524217 / 2 ]                        = 3262108
  20  item_fastergethitrate       [ 318131 * 20  * 72   / 1024 ] + 1065  = 448436
  33  item_openwounds             [ 318131 * 33  * 10   / 1024 ] + 23    = 102545
  15  item_crushingblow           [ 318131 * 15  * 40   / 1024 ] + 98    = 186502
  -20 item_staminadrainpct        [ 318131 * -20 * 20   / 1024 ] + 102   = -124167
  20  item_fastergethitrate (Shael) [ 318131 * 20  * 72 / 1024 ] + 1065  = 448436
  15  fireresist (Um)             [ 318131 * 15  * 20   / 1024 ] + 43    = 93245
  15  lightresist (Um)            [ 318131 * 15  * 20   / 1024 ] + 43    = 93245
  15  coldresist (Um)             [ 318131 * 15  * 20   / 1024 ] + 43    = 93245
  15  poisonresist (Um)           [ 318131 * 15  * 20   / 1024 ] + 43    = 93245
  30  coldresist (Thul)           [ 318131 * 30  * 20   / 1024 ] + 43    = 186447

total = 318131 + 40950 + 242349 + 11230 + 5885874 + 14048842 +
        2 * 124314 + 3262108 + 448436 + 102545 + 186502 + -124167 +
        448436 + 4 * 93245 + 186447                                      = 25679291

scaled_repair_cost = [25679291 * (69-0) / 69] = 25679291
repair_cost = [ 25679291 / 1024 ] * 128 = 3209856  (accurate in-game value: 3217408)


Superior 15/15 Dusk Shroud Hustle (0 durability)
================================================

scaled_armor_cost = [218357 * (467+1) / 467]                             = 218824
socketed_items_cost = [(16940 + 35840 + 560) / 2]                        = 26670

bonus_costs:
  15  item_armor_percent          [ (467+1) * 15 / 100 ] = 70
                                  [ 218824 * 70 * 20 / 1024 ] + 47 = 299220
                                  [ 299220 / 2 ]                         = 149610
  15  item_maxdurability_percent  [ 20 * 15 / 100 ] = 3
                                  [ 218824 * 3 * 10 / 1024 ] + 117 = 6527
                                  [ 6527 * 10 / 25 ]                     = 2610
  65  item_fastermovevelocity     [ 218824 * 65  * 156  / 1024 ] + 4083  = 2170953
  35  item_staminadrainpct        [ 218824 * 35  * 20   / 1024 ] + 102   = 149688
  40  item_fasterattackrate       [ 218824 * 40  * 156  / 1024 ] + 1042  = 1334500
  10  fireresist                  [ 218824 * 10  * 20   / 1024 ] + 43    = 42782
  10  lightresist                 [ 218824 * 10  * 20   / 1024 ] + 43    = 42782
  10  coldresist                  [ 218824 * 10  * 20   / 1024 ] + 43    = 42782
  10  poisonresist                [ 218824 * 10  * 20   / 1024 ] + 43    = 42782
  6   item_nonclassskill          [ 218824 * 6   * 768  / 1024 ] + 32000 = 1016708
      (Evade, skills.txt: mul=768 add=32000)
  20  item_fastergethitrate (Shael) [ 218824 * 20  * 72 / 1024 ] + 1065  = 308786
  10  dexterity (Ko)              [ 218824 * 10  * 55   / 1024 ] + 125   = 117657
  15  item_staminadrainpct (Eld)  [ 218824 * 15  * 20   / 1024 ] + 102   = 64210

total = 218824 + 26670 + 149610 + 2610 + 2170953 + 149688 + 1334500 +
        4 * 42782 + 1016708 + 308786 + 117657 + 64210                    = 5731344

scaled_repair_cost = [5731344 * (23-0) / 23] = 5731344
repair_cost = [ 5731344 / 1024 ] * 128 = 716416  (accurate in-game value: 715776)


Superior 15/15 Archon Plate Hustle (0 durability)
=================================================

scaled_armor_cost = [317526 * (524+1) / 524]                             = 318131
socketed_items_cost = [(16940 + 35840 + 560) / 2]                        = 26670

bonus_costs:
  15  item_armor_percent          [ (524+1) * 15 / 100 ] = 78
                                  [ 318131 * 78 * 20 / 1024 ] + 47 = 484699
                                  [ 484699 / 2 ]                         = 242349
  15  item_maxdurability_percent  [ 60 * 15 / 100 ] = 9
                                  [ 318131 * 9 * 10 / 1024 ] + 117 = 28077
                                  [ 28077 * 10 / 25 ]                    = 11230
  65  item_fastermovevelocity     [ 318131 * 65  * 156  / 1024 ] + 4083  = 3154325
  35  item_staminadrainpct        [ 318131 * 35  * 20   / 1024 ] + 102   = 217574
  40  item_fasterattackrate       [ 318131 * 40  * 156  / 1024 ] + 1042  = 1939652
  10  fireresist                  [ 318131 * 10  * 20   / 1024 ] + 43    = 62177
  10  lightresist                 [ 318131 * 10  * 20   / 1024 ] + 43    = 62177
  10  coldresist                  [ 318131 * 10  * 20   / 1024 ] + 43    = 62177
  10  poisonresist                [ 318131 * 10  * 20   / 1024 ] + 43    = 62177
  6   item_nonclassskill          [ 318131 * 6   * 768  / 1024 ] + 32000 = 1463589
      (Evade, skills.txt: mul=768 add=32000)
  20  item_fastergethitrate (Shael) [ 318131 * 20  * 72 / 1024 ] + 1065  = 448436
  10  dexterity (Ko)              [ 318131 * 10  * 55   / 1024 ] + 125   = 170996
  15  item_staminadrainpct (Eld)  [ 318131 * 15  * 20   / 1024 ] + 102   = 93304

total = 318131 + 26670 + 242349 + 11230 + 3154325 + 217574 + 1939652 +
        4 * 62177 + 1463589 + 448436 + 170996 + 93304                    = 8334964

scaled_repair_cost = [8334964 * (69-0) / 69] = 8334964
repair_cost = [ 8334964 / 1024 ] * 128 = 1041792  (accurate in-game value: 1041408)


Superior 15/15 Dusk Shroud Treachery (0 durability)
===================================================

scaled_armor_cost = [218357 * (467+1) / 467]                             = 218824
socketed_items_cost = [(16940 + 8960 + 45360) / 2]                       = 35630

bonus_costs:
  15  item_armor_percent          [ (467+1) * 15 / 100 ] = 70
                                  [ 218824 * 70 * 20 / 1024 ] + 47 = 299220
                                  [ 299220 / 2 ]                         = 149610
  15  item_maxdurability_percent  [ 20 * 15 / 100 ] = 3
                                  [ 218824 * 3 * 10 / 1024 ] + 117 = 6527
                                  [ 6527 * 10 / 25 ]                     = 2610
  15  item_skillonhit             [ 218824 * 15  * 896  / 1024 ] + 64000 = 2936065
      (Venom, skills.txt: mul=896 add=64000)
  15  item_skillongethit          [ 218824 * 15  * 640  / 1024 ] + 8000  = 2059475
      (Fade, skills.txt: mul=640 add=8000)
  2   item_addclassskills         [ 218824 * 2   * 1560 / 1024 ] + 49523 = 716252
  45  item_fasterattackrate       [ 218824 * 45  * 156  / 1024 ] + 1042  = 1501183
  20  item_fastergethitrate (Shael) [ 218824 * 20  * 72 / 1024 ] + 1065  = 308786
  30  coldresist (Thul)           [ 218824 * 30  * 20   / 1024 ] + 43    = 128260
  50  item_goldbonus (Lem)        [ 218824 * 50  * 34   / 1024 ] + 187   = 363469

total = 218824 + 35630 + 149610 + 2610 + 2936065 + 2059475 + 716252 +
        1501183 + 308786 + 128260 + 363469                               = 8420164

scaled_repair_cost = [8420164 * (23-0) / 23] = 8420164
repair_cost = [ 8420164 / 1024 ] * 128 = 1052416  (accurate in-game value: 1051648)


Superior 15/15 Archon Plate Treachery (0 durability)
====================================================

scaled_armor_cost = [317526 * (524+1) / 524]                             = 318131
socketed_items_cost = [(16940 + 8960 + 45360) / 2]                       = 35630

bonus_costs:
  15  item_armor_percent          [ (524+1) * 15 / 100 ] = 78
                                  [ 318131 * 78 * 20 / 1024 ] + 47 = 484699
                                  [ 484699 / 2 ]                         = 242349
  15  item_maxdurability_percent  [ 60 * 15 / 100 ] = 9
                                  [ 318131 * 9 * 10 / 1024 ] + 117 = 28077
                                  [ 28077 * 10 / 25 ]                    = 11230
  15  item_skillonhit             [ 318131 * 15  * 896  / 1024 ] + 64000 = 4239469
      (Venom, skills.txt: mul=896 add=64000)
  15  item_skillongethit          [ 318131 * 15  * 640  / 1024 ] + 8000  = 2990478
      (Fade, skills.txt: mul=640 add=8000)
  2   item_addclassskills         [ 318131 * 2   * 1560 / 1024 ] + 49523 = 1018828
  45  item_fasterattackrate       [ 318131 * 45  * 156  / 1024 ] + 1042  = 2181979
  20  item_fastergethitrate (Shael) [ 318131 * 20  * 72 / 1024 ] + 1065  = 448436
  30  coldresist (Thul)           [ 318131 * 30  * 20   / 1024 ] + 43    = 186447
  50  item_goldbonus (Lem)        [ 318131 * 50  * 34   / 1024 ] + 187   = 528334

total = 318131 + 35630 + 242349 + 11230 + 4239469 + 2990478 + 1018828 +
        2181979 + 448436 + 186447 + 528334                               = 12201311

scaled_repair_cost = [12201311 * (69-0) / 69] = 12201311
repair_cost = [ 12201311 / 1024 ] * 128 = 1525120  (accurate in-game value: 1524736)


Superior 15/15 Dusk Shroud Smoke (0 durability)
===============================================

scaled_armor_cost = [218357 * (467+1) / 467]                             = 218824
socketed_items_cost = [(1260 + 31500) / 2]                               = 16380

bonus_costs:
  15  item_armor_percent          [ (467+1) * 15 / 100 ] = 70
                                  [ 218824 * 70 * 20 / 1024 ] + 47 = 299220
                                  [ 299220 / 2 ]                         = 149610
  15  item_maxdurability_percent  [ 20 * 15 / 100 ] = 3
                                  [ 218824 * 3 * 10 / 1024 ] + 117 = 6527
                                  [ 6527 * 10 / 25 ]                     = 2610
  250 armorclass_vs_missile       [ 218824 * 250 * 5    / 1024 ] + 11    = 267130
  75  item_armor_percent          [ (467+1) * 75 / 100 ] = 351
                                  [ 218824 * 351 * 20 / 1024 ] + 47 = 1500188
                                  [ 1500188 / 2 ]                        = 750094
  50  fireresist                  [ 218824 * 50  * 20   / 1024 ] + 43    = 213738
  50  lightresist                 [ 218824 * 50  * 20   / 1024 ] + 43    = 213738
  50  coldresist                  [ 218824 * 50  * 20   / 1024 ] + 43    = 213738
  50  poisonresist                [ 218824 * 50  * 20   / 1024 ] + 43    = 213738
  20  item_fastergethitrate       [ 218824 * 20  * 72   / 1024 ] + 1065  = 308786
  -1  item_lightradius            [ 218824 * -1  * 51   / 1024 ] + 15    = -10883
  6   item_charged_skill          [ 218824 * 6   * 384  / 1024 ] + 3000  = 495354
      (Weaken, skills.txt: mul=384 add=3000)
  30  armorclass_vs_missile (Nef) [ 218824 * 30  * 5    / 1024 ] + 11    = 32065
  10  energy (Lum)                [ 218824 * 10  * 55   / 1024 ] + 100   = 117632

total = 218824 + 16380 + 149610 + 2610 + 267130 + 750094 +
        4 * 213738 + 308786 + -10883 + 495354 + 32065 + 117632           = 3202554

scaled_repair_cost = [3202554 * (23-0) / 23] = 3202554
repair_cost = [ 3202554 / 1024 ] * 128 = 400256  (accurate in-game value: 399360)


Superior 15/15 Archon Plate Smoke (0 durability)
================================================

scaled_armor_cost = [317526 * (524+1) / 524]                             = 318131
socketed_items_cost = [(1260 + 31500) / 2]                               = 16380

bonus_costs:
  15  item_armor_percent          [ (524+1) * 15 / 100 ] = 78
                                  [ 318131 * 78 * 20 / 1024 ] + 47 = 484699
                                  [ 484699 / 2 ]                         = 242349
  15  item_maxdurability_percent  [ 60 * 15 / 100 ] = 9
                                  [ 318131 * 9 * 10 / 1024 ] + 117 = 28077
                                  [ 28077 * 10 / 25 ]                    = 11230
  250 armorclass_vs_missile       [ 318131 * 250 * 5    / 1024 ] + 11    = 388354
  75  item_armor_percent          [ (524+1) * 75 / 100 ] = 393
                                  [ 318131 * 393 * 20 / 1024 ] + 47 = 2441950
                                  [ 2441950 / 2 ]                        = 1220975
  50  fireresist                  [ 318131 * 50  * 20   / 1024 ] + 43    = 310717
  50  lightresist                 [ 318131 * 50  * 20   / 1024 ] + 43    = 310717
  50  coldresist                  [ 318131 * 50  * 20   / 1024 ] + 43    = 310717
  50  poisonresist                [ 318131 * 50  * 20   / 1024 ] + 43    = 310717
  20  item_fastergethitrate       [ 318131 * 20  * 72   / 1024 ] + 1065  = 448436
  -1  item_lightradius            [ 318131 * -1  * 51   / 1024 ] + 15    = -15829
  6   item_charged_skill          [ 318131 * 6   * 384  / 1024 ] + 3000  = 718794
      (Weaken, skills.txt: mul=384 add=3000)
  30  armorclass_vs_missile (Nef) [ 318131 * 30  * 5    / 1024 ] + 11    = 46612
  10  energy (Lum)                [ 318131 * 10  * 55   / 1024 ] + 100   = 170971

total = 318131 + 16380 + 242349 + 11230 + 388354 + 1220975 +
        4 * 310717 + 448436 + -15829 + 718794 + 46612 + 170971           = 4809271

scaled_repair_cost = [4809271 * (69-0) / 69] = 4809271
repair_cost = [ 4809271 / 1024 ] * 128 = 601088  (accurate in-game value: 601088)


Superior 15/15 Mage Plate Smoke (0 durability)
==============================================

scaled_armor_cost = [118407 * (261+1) / 261]                             = 118860
socketed_items_cost = [(1260 + 31500) / 2]                               = 16380

bonus_costs:
  15  item_armor_percent          [ (261+1) * 15 / 100 ] = 39
                                  [ 118860 * 39 * 20 / 1024 ] + 47 = 90584
                                  [ 90584 / 2 ]                          = 45292
  15  item_maxdurability_percent  [ 60 * 15 / 100 ] = 9
                                  [ 118860 * 9 * 10 / 1024 ] + 117 = 10563
                                  [ 10563 * 10 / 25 ]                    = 4225
  250 armorclass_vs_missile       [ 118860 * 250 * 5    / 1024 ] + 11    = 145103
  75  item_armor_percent          [ (261+1) * 75 / 100 ] = 196
                                  [ 118860 * 196 * 20 / 1024 ] + 47 = 455057
                                  [ 455057 / 2 ]                         = 227528
  50  fireresist                  [ 118860 * 50  * 20   / 1024 ] + 43    = 116117
  50  lightresist                 [ 118860 * 50  * 20   / 1024 ] + 43    = 116117
  50  coldresist                  [ 118860 * 50  * 20   / 1024 ] + 43    = 116117
  50  poisonresist                [ 118860 * 50  * 20   / 1024 ] + 43    = 116117
  20  item_fastergethitrate       [ 118860 * 20  * 72   / 1024 ] + 1065  = 168211
  -1  item_lightradius            [ 118860 * -1  * 51   / 1024 ] + 15    = -5904
  6   item_charged_skill          [ 118860 * 6   * 384  / 1024 ] + 3000  = 270435
      (Weaken, skills.txt: mul=384 add=3000)
  30  armorclass_vs_missile (Nef) [ 118860 * 30  * 5    / 1024 ] + 11    = 17422
  10  energy (Lum)                [ 118860 * 10  * 55   / 1024 ] + 100   = 63940

total = 118860 + 16380 + 45292 + 4225 + 145103 + 227528 +
        4 * 116117 + 168211 + -5904 + 270435 + 17422 + 63940             = 1535960

scaled_repair_cost = [1535960 * (69-0) / 69] = 1535960
repair_cost = [ 1535960 / 1024 ] * 128 = 191872  (accurate in-game value: 191488)


Superior 15/15 Dusk Shroud Stealth (0 durability)
=================================================

scaled_armor_cost = [218357 * (467+1) / 467]                             = 218824
socketed_items_cost = [(3500 + 2240) / 2]                                = 2870

bonus_costs:
  15  item_armor_percent          [ (467+1) * 15 / 100 ] = 70
                                  [ 218824 * 70 * 20 / 1024 ] + 47 = 299220
                                  [ 299220 / 2 ]                         = 149610
  15  item_maxdurability_percent  [ 20 * 15 / 100 ] = 3
                                  [ 218824 * 3 * 10 / 1024 ] + 117 = 6527
                                  [ 6527 * 10 / 25 ]                     = 2610
  3   magic_damage_reduction      [ 218824 * 3   * 340  / 1024 ] + 397   = 218366
  6   dexterity                   [ 218824 * 6   * 55   / 1024 ] + 125   = 70644
  15  maxstamina                  [ 218824 * 15  * 20   / 1024 ] + 75    = 64183
  25  item_fastermovevelocity     [ 218824 * 25  * 156  / 1024 ] + 4083  = 837494
  25  item_fastercastrate         [ 218824 * 25  * 156  / 1024 ] + 3876  = 837287
  25  item_fastergethitrate       [ 218824 * 25  * 72   / 1024 ] + 1065  = 385716
  30  poisonresist (Tal)          [ 218824 * 30  * 20   / 1024 ] + 43    = 128260
  15  manarecoverybonus (Eth)                                            = 0

total = 218824 + 2870 + 149610 + 2610 + 218366 + 70644 + 64183 +
        837494 + 837287 + 385716 + 128260                                = 2915864

scaled_repair_cost = [2915864 * (23-0) / 23] = 2915864
repair_cost = [ 2915864 / 1024 ] * 128 = 364416  (accurate in-game value: 363520)


Superior 15/15 Mage Plate Stealth (0 durability)
================================================

scaled_armor_cost = [118407 * (261+1) / 261]                             = 118860
socketed_items_cost = [(3500 + 2240) / 2]                                = 2870

bonus_costs:
  15  item_armor_percent          [ (261+1) * 15 / 100 ] = 39
                                  [ 118860 * 39 * 20 / 1024 ] + 47 = 90584
                                  [ 90584 / 2 ]                          = 45292
  15  item_maxdurability_percent  [ 60 * 15 / 100 ] = 9
                                  [ 118860 * 9 * 10 / 1024 ] + 117 = 10563
                                  [ 10563 * 10 / 25 ]                    = 4225
  3   magic_damage_reduction      [ 118860 * 3   * 340  / 1024 ] + 397   = 118792
  6   dexterity                   [ 118860 * 6   * 55   / 1024 ] + 125   = 38429
  15  maxstamina                  [ 118860 * 15  * 20   / 1024 ] + 75    = 34897
  25  item_fastermovevelocity     [ 118860 * 25  * 156  / 1024 ] + 4083  = 456772
  25  item_fastercastrate         [ 118860 * 25  * 156  / 1024 ] + 3876  = 456565
  25  item_fastergethitrate       [ 118860 * 25  * 72   / 1024 ] + 1065  = 209998
  30  poisonresist (Tal)          [ 118860 * 30  * 20   / 1024 ] + 43    = 69687
  15  manarecoverybonus (Eth)                                            = 0

total = 118860 + 2870 + 45292 + 4225 + 118792 + 38429 + 34897 +
        456772 + 456565 + 209998 + 69687                                 = 1556387

scaled_repair_cost = [1556387 * (69-0) / 69] = 1556387
repair_cost = [ 1556387 / 1024 ] * 128 = 194432  (accurate in-game value: 193536)


Superior 15/15 Breast Plate Stealth (0 durability)
=================================================

scaled_armor_cost = [10078 * (68+1) / 68]                                = 10226
socketed_items_cost = [(3500 + 2240) / 2]                                = 2870

bonus_costs:
  15  item_armor_percent          [ (68+1) * 15 / 100 ] = 10
                                  [ 10226 * 10 * 20 / 1024 ] + 47 = 2044
                                  [ 2044 / 2 ]                           = 1022
  15  item_maxdurability_percent  [ 50 * 15 / 100 ] = 7
                                  [ 10226 * 7 * 10 / 1024 ] + 117 = 816
                                  [ 816 * 10 / 25 ]                      = 326
  3   magic_damage_reduction      [ 10226  * 3   * 340  / 1024 ] + 397   = 10583
  6   dexterity                   [ 10226  * 6   * 55   / 1024 ] + 125   = 3420
  15  maxstamina                  [ 10226  * 15  * 20   / 1024 ] + 75    = 3070
  25  item_fastermovevelocity     [ 10226  * 25  * 156  / 1024 ] + 4083  = 43029
  25  item_fastercastrate         [ 10226  * 25  * 156  / 1024 ] + 3876  = 42822
  25  item_fastergethitrate       [ 10226  * 25  * 72   / 1024 ] + 1065  = 19040
  30  poisonresist (Tal)          [ 10226  * 30  * 20   / 1024 ] + 43    = 6034
  15  manarecoverybonus (Eth)                                            = 0

total = 10226 + 2870 + 1022 + 326 + 10583 + 3420 + 3070 + 43029 +
        42822 + 19040 + 6034                                             = 142442

scaled_repair_cost = [142442 * (57-0) / 57] = 142442
repair_cost = [ 142442 / 1024 ] * 128 = 17792  (accurate in-game value: 17664)


Superior 15/15 45@ Sacred Targe Phoenix (0 durability)
======================================================

scaled_armor_cost = [72618 * (158+1) / 158]                              = 73077
socketed_items_cost = [(80640 + 80640 + 94640 + 117740) / 2]             = 186830

bonus_costs:
  15  item_armor_percent          [ (158+1) * 15 / 100 ] = 23
                                  [ 73077 * 23 * 20 / 1024 ] + 47 = 32874
                                  [ 32874 / 2 ]                          = 16437
  15  item_maxdurability_percent  [ 45 * 15 / 100 ] = 6
                                  [ 73077 * 6 * 10 / 1024 ] + 117 = 4398
                                  [ 4398 * 10 / 25 ]                     = 1759
  45  fireresist (automod)        [ 73077  * 45  * 20   / 1024 ] + 43    = 64270
  45  lightresist (automod)       [ 73077  * 45  * 20   / 1024 ] + 43    = 64270
  45  coldresist (automod)        [ 73077  * 45  * 20   / 1024 ] + 43    = 64270
  45  poisonresist (automod)      [ 73077  * 45  * 20   / 1024 ] + 43    = 64270
  400 item_mindamage_percent      [ 73077  * 400 * 20   / 1024 ] + 45    = 570959
  400 item_maxdamage_percent      [ 73077  * 400 * 20   / 1024 ] + 45    = 570959
  400 armorclass_vs_missile       [ 73077  * 400 * 5    / 1024 ] + 11    = 142739
  22  item_skillonhit             [ 73077  * 22  * 256  / 1024 ] + 1000  = 402923
      (Firestorm, skills.txt: mul=256 add=1000)
  40  item_skillonlevelup         [ 73077  * 40  * 512  / 1024 ] + 8000  = 1469540
      (Blaze, skills.txt: mul=512 add=8000)
  28  passive_fire_pierce         [ 73077  * 28  * 2578 / 1024 ] + 0     = 5151357
  15  item_aura (Level 10-15 Redemption Aura When Equipped)              = 0
  21  item_absorbfire             [ 73077  * 21  * 204  / 1024 ] + 1739  = 307463
  5   maxfireresist (Vex)         [ 73077  * 5   * 256  / 1024 ] + 584   = 91930
  5   maxfireresist (Vex)         [ 73077  * 5   * 256  / 1024 ] + 584   = 91930
  5   maxlightresist (Lo)         [ 73077  * 5   * 256  / 1024 ] + 584   = 91930
  50  maxhp (Jah)                 [ 73077  * 50  * 20   / 1024 ] + 56    = 71420

total = 73077 + 186830 + 16437 + 1759 + 4 * 64270 + 2 * 570959 +
        142739 + 402923 + 1469540 + 5151357 + 307463 + 3 * 91930 + 71420 = 9498333

scaled_repair_cost = [9498333 * (51-0) / 51] = 9498333
repair_cost = [ 9498333 / 1024 ] * 128 = 1187200  (accurate in-game value: 1186816)


Superior 15/15 Monarch Phoenix (0 durability)
=============================================

scaled_armor_cost = [81896 * (148+1) / 148]                              = 82449
socketed_items_cost = [(80640 + 80640 + 94640 + 117740) / 2]             = 186830

bonus_costs:
  15  item_armor_percent          [ (148+1) * 15 / 100 ] = 22
                                  [ 82449 * 22 * 20 / 1024 ] + 47 = 35474
                                  [ 35474 / 2 ]                          = 17737
  15  item_maxdurability_percent  [ 86 * 15 / 100 ] = 12
                                  [ 82449 * 12 * 10 / 1024 ] + 117 = 9778
                                  [ 9778 * 10 / 25 ]                     = 3911
  400 item_mindamage_percent      [ 82449  * 400 * 20   / 1024 ] + 45    = 644177
  400 item_maxdamage_percent      [ 82449  * 400 * 20   / 1024 ] + 45    = 644177
  400 armorclass_vs_missile       [ 82449  * 400 * 5    / 1024 ] + 11    = 161044
  22  item_skillonhit             [ 82449  * 22  * 256  / 1024 ] + 1000  = 454469
      (Firestorm, skills.txt: mul=256 add=1000)
  40  item_skillonlevelup         [ 82449  * 40  * 512  / 1024 ] + 8000  = 1656980
      (Blaze, skills.txt: mul=512 add=8000)
  28  passive_fire_pierce         [ 82449  * 28  * 2578 / 1024 ] + 0     = 5812010
  15  item_aura (Level 10-15 Redemption Aura When Equipped)              = 0
  21  item_absorbfire             [ 82449  * 21  * 204  / 1024 ] + 1739  = 346672
  5   maxfireresist (Vex)         [ 82449  * 5   * 256  / 1024 ] + 584   = 103645
  5   maxfireresist (Vex)         [ 82449  * 5   * 256  / 1024 ] + 584   = 103645
  5   maxlightresist (Lo)         [ 82449  * 5   * 256  / 1024 ] + 584   = 103645
  50  maxhp (Jah)                 [ 82449  * 50  * 20   / 1024 ] + 56    = 80572

total = 82449 + 186830 + 17737 + 3911 + 2 * 644177 + 161044 + 454469 +
        1656980 + 5812010 + 346672 + 3 * 103645 + 80572                  = 10401963

scaled_repair_cost = [10401963 * (98-0) / 98] = 10401963
repair_cost = [ 10401963 / 1024 ] * 128 = 1300224  (accurate in-game value: 1299456)


Superior 15/15 45@ Sacred Targe Dream (0 durability)
====================================================

scaled_armor_cost = [72618 * (158+1) / 158]                              = 73077
socketed_items_cost = [(27440 + 117740 + 50540) / 2]                     = 97860

bonus_costs:
  15  item_armor_percent          [ (158+1) * 15 / 100 ] = 23
                                  [ 73077 * 23 * 20 / 1024 ] + 47 = 32874
                                  [ 32874 / 2 ]                          = 16437
  15  item_maxdurability_percent  [ 45 * 15 / 100 ] = 6
                                  [ 73077 * 6 * 10 / 1024 ] + 117 = 4398
                                  [ 4398 * 10 / 25 ]                     = 1759
  45  fireresist (automod)        [ 73077  * 45  * 20   / 1024 ] + 43    = 64270
  45  lightresist (automod)       [ 73077  * 45  * 20   / 1024 ] + 43    = 64270
  45  coldresist (automod)        [ 73077  * 45  * 20   / 1024 ] + 43    = 64270
  45  poisonresist (automod)      [ 73077  * 45  * 20   / 1024 ] + 43    = 64270
  220 armorclass                  [ 73077  * 220 * 10   / 1024 ] + 17    = 157018
  15  item_skillongethit          [ 73077  * 15  * 640  / 1024 ] + 16000 = 701096
      (Confuse, skills.txt: mul=640 add=16000)
  5   item_mana_perlevel          [ 73077  * 5   * 128  / 1024 ] + 90    = 45763
  20  fireresist                  [ 73077  * 20  * 20   / 1024 ] + 43    = 28588
  20  lightresist                 [ 73077  * 20  * 20   / 1024 ] + 43    = 28588
  20  coldresist                  [ 73077  * 20  * 20   / 1024 ] + 43    = 28588
  20  poisonresist                [ 73077  * 20  * 20   / 1024 ] + 43    = 28588
  30  item_fastergethitrate       [ 73077  * 30  * 72   / 1024 ] + 1065  = 155211
  15  item_aura (Level 15 Holy Shock Aura When Equipped)                 = 0
  25  item_magicbonus             [ 73077  * 25  * 102  / 1024 ] + 577   = 182555
  10  vitality (Io)               [ 73077  * 10  * 55   / 1024 ] + 100   = 39350
  50  maxhp (Jah)                 [ 73077  * 50  * 20   / 1024 ] + 56    = 71420
  30  item_armor_percent (Pul)    [ (158+1) * 30 / 100 ] = 47
                                  [ 73077 * 47 * 20 / 1024 ] + 47 = 67129
                                  [ 67129 / 2 ]                          = 33564

total = 73077 + 97860 + 16437 + 1759 + 4 * 64270 + 157018 + 701096 +
        45763 + 4 * 28588 + 155211 + 182555 + 39350 + 71420 + 33564      = 1946542

scaled_repair_cost = [1946542 * (51-0) / 51] = 1946542
repair_cost = [ 1946542 / 1024 ] * 128 = 243200  (accurate in-game value: 242688)


Superior 15/15 45@ Sacred Targe Dragon (0 durability)
=====================================================

scaled_armor_cost = [72618 * (158+1) / 158]                              = 73077
socketed_items_cost = [(102060 + 94640 + 14000) / 2]                     = 105350

bonus_costs:
  15  item_armor_percent          [ (158+1) * 15 / 100 ] = 23
                                  [ 73077 * 23 * 20 / 1024 ] + 47 = 32874
                                  [ 32874 / 2 ]                          = 16437
  15  item_maxdurability_percent  [ 45 * 15 / 100 ] = 6
                                  [ 73077 * 6 * 10 / 1024 ] + 117 = 4398
                                  [ 4398 * 10 / 25 ]                     = 1759
  45  fireresist (automod)        [ 73077  * 45  * 20   / 1024 ] + 43    = 64270
  45  lightresist (automod)       [ 73077  * 45  * 20   / 1024 ] + 43    = 64270
  45  coldresist (automod)        [ 73077  * 45  * 20   / 1024 ] + 43    = 64270
  45  poisonresist (automod)      [ 73077  * 45  * 20   / 1024 ] + 43    = 64270
  360 armorclass                  [ 73077  * 360 * 10   / 1024 ] + 17    = 256928
  230 armorclass_vs_missile       [ 73077  * 230 * 5    / 1024 ] + 11    = 82079
  3   item_strength_perlevel      [ 73077  * 3   * 128  / 1024 ] + 132   = 27535
  15  item_skillonhit             [ 73077  * 15  * 896  / 1024 ] + 64000 = 1023135
      (Hydra, skills.txt: mul=896 add=64000)
  18  item_skillongethit          [ 73077  * 18  * 896  / 1024 ] + 64000 = 1214962
      (Venom, skills.txt: mul=896 add=64000)
  14  item_aura (Level 14 Holy Fire Aura When Equipped)                  = 0
  5   strength                    [ 73077  * 5   * 55   / 1024 ] + 125   = 19750
  5   dexterity                   [ 73077  * 5   * 55   / 1024 ] + 125   = 19750
  5   vitality                    [ 73077  * 5   * 55   / 1024 ] + 100   = 19725
  5   energy                      [ 73077  * 5   * 55   / 1024 ] + 100   = 19725
  50  maxmana (Sur)               [ 73077  * 50  * 20   / 1024 ] + 81    = 71445
  5   maxlightresist (Lo)         [ 73077  * 5   * 256  / 1024 ] + 584   = 91930
  7   normal_damage_reduction (Sol) [ 73077 * 7 * 200  / 1024 ] + 188   = 100097

total = 73077 + 105350 + 16437 + 1759 + 4 * 64270 + 256928 + 82079 +
        27535 + 1023135 + 1214962 + 2 * 19750 + 2 * 19725 + 71445 +
        91930 + 100097                                                   = 3400764

scaled_repair_cost = [3400764 * (51-0) / 51] = 3400764
repair_cost = [ 3400764 / 1024 ] * 128 = 425088  (accurate in-game value: 423936)


Superior 15/15 45@ Sacred Targe Spirit (0 durability)
=====================================================

scaled_armor_cost = [72618 * (158+1) / 158]                              = 73077
socketed_items_cost = [(3500 + 8960 + 6860 + 11340) / 2]                 = 15330

bonus_costs:
  15  item_armor_percent          [ (158+1) * 15 / 100 ] = 23
                                  [ 73077 * 23 * 20 / 1024 ] + 47 = 32874
                                  [ 32874 / 2 ]                          = 16437
  15  item_maxdurability_percent  [ 45 * 15 / 100 ] = 6
                                  [ 73077 * 6 * 10 / 1024 ] + 117 = 4398
                                  [ 4398 * 10 / 25 ]                     = 1759
  45  fireresist (automod)        [ 73077  * 45  * 20   / 1024 ] + 43    = 64270
  45  lightresist (automod)       [ 73077  * 45  * 20   / 1024 ] + 43    = 64270
  45  coldresist (automod)        [ 73077  * 45  * 20   / 1024 ] + 43    = 64270
  45  poisonresist (automod)      [ 73077  * 45  * 20   / 1024 ] + 43    = 64270
  55  item_fastergethitrate       [ 73077  * 55  * 72   / 1024 ] + 1065  = 283667
  112 maxmana                     [ 73077  * 112 * 20   / 1024 ] + 81    = 159936
  250 armorclass_vs_missile       [ 73077  * 250 * 5    / 1024 ] + 11    = 89216
  22  vitality                    [ 73077  * 22  * 55   / 1024 ] + 100   = 86450
  35  item_fastercastrate         [ 73077  * 35  * 156  / 1024 ] + 3876  = 393524
  8   item_absorbmagic            [ 73077  * 8   * 204  / 1024 ] + 1739  = 118205
  2   item_allskills              [ 73077  * 2   * 4096 / 1024 ] + 15123 = 599739
  35  poisonresist (Tal)          [ 73077  * 35  * 20   / 1024 ] + 43    = 49997
  35  coldresist (Thul)           [ 73077  * 35  * 20   / 1024 ] + 43    = 49997
  35  lightresist (Ort)           [ 73077  * 35  * 20   / 1024 ] + 43    = 49997
  14  item_attackertakesdamage (Amn) [ 73077 * 14 * 128 / 1024 ] + 112   = 127996

total = 73077 + 15330 + 16437 + 1759 + 4 * 64270 + 283667 + 159936 +
        89216 + 86450 + 393524 + 118205 + 599739 + 3 * 49997 + 127996    = 2372407

scaled_repair_cost = [2372407 * (51-0) / 51] = 2372407
repair_cost = [ 2372407 / 1024 ] * 128 = 296448  (accurate in-game value: 295936)


Superior 15/15 45@ Protector Shield Spirit (0 durability)
=========================================================

scaled_armor_cost = [52947 * (153+1) / 153]                              = 53293
socketed_items_cost = [(3500 + 8960 + 6860 + 11340) / 2]                 = 15330

bonus_costs:
  15  item_armor_percent          [ (153+1) * 15 / 100 ] = 23
                                  [ 53293 * 23 * 20 / 1024 ] + 47 = 23987
                                  [ 23987 / 2 ]                          = 11993
  15  item_maxdurability_percent  [ 40 * 15 / 100 ] = 6
                                  [ 53293 * 6 * 10 / 1024 ] + 117 = 3239
                                  [ 3239 * 10 / 25 ]                     = 1295
  45  fireresist (automod)        [ 53293  * 45  * 20   / 1024 ] + 43    = 46882
  45  lightresist (automod)       [ 53293  * 45  * 20   / 1024 ] + 43    = 46882
  45  coldresist (automod)        [ 53293  * 45  * 20   / 1024 ] + 43    = 46882
  45  poisonresist (automod)      [ 53293  * 45  * 20   / 1024 ] + 43    = 46882
  55  item_fastergethitrate       [ 53293  * 55  * 72   / 1024 ] + 1065  = 207159
  112 maxmana                     [ 53293  * 112 * 20   / 1024 ] + 81    = 116659
  250 armorclass_vs_missile       [ 53293  * 250 * 5    / 1024 ] + 11    = 65065
  22  vitality                    [ 53293  * 22  * 55   / 1024 ] + 100   = 63073
  35  item_fastercastrate         [ 53293  * 35  * 156  / 1024 ] + 3876  = 288035
  8   item_absorbmagic            [ 53293  * 8   * 204  / 1024 ] + 1739  = 86674
  2   item_allskills              [ 53293  * 2   * 4096 / 1024 ] + 15123 = 441467
  35  poisonresist (Tal)          [ 53293  * 35  * 20   / 1024 ] + 43    = 36473
  35  coldresist (Thul)           [ 53293  * 35  * 20   / 1024 ] + 43    = 36473
  35  lightresist (Ort)           [ 53293  * 35  * 20   / 1024 ] + 43    = 36473
  14  item_attackertakesdamage (Amn) [ 53293 * 14 * 128 / 1024 ] + 112   = 93374

total = 53293 + 15330 + 11993 + 1295 + 4 * 46882 + 207159 + 116659 +
        65065 + 63073 + 288035 + 86674 + 441467 + 3 * 36473 + 93374      = 1740364

scaled_repair_cost = [1740364 * (46-0) / 46] = 1740364
repair_cost = [ 1740364 / 1024 ] * 128 = 217472  (accurate in-game value: 217088)


Superior 15/15 Monarch Spirit (0 durability)
============================================

scaled_armor_cost = [81896 * (148+1) / 148]                              = 82449
socketed_items_cost = [(3500 + 8960 + 6860 + 11340) / 2]                 = 15330

bonus_costs:
  15  item_armor_percent          [ (148+1) * 15 / 100 ] = 22
                                  [ 82449 * 22 * 20 / 1024 ] + 47 = 35474
                                  [ 35474 / 2 ]                          = 17737
  15  item_maxdurability_percent  [ 86 * 15 / 100 ] = 12
                                  [ 82449 * 12 * 10 / 1024 ] + 117 = 9778
                                  [ 9778 * 10 / 25 ]                     = 3911
  55  item_fastergethitrate       [ 82449  * 55  * 72   / 1024 ] + 1065  = 319910
  112 maxmana                     [ 82449  * 112 * 20   / 1024 ] + 81    = 180438
  250 armorclass_vs_missile       [ 82449  * 250 * 5    / 1024 ] + 11    = 100656
  22  vitality                    [ 82449  * 22  * 55   / 1024 ] + 100   = 97525
  35  item_fastercastrate         [ 82449  * 35  * 156  / 1024 ] + 3876  = 443496
  8   item_absorbmagic            [ 82449  * 8   * 204  / 1024 ] + 1739  = 133142
  2   item_allskills              [ 82449  * 2   * 4096 / 1024 ] + 15123 = 674715
  35  poisonresist (Tal)          [ 82449  * 35  * 20   / 1024 ] + 43    = 56404
  35  coldresist (Thul)           [ 82449  * 35  * 20   / 1024 ] + 43    = 56404
  35  lightresist (Ort)           [ 82449  * 35  * 20   / 1024 ] + 43    = 56404
  14  item_attackertakesdamage (Amn) [ 82449 * 14 * 128 / 1024 ] + 112   = 144397

total = 82449 + 15330 + 17737 + 3911 + 319910 + 180438 + 100656 +
        97525 + 443496 + 133142 + 674715 + 3 * 56404 + 144397            = 2382918

scaled_repair_cost = [2382918 * (98-0) / 98] = 2382918
repair_cost = [ 2382918 / 1024 ] * 128 = 297856  (accurate in-game value: 296960)


Superior 15/15 45@ Sacred Targe Ancient's Pledge (0 durability)
===============================================================

scaled_armor_cost = [72618 * (158+1) / 158]                              = 73077
socketed_items_cost = [(5040 + 6860 + 3500) / 2]                         = 7700

bonus_costs:
  15  item_armor_percent          [ (158+1) * 15 / 100 ] = 23
                                  [ 73077  * 23 * 20 / 1024 ] + 47 = 32874
                                  [ 32874 / 2 ]                          = 16437
  15  item_maxdurability_percent  [ 45 * 15 / 100 ] = 6
                                  [ 73077 * 6 * 10 / 1024 ] + 117 = 4398
                                  [ 4398 * 10 / 25 ]                     = 1759
  45  fireresist (automod)        [ 73077  * 45  * 20   / 1024 ] + 43    = 64270
  45  lightresist (automod)       [ 73077  * 45  * 20   / 1024 ] + 43    = 64270
  45  coldresist (automod)        [ 73077  * 45  * 20   / 1024 ] + 43    = 64270
  45  poisonresist (automod)      [ 73077  * 45  * 20   / 1024 ] + 43    = 64270
  30  coldresist                  [ 73077  * 30  * 20   / 1024 ] + 43    = 42861
  13  fireresist                  [ 73077  * 13  * 20   / 1024 ] + 43    = 18597
  13  lightresist                 [ 73077  * 13  * 20   / 1024 ] + 43    = 18597
  13  coldresist                  [ 73077  * 13  * 20   / 1024 ] + 43    = 18597
  13  poisonresist                [ 73077  * 13  * 20   / 1024 ] + 43    = 18597
  50  item_armor_percent          [ (158+1) * 50 / 100 ] = 79
                                  [ 73077 * 79 * 20 / 1024 ] + 47 = 112802
                                  [ 112802 / 2 ]                         = 56401
  10  item_damagetomana           [ 73077  * 10  * 20   / 1024 ] + 43    = 14315
  35  fireresist (Ral)            [ 73077  * 35  * 20   / 1024 ] + 43    = 49997
  35  lightresist (Ort)           [ 73077  * 35  * 20   / 1024 ] + 43    = 49997
  35  poisonresist (Tal)          [ 73077  * 35  * 20   / 1024 ] + 43    = 49997

total = 73077 + 7700 + 16437 + 1759 + 4 * 64270 + 42861 + 4 * 18597 +
        56401 + 14315 + 3 * 49997                                        = 694009

scaled_repair_cost = [694009 * (51-0) / 51] = 694009
repair_cost = [ 694009 / 1024 ] * 128 = 86656  (accurate in-game value: 86016)


Superior 15/15 45@ Protector Shield Ancient's Pledge (0 durability)
===================================================================

scaled_armor_cost = [52947 * (153+1) / 153]                              = 53293
socketed_items_cost = [(5040 + 6860 + 3500) / 2]                         = 7700

bonus_costs:
  15  item_armor_percent          [ (153+1) * 15 / 100 ] = 23
                                  [ 53293  * 23 * 20 / 1024 ] + 47 = 23987
                                  [ 23987 / 2 ]                          = 11993
  15  item_maxdurability_percent  [ 40 * 15 / 100 ] = 6
                                  [ 53293 * 6 * 10 / 1024 ] + 117 = 3239
                                  [ 3239 * 10 / 25 ]                     = 1295
  45  fireresist (automod)        [ 53293  * 45  * 20   / 1024 ] + 43    = 46882
  45  lightresist (automod)       [ 53293  * 45  * 20   / 1024 ] + 43    = 46882
  45  coldresist (automod)        [ 53293  * 45  * 20   / 1024 ] + 43    = 46882
  45  poisonresist (automod)      [ 53293  * 45  * 20   / 1024 ] + 43    = 46882
  30  coldresist                  [ 53293  * 30  * 20   / 1024 ] + 43    = 31269
  13  fireresist                  [ 53293  * 13  * 20   / 1024 ] + 43    = 13574
  13  lightresist                 [ 53293  * 13  * 20   / 1024 ] + 43    = 13574
  13  coldresist                  [ 53293  * 13  * 20   / 1024 ] + 43    = 13574
  13  poisonresist                [ 53293  * 13  * 20   / 1024 ] + 43    = 13574
  50  item_armor_percent          [ (153+1) * 50 / 100 ] = 77
                                  [ 53293 * 77 * 20 / 1024 ] + 47 = 80194
                                  [ 80194 / 2 ]                          = 40097
  10  item_damagetomana           [ 53293  * 10  * 20   / 1024 ] + 43    = 10451
  35  fireresist (Ral)            [ 53293  * 35  * 20   / 1024 ] + 43    = 36473
  35  lightresist (Ort)           [ 53293  * 35  * 20   / 1024 ] + 43    = 36473
  35  poisonresist (Tal)          [ 53293  * 35  * 20   / 1024 ] + 43    = 36473

total = 53293 + 7700 + 11993 + 1295 + 4 * 46882 + 31269 + 4 * 13574 +
        40097 + 10451 + 3 * 36473                                        = 507341

scaled_repair_cost = [507341 * (46-0) / 46] = 507341
repair_cost = [ 507341 / 1024 ] * 128 = 63360  (accurate in-game value: 63360)


Superior 15/15 Large Shield Ancient's Pledge (0 durability)
===========================================================

scaled_armor_cost = [1214 * (14+1) / 14]                                 = 1300
socketed_items_cost = [(5040 + 6860 + 3500) / 2]                         = 7700

bonus_costs:
  15  item_armor_percent          [ (14+1) * 15 / 100 ] = 2
                                  [ 1300 * 2 * 20 / 1024 ] + 47 = 97
                                  [ 97 / 2 ]                             = 48
  15  item_maxdurability_percent  [ 24 * 15 / 100 ] = 3
                                  [ 1300 * 3 * 10 / 1024 ] + 117 = 155
                                  [ 155 * 10 / 25 ]                      = 62
  30  coldresist                  [ 1300   * 30  * 20   / 1024 ] + 43    = 804
  13  fireresist                  [ 1300   * 13  * 20   / 1024 ] + 43    = 373
  13  lightresist                 [ 1300   * 13  * 20   / 1024 ] + 43    = 373
  13  coldresist                  [ 1300   * 13  * 20   / 1024 ] + 43    = 373
  13  poisonresist                [ 1300   * 13  * 20   / 1024 ] + 43    = 373
  50  item_armor_percent          [ (14+1) * 50 / 100 ] = 7
                                  [ 1300 * 7 * 20 / 1024 ] + 47 = 224
                                  [ 224 / 2 ]                            = 112
  10  item_damagetomana           [ 1300   * 10  * 20   / 1024 ] + 43    = 296
  35  fireresist (Ral)            [ 1300   * 35  * 20   / 1024 ] + 43    = 931
  35  lightresist (Ort)           [ 1300   * 35  * 20   / 1024 ] + 43    = 931
  35  poisonresist (Tal)          [ 1300   * 35  * 20   / 1024 ] + 43    = 931

total = 1300 + 7700 + 48 + 62 + 804 + 4 * 373 + 112 + 296 + 3 * 931      = 14607

scaled_repair_cost = [14607 * (27-0) / 27] = 14607
repair_cost = [ 14607 * 128 / 1024 ] = 1825  (accurate in-game value: 1796)


Superior 15/15 Bone Visage Dream (0 durability)
===============================================

scaled_armor_cost = [87274 * (157+1) / 157]                              = 87829
socketed_items_cost = [(27440 + 117740 + 50540) / 2]                     = 97860

bonus_costs:
  15  item_armor_percent          [ (157+1) * 15 / 100 ] = 23
                                  [ 87829 * 23 * 20 / 1024 ] + 47 = 39501
                                  [ 39501 / 2 ]                          = 19750
  15  item_maxdurability_percent  [ 40 * 15 / 100 ] = 6
                                  [ 87829 * 6 * 10 / 1024 ] + 117 = 5263
                                  [ 5263 * 10 / 25 ]                     = 2105
  220 armorclass                  [ 87829  * 220 * 10   / 1024 ] + 17    = 188712
  15  item_skillongethit          [ 87829  * 15  * 640  / 1024 ] + 16000 = 839396
      (Confuse, skills.txt: mul=640 add=16000)
  5   item_mana_perlevel          [ 87829  * 5   * 128  / 1024 ] + 90    = 54983
  20  fireresist                  [ 87829  * 20  * 20   / 1024 ] + 43    = 34351
  20  lightresist                 [ 87829  * 20  * 20   / 1024 ] + 43    = 34351
  20  coldresist                  [ 87829  * 20  * 20   / 1024 ] + 43    = 34351
  20  poisonresist                [ 87829  * 20  * 20   / 1024 ] + 43    = 34351
  30  item_fastergethitrate       [ 87829  * 30  * 72   / 1024 ] + 1065  = 186329
  15  item_aura (Level 15 Holy Shock Aura When Equipped)                 = 0
  25  item_magicbonus             [ 87829  * 25  * 102  / 1024 ] + 577   = 219291
  10  vitality (Io)               [ 87829  * 10  * 55   / 1024 ] + 100   = 47273
  5   item_maxhp_percent (Jah)    [ 87829  * 5   * 204  / 1024 ] + 32093 = 119578
  30  item_armor_percent (Pul)    [ (157+1) * 30 / 100 ] = 47
                                  [ 87829 * 47 * 20 / 1024 ] + 47 = 80671
                                  [ 80671 / 2 ]                          = 40335

total = 87829 + 97860 + 19750 + 2105 + 188712 + 839396 + 54983 +
        4 * 34351 + 186329 + 219291 + 47273 + 119578 + 40335             = 2040845

scaled_repair_cost = [2040845 * (46-0) / 46] = 2040845
repair_cost = [ 2040845 / 1024 ] * 128 = 255104  (accurate in-game value: 254976)


Superior 15/15 Diadem Dream (0 durability)
==========================================

scaled_armor_cost = [58000 * (60+1) / 60]                                = 58966
socketed_items_cost = [(27440 + 117740 + 50540) / 2]                     = 97860

bonus_costs:
  15  item_armor_percent          [ (60+1) * 15 / 100 ] = 9
                                  [ 58966 * 9 * 20 / 1024 ] + 47 = 10412
                                  [ 10412 / 2 ]                          = 5206
  15  item_maxdurability_percent  [ 20 * 15 / 100 ] = 3
                                  [ 58966 * 3 * 10 / 1024 ] + 117 = 1844
                                  [ 1844 * 10 / 25 ]                     = 737
  220 armorclass                  [ 58966  * 220 * 10   / 1024 ] + 17    = 126701
  15  item_skillongethit          [ 58966  * 15  * 640  / 1024 ] + 16000 = 568806
      (Confuse, skills.txt: mul=640 add=16000)
  5   item_mana_perlevel          [ 58966  * 5   * 128  / 1024 ] + 90    = 36943
  20  fireresist                  [ 58966  * 20  * 20   / 1024 ] + 43    = 23076
  20  lightresist                 [ 58966  * 20  * 20   / 1024 ] + 43    = 23076
  20  coldresist                  [ 58966  * 20  * 20   / 1024 ] + 43    = 23076
  20  poisonresist                [ 58966  * 20  * 20   / 1024 ] + 43    = 23076
  30  item_fastergethitrate       [ 58966  * 30  * 72   / 1024 ] + 1065  = 125446
  15  item_aura (Level 15 Holy Shock Aura When Equipped)                 = 0
  25  item_magicbonus             [ 58966  * 25  * 102  / 1024 ] + 577   = 147416
  10  vitality (Io)               [ 58966  * 10  * 55   / 1024 ] + 100   = 31771
  5   item_maxhp_percent (Jah)    [ 58966  * 5   * 204  / 1024 ] + 32093 = 90828
  30  item_armor_percent (Pul)    [ (60+1) * 30 / 100 ] = 18
                                  [ 58966 * 18 * 20 / 1024 ] + 47 = 20777
                                  [ 20777 / 2 ]                          = 10388

total = 58966 + 97860 + 5206 + 737 + 126701 + 568806 + 36943 +
        4 * 23076 + 125446 + 147416 + 31771 + 90828 + 10388              = 1393372

scaled_repair_cost = [1393372 * (23-0) / 23] = 1393372
repair_cost = [ 1393372 / 1024 ] * 128 = 174080  (accurate in-game value: 173056)


Superior 15/15 Great Helm Dream (0 durability)
==============================================

scaled_armor_cost = [6177 * (35+1) / 35]                                 = 6353
socketed_items_cost = [(27440 + 117740 + 50540) / 2]                     = 97860

bonus_costs:
  15  item_armor_percent          [ (35+1) * 15 / 100 ] = 5
                                  [ 6353 * 5 * 20 / 1024 ] + 47 = 667
                                  [ 667 / 2 ]                            = 333
  15  item_maxdurability_percent  [ 40 * 15 / 100 ] = 6
                                  [ 6353 * 6 * 10 / 1024 ] + 117 = 489
                                  [ 489 * 10 / 25 ]                      = 195
  220 armorclass                  [ 6353   * 220 * 10   / 1024 ] + 17    = 13666
  15  item_skillongethit          [ 6353   * 15  * 640  / 1024 ] + 16000 = 75559
      (Confuse, skills.txt: mul=640 add=16000)
  5   item_mana_perlevel          [ 6353   * 5   * 128  / 1024 ] + 90    = 4060
  20  fireresist                  [ 6353   * 20  * 20   / 1024 ] + 43    = 2524
  20  lightresist                 [ 6353   * 20  * 20   / 1024 ] + 43    = 2524
  20  coldresist                  [ 6353   * 20  * 20   / 1024 ] + 43    = 2524
  20  poisonresist                [ 6353   * 20  * 20   / 1024 ] + 43    = 2524
  30  item_fastergethitrate       [ 6353   * 30  * 72   / 1024 ] + 1065  = 14465
  15  item_aura (Level 15 Holy Shock Aura When Equipped)                 = 0
  25  item_magicbonus             [ 6353   * 25  * 102  / 1024 ] + 577   = 16397
  10  vitality (Io)               [ 6353   * 10  * 55   / 1024 ] + 100   = 3512
  5   item_maxhp_percent (Jah)    [ 6353   * 5   * 204  / 1024 ] + 32093 = 38421
  30  item_armor_percent (Pul)    [ (35+1) * 30 / 100 ] = 10
                                  [ 6353 * 10 * 20 / 1024 ] + 47 = 1287
                                  [ 1287 / 2 ]                           = 643

total = 6353 + 97860 + 333 + 195 + 13666 + 75559 + 4060 + 4 * 2524 +
        14465 + 16397 + 3512 + 38421 + 643                               = 281560

scaled_repair_cost = [281560 * (46-0) / 46] = 281560
repair_cost = [ 281560 / 1024 ] * 128 = 35072  (accurate in-game value: 35072)


Superior 15/15 Bone Visage Flickering Flame (0 durability)
==========================================================

scaled_armor_cost = [87274 * (157+1) / 157]                              = 87829
socketed_items_cost = [(1260 + 50540 + 80640) / 2]                       = 66220

bonus_costs:
  15  item_armor_percent          [ (157+1) * 15 / 100 ] = 23
                                  [ 87829 * 23 * 20 / 1024 ] + 47 = 39501
                                  [ 39501 / 2 ]                          = 19750
  15  item_maxdurability_percent  [ 40 * 15 / 100 ] = 6
                                  [ 87829 * 6 * 10 / 1024 ] + 117 = 5263
                                  [ 5263 * 10 / 25 ]                     = 2105
  3   item_elemskill              [ 87829  * 3   * 1024 / 1024 ] + 76    = 263563
  8   item_aura (Level 4-8 Resist Fire Aura When Equipped)               = 0
  15  passive_fire_pierce         [ 87829  * 15  * 2578 / 1024 ] + 0     = 3316745
  75  maxmana                     [ 87829  * 75  * 20   / 1024 ] + 81    = 128736
  1   item_halffreezeduration     [ 87829  * 1   * 988  / 1024 ] + 5096  = 89837
  50  item_poisonlengthresist     [ 87829  * 50  * 10   / 1024 ] + 27    = 42912
  30  armorclass_vs_missile (Nef) [ 87829  * 30  * 5    / 1024 ] + 11    = 12876
  30  item_armor_percent (Pul)    [ (157+1) * 30 / 100 ] = 47
                                  [ 87829 * 47 * 20 / 1024 ] + 47 = 80671
                                  [ 80671 / 2 ]                          = 40335
  5   maxfireresist (Vex)         [ 87829  * 5   * 256  / 1024 ] + 584   = 110370

total = 87829 + 66220 + 19750 + 2105 + 263563 + 3316745 + 128736 +
        89837 + 42912 + 12876 + 40335 + 110370                           = 4181278

scaled_repair_cost = [4181278 * (46-0) / 46] = 4181278
repair_cost = [ 4181278 / 1024 ] * 128 = 522624  (accurate in-game value: 522240)


Superior 15/15 Diadem Flickering Flame (0 durability)
=====================================================

scaled_armor_cost = [58000 * (60+1) / 60]                                = 58966
socketed_items_cost = [(1260 + 50540 + 80640) / 2]                       = 66220

bonus_costs:
  15  item_armor_percent          [ (60+1) * 15 / 100 ] = 9
                                  [ 58966 * 9 * 20 / 1024 ] + 47 = 10412
                                  [ 10412 / 2 ]                          = 5206
  15  item_maxdurability_percent  [ 20 * 15 / 100 ] = 3
                                  [ 58966 * 3 * 10 / 1024 ] + 117 = 1844
                                  [ 1844 * 10 / 25 ]                     = 737
  3   item_elemskill              [ 58966  * 3   * 1024 / 1024 ] + 76    = 176974
  8   item_aura (Level 4-8 Resist Fire Aura When Equipped)               = 0
  15  passive_fire_pierce         [ 58966  * 15  * 2578 / 1024 ] + 0     = 2226772
  75  maxmana                     [ 58966  * 75  * 20   / 1024 ] + 81    = 86456
  1   item_halffreezeduration     [ 58966  * 1   * 988  / 1024 ] + 5096  = 61988
  50  item_poisonlengthresist     [ 58966  * 50  * 10   / 1024 ] + 27    = 28818
  30  armorclass_vs_missile (Nef) [ 58966  * 30  * 5    / 1024 ] + 11    = 8648
  30  item_armor_percent (Pul)    [ (60+1) * 30 / 100 ] = 18
                                  [ 58966 * 18 * 20 / 1024 ] + 47 = 20777
                                  [ 20777 / 2 ]                          = 10388
  5   maxfireresist (Vex)         [ 58966  * 5   * 256  / 1024 ] + 584   = 74291

total = 58966 + 66220 + 5206 + 737 + 176974 + 2226772 + 86456 + 61988 +
        28818 + 8648 + 10388 + 74291                                     = 2805464

scaled_repair_cost = [2805464 * (23-0) / 23] = 2805464
repair_cost = [ 2805464 / 1024 ] * 128 = 350592  (accurate in-game value: 350208)


Superior 15/15 Great Helm Flickering Flame (0 durability)
=========================================================

scaled_armor_cost = [6177 * (35+1) / 35]                                 = 6353
socketed_items_cost = [(1260 + 50540 + 80640) / 2]                       = 66220

bonus_costs:
  15  item_armor_percent          [ (35+1) * 15 / 100 ] = 5
                                  [ 6353 * 5 * 20 / 1024 ] + 47 = 667
                                  [ 667 / 2 ]                            = 333
  15  item_maxdurability_percent  [ 40 * 15 / 100 ] = 6
                                  [ 6353 * 6 * 10 / 1024 ] + 117 = 489
                                  [ 489 * 10 / 25 ]                      = 195
  3   item_elemskill              [ 6353   * 3   * 1024 / 1024 ] + 76    = 19135
  8   item_aura (Level 4-8 Resist Fire Aura When Equipped)               = 0
  15  passive_fire_pierce         [ 6353   * 15  * 2578 / 1024 ] + 0     = 239912
  75  maxmana                     [ 6353   * 75  * 20   / 1024 ] + 81    = 9387
  1   item_halffreezeduration     [ 6353   * 1   * 988  / 1024 ] + 5096  = 11225
  50  item_poisonlengthresist     [ 6353   * 50  * 10   / 1024 ] + 27    = 3129
  30  armorclass_vs_missile (Nef) [ 6353   * 30  * 5    / 1024 ] + 11    = 941
  30  item_armor_percent (Pul)    [ (35+1) * 30 / 100 ] = 10
                                  [ 6353 * 10 * 20 / 1024 ] + 47 = 1287
                                  [ 1287 / 2 ]                           = 643
  5   maxfireresist (Vex)         [ 6353   * 5   * 256  / 1024 ] + 584   = 8525

total = 6353 + 66220 + 333 + 195 + 19135 + 239912 + 9387 + 11225 +
        3129 + 941 + 643 + 8525                                          = 365998

scaled_repair_cost = [365998 * (46-0) / 46] = 365998
repair_cost = [ 365998 / 1024 ] * 128 = 45696  (accurate in-game value: 45696)


Superior 15/15 Bone Visage Hearth (0 durability)
================================================

scaled_armor_cost = [87274 * (157+1) / 157]                              = 87829
socketed_items_cost = [(16940 + 27440 + 8960) / 2]                       = 26670

bonus_costs:
  15  item_armor_percent          [ (157+1) * 15 / 100 ] = 23
                                  [ 87829 * 23 * 20 / 1024 ] + 47 = 39501
                                  [ 39501 / 2 ]                          = 19750
  15  item_maxdurability_percent  [ 40 * 15 / 100 ] = 6
                                  [ 87829 * 6 * 10 / 1024 ] + 117 = 5263
                                  [ 5263 * 10 / 25 ]                     = 2105
  5   item_maxhp_percent          [ 87829  * 5   * 204  / 1024 ] + 32093 = 119578
  100 item_armor_percent          [ (157+1) * 100 / 100 ] = 158
                                  [ 87829 * 158 * 20 / 1024 ] + 47 = 271081
                                  [ 271081 / 2 ]                         = 135540
  30  coldresist                  [ 87829  * 30  * 20   / 1024 ] + 43    = 51505
  15  item_absorbcold_percent     [ 87829  * 15  * 102  / 1024 ] + 5486  = 136714
  1   item_cannotbefrozen         [ 87829  * 1   * 2048 / 1024 ] + 15011 = 190669
  20  item_fastergethitrate (Shael) [ 87829 * 20 * 72   / 1024 ] + 1065  = 124574
  10  vitality (Io)               [ 87829  * 10  * 55   / 1024 ] + 100   = 47273
  30  coldresist (Thul)           [ 87829  * 30  * 20   / 1024 ] + 43    = 51505

total = 87829 + 26670 + 19750 + 2105 + 119578 + 135540 + 51505 +
        136714 + 190669 + 124574 + 47273 + 51505                         = 993712

scaled_repair_cost = [993712 * (46-0) / 46] = 993712
repair_cost = [ 993712 / 1024 ] * 128 = 124160  (accurate in-game value: 123904)


Superior 15/15 Diadem Hearth (0 durability)
===========================================

scaled_armor_cost = [58000 * (60+1) / 60]                                = 58966
socketed_items_cost = [(16940 + 27440 + 8960) / 2]                       = 26670

bonus_costs:
  15  item_armor_percent          [ (60+1) * 15 / 100 ] = 9
                                  [ 58966 * 9 * 20 / 1024 ] + 47 = 10412
                                  [ 10412 / 2 ]                          = 5206
  15  item_maxdurability_percent  [ 20 * 15 / 100 ] = 3
                                  [ 58966 * 3 * 10 / 1024 ] + 117 = 1844
                                  [ 1844 * 10 / 25 ]                     = 737
  5   item_maxhp_percent          [ 58966  * 5   * 204  / 1024 ] + 32093 = 90828
  100 item_armor_percent          [ (60+1) * 100 / 100 ] = 61
                                  [ 58966 * 61 * 20 / 1024 ] + 47 = 70299
                                  [ 70299 / 2 ]                          = 35149
  30  coldresist                  [ 58966  * 30  * 20   / 1024 ] + 43    = 34593
  15  item_absorbcold_percent     [ 58966  * 15  * 102  / 1024 ] + 5486  = 93589
  1   item_cannotbefrozen         [ 58966  * 1   * 2048 / 1024 ] + 15011 = 132943
  20  item_fastergethitrate (Shael) [ 58966 * 20 * 72   / 1024 ] + 1065  = 83985
  10  vitality (Io)               [ 58966  * 10  * 55   / 1024 ] + 100   = 31771
  30  coldresist (Thul)           [ 58966  * 30  * 20   / 1024 ] + 43    = 34593

total = 58966 + 26670 + 5206 + 737 + 90828 + 35149 + 34593 + 93589 +
        132943 + 83985 + 31771 + 34593                                   = 629030

scaled_repair_cost = [629030 * (23-0) / 23] = 629030
repair_cost = [ 629030 / 1024 ] * 128 = 78592  (accurate in-game value: 77824)


Superior 15/15 Great Helm Hearth (0 durability)
===============================================

scaled_armor_cost = [6177 * (35+1) / 35]                                 = 6353
socketed_items_cost = [(16940 + 27440 + 8960) / 2]                       = 26670

bonus_costs:
  15  item_armor_percent          [ (35+1) * 15 / 100 ] = 5
                                  [ 6353 * 5 * 20 / 1024 ] + 47 = 667
                                  [ 667 / 2 ]                            = 333
  15  item_maxdurability_percent  [ 40 * 15 / 100 ] = 6
                                  [ 6353 * 6 * 10 / 1024 ] + 117 = 489
                                  [ 489 * 10 / 25 ]                      = 195
  5   item_maxhp_percent          [ 6353   * 5   * 204  / 1024 ] + 32093 = 38421
  100 item_armor_percent          [ (35+1) * 100 / 100 ] = 36
                                  [ 6353 * 36 * 20 / 1024 ] + 47 = 4513
                                  [ 4513 / 2 ]                           = 2256
  30  coldresist                  [ 6353   * 30  * 20   / 1024 ] + 43    = 3765
  15  item_absorbcold_percent     [ 6353   * 15  * 102  / 1024 ] + 5486  = 14978
  1   item_cannotbefrozen         [ 6353   * 1   * 2048 / 1024 ] + 15011 = 27717
  20  item_fastergethitrate (Shael) [ 6353 * 20  * 72   / 1024 ] + 1065  = 9998
  10  vitality (Io)               [ 6353   * 10  * 55   / 1024 ] + 100   = 3512
  30  coldresist (Thul)           [ 6353   * 30  * 20   / 1024 ] + 43    = 3765

total = 6353 + 26670 + 333 + 195 + 38421 + 2256 + 3765 + 14978 +
        27717 + 9998 + 3512 + 3765                                       = 137963

scaled_repair_cost = [137963 * (46-0) / 46] = 137963
repair_cost = [ 137963 / 1024 ] * 128 = 17152  (accurate in-game value: 17152)


Superior 15/15 Bone Visage Lore (0 durability)
==============================================

scaled_armor_cost = [87274 * (157+1) / 157]                              = 87829
socketed_items_cost = [(6860 + 14000) / 2]                               = 10430

bonus_costs:
  15  item_armor_percent          [ (157+1) * 15 / 100 ] = 23
                                  [ 87829 * 23 * 20 / 1024 ] + 47 = 39501
                                  [ 39501 / 2 ]                          = 19750
  15  item_maxdurability_percent  [ 40 * 15 / 100 ] = 6
                                  [ 87829 * 6 * 10 / 1024 ] + 117 = 5263
                                  [ 5263 * 10 / 25 ]                     = 2105
  10  energy                      [ 87829  * 10  * 55   / 1024 ] + 100   = 47273
  1   item_allskills              [ 87829  * 1   * 4096 / 1024 ] + 15123 = 366439
  2   item_lightradius            [ 87829  * 2   * 51   / 1024 ] + 15    = 8763
  2   item_manaafterkill          [ 87829  * 2   * 102  / 1024 ] + 17    = 17514
  30  lightresist (Ort)           [ 87829  * 30  * 20   / 1024 ] + 43    = 51505
  7   normal_damage_reduction (Sol) [ 87829 * 7  * 200  / 1024 ] + 188   = 120266

total = 87829 + 10430 + 19750 + 2105 + 47273 + 366439 + 8763 +
        17514 + 51505 + 120266                                           = 731874

scaled_repair_cost = [731874 * (46-0) / 46] = 731874
repair_cost = [ 731874 / 1024 ] * 128 = 91392  (accurate in-game value: 90112)


Superior 15/15 Diadem Lore (0 durability)
=========================================

scaled_armor_cost = [58000 * (60+1) / 60]                                = 58966
socketed_items_cost = [(6860 + 14000) / 2]                               = 10430

bonus_costs:
  15  item_armor_percent          [ (60+1) * 15 / 100 ] = 9
                                  [ 58966 * 9 * 20 / 1024 ] + 47 = 10412
                                  [ 10412 / 2 ]                          = 5206
  15  item_maxdurability_percent  [ 20 * 15 / 100 ] = 3
                                  [ 58966 * 3 * 10 / 1024 ] + 117 = 1844
                                  [ 1844 * 10 / 25 ]                     = 737
  10  energy                      [ 58966  * 10  * 55   / 1024 ] + 100   = 31771
  1   item_allskills              [ 58966  * 1   * 4096 / 1024 ] + 15123 = 250987
  2   item_lightradius            [ 58966  * 2   * 51   / 1024 ] + 15    = 5888
  2   item_manaafterkill          [ 58966  * 2   * 102  / 1024 ] + 17    = 11764
  30  lightresist (Ort)           [ 58966  * 30  * 20   / 1024 ] + 43    = 34593
  7   normal_damage_reduction (Sol) [ 58966 * 7  * 200  / 1024 ] + 188   = 80805

total = 58966 + 10430 + 5206 + 737 + 31771 + 250987 + 5888 +
        11764 + 34593 + 80805                                            = 491147

scaled_repair_cost = [491147 * (23-0) / 23] = 491147
repair_cost = [ 491147 / 1024 ] * 128 = 61312  (accurate in-game value: 61312)


Superior 15/15 Grim Helm Lore (0 durability)
============================================

scaled_armor_cost = [37683 * (125+1) / 125]                              = 37984
socketed_items_cost = [(6860 + 14000) / 2]                               = 10430

bonus_costs:
  15  item_armor_percent          [ (125+1) * 15 / 100 ] = 18
                                  [ 37984 * 18 * 20 / 1024 ] + 47 = 13400
                                  [ 13400 / 2 ]                          = 6700
  15  item_maxdurability_percent  [ 40 * 15 / 100 ] = 6
                                  [ 37984 * 6 * 10 / 1024 ] + 117 = 2342
                                  [ 2342 * 10 / 25 ]                     = 936
  10  energy                      [ 37984  * 10  * 55   / 1024 ] + 100   = 20501
  1   item_allskills              [ 37984  * 1   * 4096 / 1024 ] + 15123 = 167059
  2   item_lightradius            [ 37984  * 2   * 51   / 1024 ] + 15    = 3798
  2   item_manaafterkill          [ 37984  * 2   * 102  / 1024 ] + 17    = 7584
  30  lightresist (Ort)           [ 37984  * 30  * 20   / 1024 ] + 43    = 22299
  7   normal_damage_reduction (Sol) [ 37984 * 7  * 200  / 1024 ] + 188   = 52119

total = 37984 + 10430 + 6700 + 936 + 20501 + 167059 + 3798 + 7584 +
        22299 + 52119                                                    = 329410

scaled_repair_cost = [329410 * (46-0) / 46] = 329410
repair_cost = [ 329410 / 1024 ] * 128 = 41088  (accurate in-game value: 41088)


Superior 15/15 Bone Helm Lore (0 durability)
============================================

scaled_armor_cost = [6323 * (36+1) / 36]                                 = 6498
socketed_items_cost = [(6860 + 14000) / 2]                               = 10430

bonus_costs:
  15  item_armor_percent          [ (36+1) * 15 / 100 ] = 5
                                  [ 6498 * 5 * 20 / 1024 ] + 47 = 681
                                  [ 681 / 2 ]                            = 340
  15  item_maxdurability_percent  [ 40 * 15 / 100 ] = 6
                                  [ 6498 * 6 * 10 / 1024 ] + 117 = 497
                                  [ 497 * 10 / 25 ]                      = 198
  10  energy                      [ 6498   * 10  * 55   / 1024 ] + 100   = 3590
  1   item_allskills              [ 6498   * 1   * 4096 / 1024 ] + 15123 = 41115
  2   item_lightradius            [ 6498   * 2   * 51   / 1024 ] + 15    = 662
  2   item_manaafterkill          [ 6498   * 2   * 102  / 1024 ] + 17    = 1311
  30  lightresist (Ort)           [ 6498   * 30  * 20   / 1024 ] + 43    = 3850
  7   normal_damage_reduction (Sol) [ 6498 * 7   * 200  / 1024 ] + 188   = 9071

total = 6498 + 10430 + 340 + 198 + 3590 + 41115 + 662 + 1311 +
        3850 + 9071                                                      = 77065

scaled_repair_cost = [77065 * (46-0) / 46] = 77065
repair_cost = [ 77065 / 1024 ] * 128 = 9600  (accurate in-game value: 9600)


Superior 15/15 Great Helm Lore (0 durability)
=============================================

scaled_armor_cost = [6177 * (35+1) / 35]                                 = 6353
socketed_items_cost = [(6860 + 14000) / 2]                               = 10430

bonus_costs:
  15  item_armor_percent          [ (35+1) * 15 / 100 ] = 5
                                  [ 6353 * 5 * 20 / 1024 ] + 47 = 667
                                  [ 667 / 2 ]                            = 333
  15  item_maxdurability_percent  [ 40 * 15 / 100 ] = 6
                                  [ 6353 * 6 * 10 / 1024 ] + 117 = 489
                                  [ 489 * 10 / 25 ]                      = 195
  10  energy                      [ 6353   * 10  * 55   / 1024 ] + 100   = 3512
  1   item_allskills              [ 6353   * 1   * 4096 / 1024 ] + 15123 = 40535
  2   item_lightradius            [ 6353   * 2   * 51   / 1024 ] + 15    = 647
  2   item_manaafterkill          [ 6353   * 2   * 102  / 1024 ] + 17    = 1282
  30  lightresist (Ort)           [ 6353   * 30  * 20   / 1024 ] + 43    = 3765
  7   normal_damage_reduction (Sol) [ 6353 * 7   * 200  / 1024 ] + 188   = 8873

total = 6353 + 10430 + 333 + 195 + 3512 + 40535 + 647 + 1282 +
        3765 + 8873                                                      = 75925

scaled_repair_cost = [75925 * (46-0) / 46] = 75925
repair_cost = [ 75925 / 1024 ] * 128 = 9472  (accurate in-game value: 9472)


Superior 15/15 Helm Lore (0 durability)
=======================================

scaled_armor_cost = [1558 * (18+1) / 18]                                 = 1644
socketed_items_cost = [(6860 + 14000) / 2]                               = 10430

bonus_costs:
  15  item_armor_percent          [ (18+1) * 15 / 100 ] = 2
                                  [ 1644 * 2 * 20 / 1024 ] + 47 = 111
                                  [ 111 / 2 ]                            = 55
  15  item_maxdurability_percent  [ 24 * 15 / 100 ] = 3
                                  [ 1644 * 3 * 10 / 1024 ] + 117 = 165
                                  [ 165 * 10 / 25 ]                      = 66
  10  energy                      [ 1644   * 10  * 55   / 1024 ] + 100   = 983
  1   item_allskills              [ 1644   * 1   * 4096 / 1024 ] + 15123 = 21699
  2   item_lightradius            [ 1644   * 2   * 51   / 1024 ] + 15    = 178
  2   item_manaafterkill          [ 1644   * 2   * 102  / 1024 ] + 17    = 344
  30  lightresist (Ort)           [ 1644   * 30  * 20   / 1024 ] + 43    = 1006
  7   normal_damage_reduction (Sol) [ 1644 * 7   * 200  / 1024 ] + 188   = 2435

total = 1644 + 10430 + 55 + 66 + 983 + 21699 + 178 + 344 + 1006 + 2435   = 38840

scaled_repair_cost = [38840 * (27-0) / 27] = 38840
repair_cost = [ 38840 * 128 / 1024 ] = 4855  (accurate in-game value: 4849)
```


Repair cost tables
------------------

Summary of the above calculations:

| runeword         | 15/15 base           | charsi cost | calculated cost |
|:-----------------|:---------------------|:------------|:----------------|
| Enigma           | Archon Plate         | 1,485,824   | 1,486,080       |
|                  | Dusk Shroud          | 1,026,048   | 1,026,560       |
|                  | Mage Plate           | 563,200     | 563,968         |
|                  | Full Plate Mail      | 237,568     | 238,208         |
|                  | Breast Plate         | 68,608      | 68,992          |
| Chains of Honor  | Archon Plate         | 1,271,808   | 1,272,192       |
|                  | Dusk Shroud          | 866,304     | 867,200         |
| Dragon           | Archon Plate         | 1,641,472   | 1,641,856       |
|                  | Dusk Shroud          | 1,137,664   | 1,137,920       |
| Fortitude        | Archon Plate         | 1,911,808   | 1,912,448       |
|                  | Dusk Shroud          | 1,285,120   | 1,286,272       |
| Duress           | Archon Plate         | 3,217,408   | 3,209,856       |
|                  | Dusk Shroud          | 2,181,120   | 2,176,384       |
| Hustle           | Archon Plate         | 1,041,408   | 1,041,792       |
|                  | Dusk Shroud          | 715,776     | 716,416         |
| Treachery        | Archon Plate         | 1,524,736   | 1,525,120       |
|                  | Dusk Shroud          | 1,051,648   | 1,052,416       |
| Smoke            | Archon Plate         | 601,088     | 601,088         |
|                  | Dusk Shroud          | 399,360     | 400,256         |
|                  | Mage Plate           | 191,488     | 191,872         |
| Stealth          | Dusk Shroud          | 363,520     | 364,416         |
|                  | Mage Plate           | 193,536     | 194,432         |
|                  | Breast Plate         | 17,664      | 17,792          |
| Phoenix          | 45@ Sacred Targe     | 1,186,816   | 1,187,200       |
|                  | Monarch              | 1,299,456   | 1,300,224       |
| Dream            | 45@ Sacred Targe     | 242,688     | 243,200         |
| Dragon           | 45@ Sacred Targe     | 423,936     | 425,088         |
| Spirit           | 45@ Sacred Targe     | 295,936     | 296,448         |
|                  | 45@ Protector Shield | 217,088     | 217,472         |
|                  | Monarch              | 296,960     | 297,856         |
| Ancient's Pledge | 45@ Sacred Targe     | 86,016      | 86,656          |
|                  | 45@ Protector Shield | 63,360      | 63,360          |
|                  | Large Shield         | 1,796       | 1,825           |
| Dream            | Bone Visage          | 254,976     | 255,104         |
|                  | Diadem               | 173,056     | 174,080         |
|                  | Great Helm           | 35,072      | 35,072          |
| Flickering Flame | Bone Visage          | 522,240     | 522,624         |
|                  | Diadem               | 350,208     | 350,592         |
|                  | Great Helm           | 45,696      | 45,696          |
| Hearth           | Bone Visage          | 123,904     | 124,160         |
|                  | Diadem               | 77,824      | 78,592          |
|                  | Great Helm           | 17,152      | 17,152          |
| Lore             | Bone Visage          | 90,112      | 91,392          |
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

Charsi's repair costs seem to be an integer multiple of 1024 for expensive repairs. This means that the least significant 10 bits of the cost are all zero. Zeroing out the least significant 10 bits of my calculated repair cost often results in the same value as Charsi's. For example, for a Dusk Shroud Enigma: `[1026560 / 1024] * 1024 = 1026048`.

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
