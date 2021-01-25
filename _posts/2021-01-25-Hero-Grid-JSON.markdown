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
# Starts With A
Columns=1

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
                        "x_position": 0.000000,
                        "y_position": 0.000000,
                        "width": 0.000000,
                        "height": 0.000000,
                        "hero_ids": 
                        [
                            102,
                            73,
                            2
                        ]
                    },
                    {
                        "category_name": "Agility",
                        "x_position": 0.000000,
                        "y_position": 0.000000,
                        "width": 0.000000,
                        "height": 0.000000,
                        "hero_ids": 
                        [
                            1,
                            113
                        ]
                    },
                    {
                        "category_name": "Intelligence",
                        "x_position": 0.000000,
                        "y_position": 0.000000,
                        "width": 0.000000,
                        "height": 0.000000,
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

Having the position and size properties as zero is going to mean all our categories
will stack on top of one another. We need some estimate of portrait size in order
to calculate position and size. Let's export the default _Attributes_ hero grid
and see what values it's using.

## Default Hero Grid Json

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

This is enough information to start putting together some rudimentary layout logic.
