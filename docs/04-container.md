# Container (type 17)

## Description

`discord.ui.Container` est l'enveloppe visuelle principale de CV2. Il groupe des composants et peut afficher un **bord coloré** sur son côté gauche — identique visuellement aux embeds classiques. C'est le seul composant qui peut contenir tous les autres types (sauf un autre Container).

Un Container n'est pas obligatoire : les composants peuvent être placés directement à la racine de la `LayoutView`. Mais dès qu'on veut un bord coloré, un regroupement visuel cohérent, ou un mode spoiler, le Container est indispensable.

---

## Paramètres

| Paramètre | Type | Défaut | Description |
|---|---|---|---|
| `*items` | `Component...` | requis | Composants enfants (positionnels) |
| `accent_color` | `Color \| int \| None` | `None` | Couleur du bord gauche |
| `spoiler` | `bool` | `False` | Cache tout le contenu |
| `id` | `int \| None` | `None` | Identifiant optionnel |

Les enfants se passent en **positionnels** dans le constructeur, puis via `add_item()` pour le style impératif.

---

## Enfants acceptés

Un Container peut contenir (max **10 enfants directs**) :

- `ui.TextDisplay`
- `ui.Section`
- `ui.Separator`
- `ui.MediaGallery`
- `ui.File`
- `ui.ActionRow`

Il ne peut **pas** contenir un autre `Container`.

---

## Exemples

### Style déclaratif — Container statique

```python
import discord
from discord import ui

class InfoLayout(ui.LayoutView):

    class InfoContainer(ui.Container):
        titre  = ui.TextDisplay("## Statut du serveur")
        sep    = ui.Separator(divider=True, spacing=discord.SeparatorSpacingSize.small)
        corps  = ui.TextDisplay("Tous les systèmes sont opérationnels.")

    container = InfoContainer(accent_color=discord.Color.green())

@bot.tree.command(name="statut")
async def statut(interaction: discord.Interaction):
    await interaction.response.send_message(view=InfoLayout())
```

### Style impératif — Container dynamique

```python
def build_statut(services: dict[str, bool]) -> ui.LayoutView:
    view = ui.LayoutView()
    container = ui.Container(accent_color=discord.Color.blurple())
    container.add_item(ui.TextDisplay("## Statut des services"))
    container.add_item(ui.Separator(divider=True, spacing=discord.SeparatorSpacingSize.small))

    for name, online in services.items():
        status = "En ligne" if online else "Hors ligne"
        container.add_item(ui.TextDisplay(f"**{name}** : {status}"))

    view.add_item(container)
    return view

# Usage
services = {"API": True, "Base de données": True, "Cache": False}
await interaction.response.send_message(view=build_statut(services))
```

### Constructeur direct avec enfants positionnels

```python
class RapportLayout(ui.LayoutView):
    container = ui.Container(
        ui.TextDisplay("## Rapport mensuel"),
        ui.Separator(divider=True, spacing=discord.SeparatorSpacingSize.large),
        ui.TextDisplay("**Ventes :** 4 200 unités\n**Revenus :** 84 000 EUR"),
        ui.Separator(divider=False, spacing=discord.SeparatorSpacingSize.small),
        ui.ActionRow(
            ui.Button(label="Exporter PDF", custom_id="export:pdf", style=discord.ButtonStyle.primary),
            ui.Button(label="Partager", custom_id="export:partager", style=discord.ButtonStyle.secondary),
        ),
        accent_color=discord.Color.blurple(),
    )
```

### Mode spoiler

```python
class SpoilerLayout(ui.LayoutView):
    container = ui.Container(
        ui.TextDisplay("Contenu sensible — cliquez pour révéler."),
        ui.MediaGallery(
            ui.MediaGalleryItem(
                media=discord.UnfurledMediaItem(url="https://exemple.com/image.png")
            )
        ),
        spoiler=True,
    )
```

Le mode `spoiler=True` masque **tout** le contenu du Container — texte, images et boutons — jusqu'à ce que l'utilisateur clique.

### Plusieurs Containers dans une LayoutView

```python
class MultiLayout(ui.LayoutView):

    class InfoC(ui.Container):
        text = ui.TextDisplay("## Informations")

    class WarnC(ui.Container):
        text = ui.TextDisplay("## Avertissement")

    class ErrC(ui.Container):
        text = ui.TextDisplay("## Erreur critique")

    info = InfoC(accent_color=discord.Color.blurple())
    warn = WarnC(accent_color=discord.Color.yellow())
    err  = ErrC(accent_color=discord.Color.red())
```

---

## Palette de couleurs Discord

```python
discord.Color.blurple()     # #5865F2
discord.Color.green()       # #57F287
discord.Color.yellow()      # #FEE75C
discord.Color.red()         # #ED4245
discord.Color.gold()        # #F1C40F
discord.Color.orange()      # #E67E22
discord.Color.teal()        # #1ABC9C
discord.Color.purple()      # #9B59B6
discord.Color.dark_blue()   # #206694
discord.Color.dark_gray()   # #607D8B

# Couleur hexadécimale
discord.Color(0xFF6B35)

# Entier décimal direct
ui.Container(accent_color=0x5865F2)
```

---

## Payload API brut

```json
{
  "type": 17,
  "accent_color": 5789778,
  "spoiler": false,
  "components": [
    { "type": 10, "content": "Contenu du container" }
  ]
}
```

`accent_color` est un entier décimal RGB. Exemple : `#5865F2` = `5789778`.

---

## Notes importantes

- Un Container ne peut **pas** contenir un autre Container
- `accent_color=None` = aucun bord coloré
- La limite de 10 enfants est pour les enfants **directs** du Container
- Le spoiler masque tout le contenu du Container d'un coup
- Les composants passés en positionnels au constructeur ET via `add_item()` sont cumulables

---

## Liens

- [Section](05-section.md)
- [Separator](08-separator.md)
- [Retour au README](../README.md)
