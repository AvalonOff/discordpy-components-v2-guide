# Media Gallery (type 12)

## Description

`discord.ui.MediaGallery` affiche une collection d'images ou de vidéos (1 à 10) dans une mise en page grille calculée automatiquement par le client Discord. C'est le composant idéal pour les galeries de screenshots, collections d'illustrations, ou rapports visuels.

---

## Paramètres

| Paramètre | Type | Défaut | Description |
|---|---|---|---|
| `*items` | `MediaGalleryItem...` | requis | 1 à 10 items (positionnels) |
| `id` | `int \| None` | `None` | Identifiant optionnel |

---

## MediaGalleryItem

| Paramètre | Type | Défaut | Description |
|---|---|---|---|
| `media` | `UnfurledMediaItem` | requis | Source de l'image ou vidéo |
| `description` | `str \| None` | `None` | Légende / texte alternatif |
| `spoiler` | `bool` | `False` | Cache cet item |
| `id` | `int \| None` | `None` | Identifiant optionnel |

---

## Mise en page automatique

Discord calcule la disposition selon le nombre d'images :

| Nombre | Disposition |
|---|---|
| 1 | Image pleine largeur |
| 2 | Deux images côte à côte |
| 3 | Une grande à gauche, deux petites à droite |
| 4 | Grille 2x2 |
| 5 à 10 | Grille mixte (variable selon le client) |

---

## Exemples

### Galerie simple (style déclaratif)

```python
import discord
from discord import ui

class GalerieLayout(ui.LayoutView):
    container = ui.Container(
        ui.TextDisplay("## Galerie"),
        ui.MediaGallery(
            ui.MediaGalleryItem(
                media=discord.UnfurledMediaItem(url="https://exemple.com/img1.png"),
                description="Screenshot 1",
            ),
            ui.MediaGalleryItem(
                media=discord.UnfurledMediaItem(url="https://exemple.com/img2.png"),
                description="Screenshot 2",
            ),
            ui.MediaGalleryItem(
                media=discord.UnfurledMediaItem(url="https://exemple.com/img3.png"),
                description="Screenshot 3",
            ),
            ui.MediaGalleryItem(
                media=discord.UnfurledMediaItem(url="https://exemple.com/img4.png"),
                description="Screenshot 4",
            ),
        ),
    )
```

### Galerie depuis des pièces jointes

```python
@bot.tree.command(name="galerie")
async def galerie(interaction: discord.Interaction):
    if not interaction.message or not interaction.message.attachments:
        await interaction.response.send_message("Joignez des images à votre commande.", ephemeral=True)
        return

    images = [
        a for a in interaction.message.attachments
        if a.content_type and a.content_type.startswith("image/")
    ][:10]

    if not images:
        await interaction.response.send_message("Aucune image valide trouvée.", ephemeral=True)
        return

    view = ui.LayoutView()
    container = ui.Container(accent_color=discord.Color.blurple())
    container.add_item(ui.TextDisplay(f"## Galerie — {len(images)} image(s)"))
    gallery = ui.MediaGallery(
        *[
            ui.MediaGalleryItem(
                media=discord.UnfurledMediaItem(url=a.url),
                description=a.filename,
            )
            for a in images
        ]
    )
    container.add_item(gallery)
    view.add_item(container)
    await interaction.response.send_message(view=view)
```

### Galerie construite dynamiquement

```python
def build_gallery(urls: list[str], descriptions: list[str] | None = None) -> ui.MediaGallery:
    if not urls:
        raise ValueError("La liste d'URLs ne peut pas être vide.")
    items = []
    for i, url in enumerate(urls[:10]):
        desc = descriptions[i] if descriptions and i < len(descriptions) else None
        items.append(ui.MediaGalleryItem(
            media=discord.UnfurledMediaItem(url=url),
            description=desc,
        ))
    return ui.MediaGallery(*items)
```

### Galerie avec spoiler partiel

```python
class GalerieSpoilerLayout(ui.LayoutView):
    container = ui.Container(
        ui.MediaGallery(
            ui.MediaGalleryItem(
                media=discord.UnfurledMediaItem(url="https://exemple.com/public.png"),
                description="Image publique",
                spoiler=False,
            ),
            ui.MediaGalleryItem(
                media=discord.UnfurledMediaItem(url="https://exemple.com/sensible.png"),
                description="Contenu sensible",
                spoiler=True,
            ),
        ),
    )
```

Chaque item peut être spoiler indépendamment.

---

## Payload API brut

```json
{
  "type": 12,
  "items": [
    {
      "media": { "url": "https://exemple.com/img1.png" },
      "description": "Image 1",
      "spoiler": false
    },
    {
      "media": { "url": "https://exemple.com/img2.png" },
      "description": "Image 2",
      "spoiler": true
    }
  ]
}
```

---

## Notes importantes

- Minimum **1 item**, maximum **10 items**
- Les URLs doivent être publiquement accessibles (ou `attachment://`)
- Les GIFs animés sont supportés
- `description` s'affiche comme légende et texte alternatif
- Le spoiler s'applique **image par image** — pas à toute la galerie
- La galerie compte pour **1 composant** dans la limite de 40 (pas 1 par image)

---

## Liens

- [Thumbnail](07-thumbnail.md)
- [File](10-file.md)
- [Retour au README](../README.md)
