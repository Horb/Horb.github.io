---
layout: post
title:  "Hero Grid JSON"
date:   2021-01-25 00:00:00 +0000
categories: dota2 software
---

Here's a snippet of a `hero_grid_config.json`;

```
{
    "version": 3,
    "configs":
    [
        {
            "config_name": "Config Name",
            "categories":
                [
                    {
                        "category_name": "Category Name",
                        "x_position": 0.000000,
                        "y_position": 0.000000,
                        "width": 479.130432,
                        "height": 155.652176,
                        "hero_ids": [
                            2,
                            6,
                            48,
                            56,
...
```

On Linux these are stored at `.steam/steam/userdata/<youruserid>/570/remote/cfg/hero_grid_config.json`.

The heroes are specified by HeroId. Here's a snippet of the mapping;

```
1,Anti-Mage
2,Axe
3,Bane
4,Bloodseeker
5,Crystal Maiden
6,Drow Ranger
```

I'd like a text-to-json mapping from a format such as;

```
title=Starts With A
columns=1

# Strength
Axe
Alchemist
Axe

# Agility
Anti-Mage
Arc Warden

# Intelligence
Ancient Aparition
```

Which would be converted to;
```
{
    "version": 3,
    "configs":
    [
        {
            "config_name": "Starts With A",
            "categories":
                [
                    {
                        "category_name": "Strength",
                        "hero_ids": 
                        [
                            102,
                            73,
                            2
                        ]
                    },
                    {
                        "category_name": "Agility",
                        "hero_ids": 
                        [
                            1,
                            113
                        ]
                    },
                    {
                        "category_name": "Intelligence",
                        "hero_ids": 
                        [
                            68
                        ]
                    }
                ]
        }
    ]
}
```

Without the position or size properties the Dota2 client just stacks the categories
on top of each other.  We need some estimate of portrait size in order
to calculate position and size. Let's export the default _Attributes_ hero grid
and see what values it's using.

## Attributes Hero Grid JSON

```
{
    "config_name": "CopyOfAttributes",
    "categories":
    [
        {
            "category_name": "Strength",
            "x_position": 0.000000,
            "y_position": 0.000000,
            "width": 1188.695679,
            "height": 173.913040,
    ...
            "category_name": "Agility",
            "x_position": 0.000000,
            "y_position": 201.739136,
            "width": 1188.695679,
            "height": 173.913040,
    ...
            "category_name": "Intelligence",
            "x_position": 0.000000,
            "y_position": 403.478271,
            "width": 1188.695679,
            "height": 173.913040,
```

I assume those units are pixels and we have 1188 pixels of width to work with.
A hero portrait is roughly 50x85 pixels and the category titles need ca. 30
pixels of padding for the category name.

From there it's possible to write rudimentary _parse config and layout hero grid_ code. Try editing the Config below and clicking Generate to see the Hero Grid JSON.

# Config

<textarea id="input" style="height:20em;width:100%;padding-bottom:1em;">
title=Starts With A
columns=2

# Strength
Axe
Alchemist
Axe

# Agility
Anti-Mage
Arc Warden

# Intelligence
Ancient Apparition
</textarea>
<button id="generate" onclick="generate();">Generate</button>
# Hero Grid JSON 
<textarea id="output" style="height:20em;width:100%;padding-bottom:1em;">
</textarea>

I'll do a breakdown of how the code works and some problems it's got soon. Enquiring minds can inspect the script tag at the end of this article. See the appearance of the MVP in the Dota client below;

![Appearance of MVP in Dota client.](/assets/2021-01-25_MVP.png)


<script type="text/javascript">
var hero_id_by_name = {
  "Anti-Mage": 1,
  "Axe": 2,
  "Bane": 3,
  "Bloodseeker": 4,
  "Crystal Maiden": 5,
  "Drow Ranger": 6,
  "Earthshaker": 7,
  "Juggernaut": 8,
  "Mirana": 9,
  "Shadow Fiend": 11,
  "Morphling": 10,
  "Phantom Lancer": 12,
  "Puck": 13,
  "Pudge": 14,
  "Razor": 15,
  "Sand King": 16,
  "Storm Spirit": 17,
  "Sven": 18,
  "Tiny": 19,
  "Vengeful Spirit": 20,
  "Windranger": 21,
  "Zeus": 22,
  "Kunkka": 23,
  "Lina": 25,
  "Lich": 31,
  "Lion": 26,
  "Shadow Shaman": 27,
  "Slardar": 28,
  "Tidehunter": 29,
  "Witch Doctor": 30,
  "Riki": 32,
  "Enigma": 33,
  "Tinker": 34,
  "Sniper": 35,
  "Necrophos": 36,
  "Warlock": 37,
  "Beastmaster": 38,
  "Queen of Pain": 39,
  "Venomancer": 40,
  "Faceless Void": 41,
  "Wraith King": 42,
  "Death Prophet": 43,
  "Phantom Assassin": 44,
  "Pugna": 45,
  "Templar Assassin": 46,
  "Viper": 47,
  "Luna": 48,
  "Dragon Knight": 49,
  "Dazzle": 50,
  "Clockwerk": 51,
  "Leshrac": 52,
  "Nature's Prophet": 53,
  "Lifestealer": 54,
  "Dark Seer": 55,
  "Clinkz": 56,
  "Omniknight": 57,
  "Enchantress": 58,
  "Huskar": 59,
  "Night Stalker": 60,
  "Broodmother": 61,
  "Bounty Hunter": 62,
  "Weaver": 63,
  "Jakiro": 64,
  "Batrider": 65,
  "Chen": 66,
  "Spectre": 67,
  "Doom": 69,
  "Ancient Apparition": 68,
  "Ursa": 70,
  "Spirit Breaker": 71,
  "Gyrocopter": 72,
  "Alchemist": 73,
  "Invoker": 74,
  "Silencer": 75,
  "Outworld Destroyer": 76,
  "Lycan": 77,
  "Brewmaster": 78,
  "Shadow Demon": 79,
  "Lone Druid": 80,
  "Chaos Knight": 81,
  "Meepo": 82,
  "Treant Protector": 83,
  "Ogre Magi": 84,
  "Undying": 85,
  "Rubick": 86,
  "Disruptor": 87,
  "Nyx Assassin": 88,
  "Naga Siren": 89,
  "Keeper of the Light": 90,
  "Io": 91,
  "Visage": 92,
  "Slark": 93,
  "Medusa": 94,
  "Troll Warlord": 95,
  "Centaur Warrunner": 96,
  "Magnus": 97,
  "Timbersaw": 98,
  "Bristleback": 99,
  "Tusk": 100,
  "Skywrath Mage": 101,
  "Abaddon": 102,
  "Elder Titan": 103,
  "Legion Commander": 104,
  "Ember Spirit": 106,
  "Earth Spirit": 107,
  "Terrorblade": 109,
  "Phoenix": 110,
  "Oracle": 111,
  "Techies": 105,
  "Winter Wyvern": 112,
  "Arc Warden": 113,
  "Underlord": 108,
  "Monkey King": 114,
  "Pangolier": 120,
  "Dark Willow": 119,
  "Grimstroke": 121,
  "Mars": 129,
  "Void Spirit": 126,
  "Snapfire": 128,
  "Hoodwink": 123
}
  var hero_pattern = /^Anti-Mage|Axe|Bane|Bloodseeker|Crystal Maiden|Drow Ranger|Earthshaker|Juggernaut|Mirana|Shadow Fiend|Morphling|Phantom Lancer|Puck|Pudge|Razor|Sand King|Storm Spirit|Sven|Tiny|Vengeful Spirit|Windranger|Zeus|Kunkka|Lina|Lich|Lion|Shadow Shaman|Slardar|Tidehunter|Witch Doctor|Riki|Enigma|Tinker|Sniper|Necrophos|Warlock|Beastmaster|Queen of Pain|Venomancer|Faceless Void|Wraith King|Death Prophet|Phantom Assassin|Pugna|Templar Assassin|Viper|Luna|Dragon Knight|Dazzle|Clockwerk|Leshrac|Nature's Prophet|Lifestealer|Dark Seer|Clinkz|Omniknight|Enchantress|Huskar|Night Stalker|Broodmother|Bounty Hunter|Weaver|Jakiro|Batrider|Chen|Spectre|Doom|Ancient Apparition|Ursa|Spirit Breaker|Gyrocopter|Alchemist|Invoker|Silencer|Outworld Destroyer|Lycan|Brewmaster|Shadow Demon|Lone Druid|Chaos Knight|Meepo|Treant Protector|Ogre Magi|Undying|Rubick|Disruptor|Nyx Assassin|Naga Siren|Keeper of the Light|Io|Visage|Slark|Medusa|Troll Warlord|Centaur Warrunner|Magnus|Timbersaw|Bristleback|Tusk|Skywrath Mage|Abaddon|Elder Titan|Legion Commander|Ember Spirit|Earth Spirit|Terrorblade|Phoenix|Oracle|Techies|Winter Wyvern|Arc Warden|Underlord|Monkey King|Pangolier|Dark Willow|Grimstroke|Mars|Void Spirit|Snapfire|Hoodwink$/

var hero_grid_json = function (hero_grid_text)
{
    var config = parse_lines(hero_grid_text.split("\n"));
    console.log(config);
    return layout(config);
}

var parse_lines = function(lines_array)
{
    var g = {
        title: "",
        categories: [],
        columns: 1,
        width: 1200,
        height: 600,
        padding: 30,
        unit_width: 45,
        unit_height: 85,
        current_category: null,

        column_width: function()
        {
            var wells = 0;
            if(this.columns > 1)
            {
                wells = this.columns - 1;
            }
            return (this.width - this.padding * wells) / this.columns;
        },

        parse_line: function(line)
        {
            var title_pattern = /^\s*(?<label>\w*)\s*=\s*(?<value>.*)$/;
            var title_match = line.match(title_pattern);
            if(title_match)
            {
                var label = title_match.groups.label;
                if(label == "title")
                {
                    this.title = title_match.groups.value;
                    return;
                }
                if(label == "columns")
                {
                    this.columns = parseInt(title_match.groups.value);
                    return;
                }
            }

            var category_pattern = /^#\s*(?<label>.+)$/;
            var category_match = line.match(category_pattern);
            if(category_match)
            {
                var c = 
                {
                    category_name: category_match.groups.label,
                    hero_ids: []
                };
                this.current_category = c;
                this.categories.push(c);
                return;
            }

            var hero_pattern = /^(?<name>.+)$/;
            var hero_match = line.match(hero_pattern);
            if(hero_match)
            {
                if(line in hero_id_by_name)
                {
                    var hero_id = hero_id_by_name[line];
                    this.current_category.hero_ids.push(hero_id);
                    return;
                }
            }
            return;
        }
    }

    while (lines_array.length != 0)
    {
        var line = lines_array.shift();
        g.parse_line(line);
    }

    return g;
}

var layout = function(config)
{
    return {
        version: 3,
        configs: 
        [
            {
                config_name: config.title,
                categories: layout_categories(config)
            }
        ]
    }
};

var layout_categories = function(config)
{
    var categories = config.categories;
    var c = [];
    var x = 0;
    var y = 0;
    var max_height = 0;

    while(categories.length != 0)
    {
        var category = categories.shift();
        c.push({
            category_name: category.category_name,
            x_position: x,
            y_position: y,
            width: config.column_width(),
            height: category_height(config, category),
            hero_ids: category.hero_ids
        });

        x += config.column_width() + config.padding;
        if(category_height(config, category) > max_height)
        {
            max_height = category_height(config, category);
        }

        if(x > config.width)
        {
            x = 0;
            y += max_height + config.padding;
            max_height = 0;
        }
    }
    return c;
};

var category_height = function(config, category)
{
    var result = config.unit_height * (1 + Math.floor(category.hero_ids.length * config.unit_width / config.column_width()));
    return result;
};

var generate = function()
{
    var hero_grid_text = document.getElementById("input").value;
    var json = JSON.stringify(hero_grid_json(hero_grid_text), null, '    ');
    document.getElementById("output").value = json;
}
</script>
