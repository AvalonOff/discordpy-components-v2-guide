# Référence discord.py — Classes et méthodes CV2

## Versions

| Version | Date | Changements CV2 |
|---|---|---|
| **2.7.1** | Mars 2026 | Dernière version stable |
| **2.7.0** | Février 2026 | Correctifs |
| **2.6.4** | Octobre 2025 | Correctifs |
| **2.6.0** | Août 2025 | Introduction de CV2 et `LayoutView` |

```bash
# Installer la dernière version stable
pip install -U discord.py

# Vérifier la version
python -c "import discord; print(discord.__version__)"

# Minimum requis
pip install -U "discord.py>=2.6"
```

---

## Classes principales

### `discord.ui.LayoutView`

```python
discord.ui.LayoutView(
    timeout: float | None = 180.0,   # None = persistante
)
```

Point d'entrée de tout message CV2. Remplace `ui.View` pour les layouts V2. Pose automatiquement le flag `IS_COMPONENTS_V2`.

---

### `discord.ui.Container`

```python
discord.ui.Container(
    *items,                           # Composants enfants (positionnels)
    accent_color: Color | int | None = None,
    spoiler: bool = False,
    id: int | None = None,
)
```

Boite visuelle avec bord coloré optionnel. Max 10 enfants directs.

---

### `discord.ui.Section`

```python
discord.ui.Section(
    *items,                           # TextDisplay(s) (positionnels)
    accessory: Thumbnail | Button | None = None,
    id: int | None = None,
)
```

---

### `discord.ui.TextDisplay`

```python
discord.ui.TextDisplay(
    content: str,                     # Markdown, max 4000 chars partagés
    id: int | None = None,
)
```

---

### `discord.ui.Thumbnail`

```python
discord.ui.Thumbnail(
    media: discord.UnfurledMediaItem,
    description: str | None = None,
    spoiler: bool = False,
    id: int | None = None,
)
```

Uniquement en `Section.accessory`.

---

### `discord.ui.MediaGallery`

```python
discord.ui.MediaGallery(
    *items,                           # MediaGalleryItem(s), 1 à 10 (positionnels)
    id: int | None = None,
)
```

---

### `discord.ui.MediaGalleryItem`

```python
discord.ui.MediaGalleryItem(
    media: discord.UnfurledMediaItem,
    description: str | None = None,
    spoiler: bool = False,
    id: int | None = None,
)
```

---

### `discord.ui.File`

```python
discord.ui.File(
    file: discord.UnfurledMediaItem,
    spoiler: bool = False,
    id: int | None = None,
)
```

---

### `discord.ui.Separator`

```python
discord.ui.Separator(
    divider: bool = True,
    spacing: discord.SeparatorSpacingSize = discord.SeparatorSpacingSize.small,
    id: int | None = None,
)
```

---

### `discord.ui.ActionRow`

```python
discord.ui.ActionRow(
    *items,                           # Button(s) ou SelectMenu (positionnels)
)
```

---

### `discord.ui.Button`

```python
discord.ui.Button(
    label: str,
    style: discord.ButtonStyle,
    custom_id: str | None = None,
    url: str | None = None,
    disabled: bool = False,
    emoji: discord.PartialEmoji | str | None = None,
    id: int | None = None,
)
```

---

### `discord.UnfurledMediaItem`

```python
discord.UnfurledMediaItem(
    url: str,    # URL externe ou "attachment://nom_du_fichier"
)
```

---

### `discord.SeparatorSpacingSize`

```python
discord.SeparatorSpacingSize.small   # Valeur API : 1
discord.SeparatorSpacingSize.large   # Valeur API : 2
```

---

## Méthodes d'envoi

### Slash command — `interaction.response.send_message`

```python
await interaction.response.send_message(
    view=MonLayout(),
    ephemeral=True,          # optionnel
    files=[file],            # optionnel, pour ui.File
    allowed_mentions=discord.AllowedMentions.none(),
)
```

### Slash command — `interaction.response.edit_message`

```python
await interaction.response.edit_message(view=NouveauLayout())
```

### Après defer — `interaction.followup.send`

```python
await interaction.response.defer()
# ...
await interaction.followup.send(view=NouveauLayout())
```

### Commande prefix — `ctx.send`

```python
await ctx.send(view=MonLayout())
```

### Salon — `channel.send`

```python
channel = bot.get_channel(CHANNEL_ID)
await channel.send(view=MonLayout())
```

### DM — `user.send`

```python
await user.send(view=MonLayout())
```

### Éditer un message existant — `message.edit`

```python
message = await channel.fetch_message(MESSAGE_ID)
await message.edit(view=NouveauLayout())
```

---

## IDs de type API

| Composant | `type` API |
|---|---|
| ActionRow | `1` |
| Button | `2` |
| StringSelect | `3` |
| TextInput (modal uniquement) | `4` |
| UserSelect | `5` |
| RoleSelect | `6` |
| MentionableSelect | `7` |
| ChannelSelect | `8` |
| Section | `9` |
| TextDisplay | `10` |
| Thumbnail | `11` |
| MediaGallery | `12` |
| File | `13` |
| Separator | `14` |
| Container | `17` |

---

## Gestion des erreurs

```python
import discord
from discord import app_commands

@bot.tree.error
async def on_app_command_error(interaction: discord.Interaction, error: app_commands.AppCommandError):
    if isinstance(error, app_commands.MissingPermissions):
        view = discord.ui.LayoutView()
        view.add_item(discord.ui.Container(
            discord.ui.TextDisplay("Permission refusée."),
            accent_color=discord.Color.red(),
        ))
        await interaction.response.send_message(view=view, ephemeral=True)
        return
    raise error
```

Codes d'erreur API courants pour CV2 :

| Code HTTP | Code Discord | Cause | Solution |
|---|---|---|---|
| 400 | 50035 | Flag manquant (sans LayoutView) | Utiliser `ui.LayoutView` |
| 400 | 50035 | `embeds` présent | Supprimer les embeds |
| 400 | 50035 | `content` non vide | Vider `content=` |
| 400 | 50035 | Trop de composants | Rester sous 40 |
| 400 | 50035 | `stickers` ou `polls` présents | Les supprimer |
| 403 | 50013 | Permission manquante | Vérifier les permissions bot |
| 404 | 10008 | Message introuvable | Vérifier l'ID du message |

---

## Vérification de la version au démarrage

```python
import discord

@bot.event
async def on_ready():
    parts = discord.__version__.split(".")
    major, minor = int(parts[0]), int(parts[1])
    if major < 2 or (major == 2 and minor < 6):
        raise RuntimeError(
            f"discord.py 2.6+ requis pour CV2. Version actuelle : {discord.__version__}"
        )
    print(f"Version discord.py : {discord.__version__} (compatible CV2)")
```

---

## Liens utiles

- [Documentation discord.py](https://discordpy.readthedocs.io)
- [discord.py Changelog](https://discordpy.readthedocs.io/en/latest/whats_new.html)
- [Référence API Discord — Components](https://discord.com/developers/docs/components/reference)
- [Discord Change Log](https://docs.discord.com/developers/change-log)
- [Visual Builder Discord](https://discord.builders)
- [Retour au README](../README.md)
