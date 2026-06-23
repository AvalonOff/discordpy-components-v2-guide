# File (type 13)

## Description

`discord.ui.File` affiche une pièce jointe dans le flux de composants CV2. Contrairement aux pièces jointes classiques qui apparaissent automatiquement en bas du message, `ui.File` s'intègre dans la mise en page au sein d'un Container, à l'endroit exact où le développeur le place.

> [!IMPORTANT]
> En CV2, les pièces jointes **ne s'affichent plus automatiquement**. Elles doivent obligatoirement être exposées via `ui.File` (ou `ui.Thumbnail`, `ui.MediaGallery`) pour apparaître dans le message.

---

## Paramètres

| Paramètre | Type | Défaut | Description |
|---|---|---|---|
| `file` | `UnfurledMediaItem` | requis | Référence vers la pièce jointe |
| `spoiler` | `bool` | `False` | Cache la pièce jointe |
| `id` | `int \| None` | `None` | Identifiant optionnel |

`ui.File` ne supporte **pas** de `description` (texte alternatif), contrairement à `Thumbnail` et `MediaGallery`.

---

## Convention attachment://

La convention `attachment://nom_du_fichier` référence un fichier joint au message. Le nom doit correspondre exactement au `filename` du `discord.File`.

```python
# 1. Préparer le fichier
file = discord.File("chemin/vers/rapport.pdf", filename="rapport.pdf")

# 2. Créer le composant
file_component = discord.ui.File(
    file=discord.UnfurledMediaItem(url="attachment://rapport.pdf"),
)

# 3. Envoyer
view = discord.ui.LayoutView()
container = discord.ui.Container(file_component)
view.add_item(container)
await interaction.response.send_message(view=view, files=[file])
```

---

## Exemples

### PDF dans un rapport

```python
import discord
from discord import ui

@bot.tree.command(name="rapport")
async def rapport(interaction: discord.Interaction):
    file = discord.File("rapports/rapport_juin.pdf", filename="rapport_juin.pdf")

    view = ui.LayoutView()
    container = ui.Container(
        ui.Section(
            ui.TextDisplay(
                "## Rapport mensuel — Juin 2026\n"
                "**Généré le :** <t:1750000000:F>\n"
                "**Pages :** 14 | **Taille :** 2,3 Mo"
            ),
        ),
        ui.Separator(divider=True, spacing=discord.SeparatorSpacingSize.small),
        ui.File(file=discord.UnfurledMediaItem(url="attachment://rapport_juin.pdf")),
        accent_color=discord.Color.blurple(),
    )
    view.add_item(container)
    await interaction.response.send_message(view=view, files=[file])
```

### Export CSV avec contexte

```python
import io
import csv

@bot.tree.command(name="exporter")
async def exporter(interaction: discord.Interaction):
    output = io.StringIO()
    writer = csv.writer(output)
    writer.writerow(["ID", "Nom", "Date"])
    writer.writerow([1, "Alice", "2026-06-22"])
    writer.writerow([2, "Bob",   "2026-06-23"])
    output.seek(0)

    file = discord.File(io.BytesIO(output.read().encode()), filename="export.csv")

    view = ui.LayoutView()
    container = ui.Container(
        ui.TextDisplay("## Export des données\n**Format :** CSV"),
        ui.File(file=discord.UnfurledMediaItem(url="attachment://export.csv")),
    )
    view.add_item(container)
    await interaction.response.send_message(view=view, files=[file])
```

### Plusieurs fichiers dans un message

```python
@bot.tree.command(name="docs")
async def docs(interaction: discord.Interaction):
    files = [
        discord.File("doc1.pdf", filename="doc1.pdf"),
        discord.File("doc2.pdf", filename="doc2.pdf"),
    ]

    view = ui.LayoutView()
    container = ui.Container(
        ui.TextDisplay("## Documents joints"),
        ui.File(file=discord.UnfurledMediaItem(url="attachment://doc1.pdf")),
        ui.Separator(divider=False, spacing=discord.SeparatorSpacingSize.small),
        ui.File(file=discord.UnfurledMediaItem(url="attachment://doc2.pdf")),
    )
    view.add_item(container)
    await interaction.response.send_message(view=view, files=files)
```

### Fichier en mode spoiler

```python
view = ui.LayoutView()
container = ui.Container(
    ui.TextDisplay("Document confidentiel — cliquez pour voir."),
    ui.File(
        file=discord.UnfurledMediaItem(url="attachment://confidentiel.pdf"),
        spoiler=True,
    ),
)
view.add_item(container)
```

---

## Différences avec les pièces jointes V1

| Critère | V1 (pièce jointe classique) | CV2 (ui.File) |
|---|---|---|
| Affichage automatique | Oui | Non — doit passer par `ui.File` |
| Position | Toujours en bas du message | N'importe où dans le Container |
| Contexte visuel | Aucun | Entouré de TextDisplay, Separator... |
| Texte alternatif | Non | Non (pas de `description`) |
| Spoiler | Via filename `SPOILER_` | Via `spoiler=True` |

---

## Payload API brut

```json
{
  "type": 13,
  "file": {
    "url": "attachment://rapport.pdf"
  },
  "spoiler": false
}
```

---

## Notes importantes

- Le fichier doit être joint via `files=[...]` dans le `send_message()` ET référencé dans `ui.File`
- Le `filename` de `discord.File` et l'URL `attachment://` doivent correspondre exactement
- `ui.File` ne supporte pas de `description`
- En CV2, aucune pièce jointe n'est visible sans `ui.File`, `ui.Thumbnail` ou `ui.MediaGallery`

---

## Liens

- [Media Gallery](09-media-gallery.md)
- [Retour au README](../README.md)
