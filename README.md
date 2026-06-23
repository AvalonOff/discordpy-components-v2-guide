# Discord Components V2 — Guide complet discord.py

![discord.py](https://img.shields.io/badge/discord.py-2.6%2B-blue?logo=python&logoColor=white)
![License](https://img.shields.io/badge/license-MIT-green)
![API](https://img.shields.io/badge/Discord%20API-v10-5865F2)
![Updated](https://img.shields.io/badge/updated-June%202026-orange)

Guide de référence complet sur les **Discord Components V2** (CV2).  
A jour pour discord.py **2.6+** (2.7.1 recommande) et l'API Discord de juin 2026.

---

## Presentation

Les Components V2 ont ete publies officiellement par Discord en mars 2025. Ils remplacent le trio classique `content + embeds + view` par un systeme de blocs composables. Le controle visuel est total : texte Markdown riche n'importe ou, images dans le flux, boutons positionnes en face du contenu qu'ils controlent.

discord.py **2.6** (aout 2025) introduit `discord.ui.LayoutView` comme point d'entree principal.  
La version **2.7.1** (mars 2026) est la derniere stable.

> [!IMPORTANT]
> Le flag `IS_COMPONENTS_V2` (`1 << 15` = `32768`) est obligatoire sur chaque message. Il est **permanent** — un message CV2 ne peut pas revenir en V1. Quand vous utilisez `LayoutView`, discord.py le pose automatiquement.

> [!WARNING]
> `content`, `embeds`, `stickers` et `polls` sont **incompatibles** avec le flag CV2. La limite est de **40 composants** par `LayoutView`.

---

## Table des matieres

| # | Fichier | Description |
|---|---|---|
| 01 | [Introduction](docs/01-introduction.md) | Historique, differences V1/V2, quand utiliser CV2 |
| 02 | [Architecture](docs/02-architecture.md) | Couches Layout/Content/Interactive, IDs de type |
| 03 | [Flag IS_COMPONENTS_V2](docs/03-flag-activation.md) | Activation, comportements, permanence, erreurs |
| 04 | [Container](docs/04-container.md) | Boite avec bord colore, deux styles d'ecriture |
| 05 | [Section](docs/05-section.md) | Layout texte + accessoire, Thumbnail, Button en accessoire |
| 06 | [Text Display](docs/06-text-display.md) | Markdown complet, timestamps, mentions, troncation |
| 07 | [Thumbnail](docs/07-thumbnail.md) | Image accessoire de Section, URL externe vs attachment |
| 08 | [Separator](docs/08-separator.md) | Ligne de separation, spacing small/large, espace vide |
| 09 | [Media Gallery](docs/09-media-gallery.md) | Galerie 1-10 images/videos, dispositions, spoiler |
| 10 | [File](docs/10-file.md) | ui.File, convention attachment://, pieces jointes en CV2 |
| 11 | [Action Rows et boutons](docs/11-action-rows.md) | Boutons, positionnement, callbacks, custom_ids |
| 12 | [Select Menus](docs/16-select-menus.md) | StringSelect, UserSelect, RoleSelect, ChannelSelect, MentionableSelect |
| 13 | [LayoutView](docs/17-layout-view.md) | Point d'entree CV2, timeout, persistance, add_item |
| 14 | [Exemples pratiques](docs/12-exemples.md) | 7 cas d'usage complets combinant plusieurs composants |
| 15 | [Migration V1 a CV2](docs/13-migration.md) | Equivalences directes, migration progressive, checklist |
| 16 | [Bonnes pratiques](docs/14-bonnes-pratiques.md) | Structure, mentions, patterns a eviter |
| 17 | [Reference discord.py](docs/15-discord-py.md) | Toutes les classes, methodes, IDs de type, codes d'erreur |

---

## Demarrage rapide

```bash
pip install -U "discord.py>=2.6"
```

```python
import discord
from discord import ui

bot = discord.Bot()

class BonjLayout(ui.LayoutView):
    container = ui.Container(
        ui.TextDisplay("## Bonjour !\nCeci est un message CV2."),
        ui.Separator(),
        ui.ActionRow(
            ui.Button(
                label="Documentation",
                style=discord.ButtonStyle.link,
                url="https://discord.com/developers/docs/components/reference",
            )
        ),
        accent_color=discord.Color.blurple(),
    )

@bot.tree.command(name="bonjour")
async def bonjour(interaction: discord.Interaction):
    await interaction.response.send_message(view=BonjLayout())

bot.run("TOKEN")
```

---

## Les deux styles d'ecriture

### Declaratif — attributs de classe

```python
class MonLayout(discord.ui.LayoutView):
    text = discord.ui.TextDisplay("## Titre")
    sep  = discord.ui.Separator()
    row  = discord.ui.ActionRow(
        discord.ui.Button(label="OK", custom_id="ok", style=discord.ButtonStyle.primary)
    )

await interaction.response.send_message(view=MonLayout())
```

### Imperatif — add_item()

```python
view = discord.ui.LayoutView()
container = discord.ui.Container(accent_color=discord.Color.blurple())
container.add_item(discord.ui.TextDisplay("## Titre"))
container.add_item(discord.ui.Separator())
view.add_item(container)

await interaction.response.send_message(view=view)
```

Le style **declaratif** est ideal pour les layouts statiques.  
Le style **imperatif** est indispensable pour construire dynamiquement (boucles, conditions).

---

## Tous les composants CV2

```
ui.LayoutView                              point d'entree, pose le flag CV2
├── ui.Container          (type 17)        boite avec bord colore optionnel
│   ├── ui.TextDisplay    (type 10)        texte Markdown
│   ├── ui.Section        (type 9)         texte gauche + accessoire droite
│   │   ├── ui.TextDisplay(s)
│   │   └── accessory: ui.Thumbnail (11) ou ui.Button (2)
│   ├── ui.Separator      (type 14)        ligne ou espace
│   ├── ui.MediaGallery   (type 12)        galerie 1-10 images/videos
│   │   └── ui.MediaGalleryItem
│   ├── ui.File           (type 13)        piece jointe dans le flux
│   └── ui.ActionRow      (type 1)         boutons ou select menu
│       ├── ui.Button     (type 2)
│       ├── ui.Select     (type 3)         options texte libres
│       ├── ui.UserSelect (type 5)
│       ├── ui.RoleSelect (type 6)
│       ├── ui.MentionableSelect (type 7)
│       └── ui.ChannelSelect (type 8)
├── ui.TextDisplay                         texte direct dans la LayoutView
└── ui.ActionRow                           boutons directs dans la LayoutView
```

---

## Regles fondamentales

| Regle | Detail |
|---|---|
| Flag automatique | `LayoutView` pose `IS_COMPONENTS_V2` automatiquement |
| Flag permanent | Une fois pose, ne peut pas etre retire du message |
| Incompatible | `content`, `embeds`, `stickers`, `polls` doivent etre absents |
| Pieces jointes | Passer par `ui.File`, `ui.Thumbnail` ou `ui.MediaGallery` |
| Limite totale | 40 composants maximum par `LayoutView` (imbriques inclus) |
| Texte partage | 4000 chars max partages entre tous les `TextDisplay` de la view |
| Galerie | 1 a 10 items par `MediaGallery` |
| Boutons | 5 maximum par `ActionRow` |
| Select par ActionRow | 1 maximum, exclusif des boutons |
| Button accessoire | Ne peut pas avoir `style=link` (pas d'URL en accessoire) |
| discord.py requis | Version 2.6.0 minimum, 2.7.1 recommande |

---

## Correspondance API — IDs de type

| Composant | Type API | Couche |
|---|---|---|
| ActionRow | `1` | Layout |
| Button | `2` | Interactive |
| StringSelect | `3` | Interactive |
| UserSelect | `5` | Interactive |
| RoleSelect | `6` | Interactive |
| MentionableSelect | `7` | Interactive |
| ChannelSelect | `8` | Interactive |
| Section | `9` | Layout |
| TextDisplay | `10` | Content |
| Thumbnail | `11` | Content |
| MediaGallery | `12` | Content |
| File | `13` | Content |
| Separator | `14` | Layout |
| Container | `17` | Layout |

---

## Ressources officielles

- [Discord Components Reference](https://discord.com/developers/docs/components/reference)
- [Discord Change Log](https://docs.discord.com/developers/change-log)
- [Discord Visual Builder](https://discord.builders)
- [discord.py Documentation](https://discordpy.readthedocs.io)
- [discord.py Changelog](https://discordpy.readthedocs.io/en/latest/whats_new.html)

---

## Contribuer

Voir [CONTRIBUTING.md](CONTRIBUTING.md) pour les regles de contribution.

## Licence

[MIT](LICENSE)
