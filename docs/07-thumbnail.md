# Thumbnail (type 11)

## Description

`discord.ui.Thumbnail` est un composant image utilisé **uniquement comme accessoire d'une `Section`**. Il s'affiche à droite du texte de la section, en format miniature. Il ne peut pas être placé directement dans un Container ou à la racine d'une LayoutView.

Pour afficher des images en grand ou plusieurs images, utilisez `ui.MediaGallery`.

---

## Paramètres

| Paramètre | Type | Défaut | Description |
|---|---|---|---|
| `media` | `UnfurledMediaItem` | requis | Source de l'image |
| `description` | `str \| None` | `None` | Texte alternatif (accessibilité) |
| `spoiler` | `bool` | `False` | Cache l'image sous un filtre spoiler |
| `id` | `int \| None` | `None` | Identifiant optionnel |

---

## UnfurledMediaItem

```python
# URL externe publique
discord.UnfurledMediaItem(url="https://exemple.com/avatar.png")

# Fichier joint au message
discord.UnfurledMediaItem(url="attachment://avatar.png")
```

Formats supportés : PNG, JPG, GIF animé, WebP. Les vidéos ne sont pas supportées dans un Thumbnail.

---

## Exemples

### Thumbnail avec URL externe

```python
import discord
from discord import ui

class MembreLayout(ui.LayoutView):
    container = ui.Container(
        ui.Section(
            ui.TextDisplay("**Alice Martin**\nAdministratrice"),
            accessory=ui.Thumbnail(
                media=discord.UnfurledMediaItem(url="https://exemple.com/alice.png"),
                description="Avatar d'Alice",
            ),
        ),
    )
```

### Thumbnail avec l'avatar Discord d'un membre

```python
@bot.tree.command(name="profil")
@discord.app_commands.describe(membre="Membre à afficher")
async def profil(interaction: discord.Interaction, membre: discord.Member = None):
    m = membre or interaction.user

    view = ui.LayoutView()
    container = ui.Container(
        ui.Section(
            ui.TextDisplay(f"## {m.display_name}\n**ID :** `{m.id}`"),
            accessory=ui.Thumbnail(
                media=discord.UnfurledMediaItem(url=m.display_avatar.url),
                description=f"Avatar de {m.display_name}",
            ),
        ),
    )
    view.add_item(container)
    await interaction.response.send_message(view=view)
```

### Thumbnail avec fichier joint

```python
@bot.tree.command(name="icone")
async def icone(interaction: discord.Interaction):
    file = discord.File("assets/icone.png", filename="icone.png")

    view = ui.LayoutView()
    container = ui.Container(
        ui.Section(
            ui.TextDisplay("**Icône du serveur**\nVersion 2026"),
            accessory=ui.Thumbnail(
                media=discord.UnfurledMediaItem(url="attachment://icone.png"),
            ),
        ),
    )
    view.add_item(container)
    await interaction.response.send_message(view=view, files=[file])
```

### Thumbnail en mode spoiler

```python
class SpoilerSection(ui.LayoutView):
    container = ui.Container(
        ui.Section(
            ui.TextDisplay("Image sensible — cliquez pour voir."),
            accessory=ui.Thumbnail(
                media=discord.UnfurledMediaItem(url="https://exemple.com/image.png"),
                spoiler=True,
            ),
        ),
    )
```

---

## Thumbnail vs MediaGallery

| Critère | Thumbnail | MediaGallery |
|---|---|---|
| Position | Accessoire de Section (droite) | Composant indépendant (pleine largeur) |
| Nombre d'images | 1 | 1 à 10 |
| Taille | Miniature | Grande, mise en page auto |
| Usage | Avatar, icône, logo produit | Galerie photos, screenshots |
| Peut être dans Container | Non directement | Oui |

---

## Payload API brut

```json
{
  "type": 11,
  "media": {
    "url": "https://exemple.com/alice.png"
  },
  "description": "Avatar d'Alice",
  "spoiler": false
}
```

---

## Notes importantes

- Le Thumbnail ne peut **pas** être placé directement dans un Container
- Il doit toujours être dans `Section.accessory`
- Les GIFs animés sont supportés
- La `description` sert de texte alternatif pour l'accessibilité — remplissez-la systématiquement

---

## Liens

- [Section](05-section.md)
- [Media Gallery](09-media-gallery.md)
- [Retour au README](../README.md)
