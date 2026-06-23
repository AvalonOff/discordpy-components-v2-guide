# Flag IS_COMPONENTS_V2

## Valeur et rôle

Le flag `IS_COMPONENTS_V2` a la valeur brute `1 << 15` = `32768`. Il doit être présent sur tout message qui utilise des composants CV2. Sans lui, l'API Discord renvoie une erreur `50035` et le message n'est pas envoyé.

**Avec discord.py 2.6+** et `ui.LayoutView`, le flag est **posé automatiquement**. Vous n'avez pas à le spécifier manuellement dans la plupart des cas.

---

## Avec LayoutView (discord.py 2.6+)

La méthode recommandée. Le flag est géré par discord.py :

```python
import discord
from discord import ui

class MonLayout(ui.LayoutView):
    text = ui.TextDisplay("## Bonjour !")

# slash command
await interaction.response.send_message(view=MonLayout())

# commande prefix
await ctx.send(view=MonLayout())

# message direct dans un salon
await channel.send(view=MonLayout())

# DM
await user.send(view=MonLayout())
```

---

## Avec le flag explicite (approche bas niveau)

Si vous utilisez `components=` directement (par exemple pour une transition progressive ou du code hérité) :

```python
flags = discord.MessageFlags(is_components_v2=True)

# Slash command
await interaction.response.send_message(
    components=[container],
    flags=flags,
)

# Après defer
await interaction.response.defer()
await interaction.followup.send(components=[container], flags=flags)

# Message direct
await channel.send(components=[container], flags=flags)
```

---

## Combiner avec d'autres flags

```python
# Message éphémère + CV2
await interaction.response.send_message(view=MonLayout(), ephemeral=True)

# Via flags explicites
flags = discord.MessageFlags(is_components_v2=True, ephemeral=True)
await interaction.response.send_message(components=[...], flags=flags)
```

Valeurs brutes des flags courants :

| Flag | Valeur brute |
|---|---|
| `suppress_embeds` | `1 << 2` = `4` |
| `ephemeral` | `1 << 6` = `64` |
| `suppress_notifications` | `1 << 12` = `4096` |
| `is_components_v2` | `1 << 15` = `32768` |

---

## Ce que le flag change

Quand `IS_COMPONENTS_V2` est actif sur un message :

| Élément | Comportement |
|---|---|
| `content` | Ne fonctionne plus — doit être vide ou absent |
| `embeds` | Incompatibles — rejetés par l'API |
| `stickers` | Incompatibles — rejetés par l'API |
| `polls` | Incompatibles — rejetés par l'API |
| Pièces jointes | Ne s'affichent plus automatiquement — nécessitent `ui.File` |
| Composants CV2 | Activés et rendus par le client Discord |
| Markdown dans TextDisplay | Complet (H1/H2/H3, listes, code blocks...) |

> [!WARNING]
> Mentions et pings : le texte dans `ui.TextDisplay` **déclenche des pings** normalement, même si le TextDisplay est dans un Container. Utilisez `discord.AllowedMentions` pour contrôler ce comportement.

---

## Le flag est permanent

Une fois qu'un message est envoyé avec `IS_COMPONENTS_V2`, ce flag ne peut **jamais** être retiré. Vous pouvez éditer les composants du message, mais il ne peut pas revenir à un message V1.

---

## Éditer un message CV2

```python
# Via interaction (bouton cliqué)
await interaction.response.edit_message(view=NouveauLayout())

# Via l'objet message
message = await channel.fetch_message(message_id)
await message.edit(view=NouveauLayout())
```

Si vous éditez avec `components=` et le flag explicite :

```python
await interaction.response.edit_message(
    components=[nouveau_container],
    flags=discord.MessageFlags(is_components_v2=True),
)
```

---

## Payload API brut (sans discord.py)

Pour du code qui n'utilise pas discord.py ou une version antérieure à 2.6 :

```python
from discord.http import Route

async def send_cv2_raw(bot, channel_id: int, components: list) -> dict:
    payload = {
        "components": components,
        "flags": 1 << 15,
    }
    route = Route("POST", "/channels/{channel_id}/messages", channel_id=channel_id)
    return await bot.http.request(route, json=payload)

# Usage
await send_cv2_raw(bot, channel_id=123456, components=[
    {
        "type": 17,
        "components": [
            {"type": 10, "content": "# Bonjour depuis le payload brut"}
        ]
    }
])
```

---

## Erreurs courantes

| Code Discord | Message | Cause | Solution |
|---|---|---|---|
| `50035` | Invalid Form Body | Flag absent | Utiliser `LayoutView` ou `flags=discord.MessageFlags(is_components_v2=True)` |
| `50035` | Invalid Form Body | `embeds` présent | Supprimer les embeds |
| `50035` | Invalid Form Body | `content` non vide | Vider `content=` |
| `50035` | Invalid Form Body | Trop de composants | Rester sous 40 composants |
| `50035` | Invalid Form Body | `stickers` présent | Supprimer les stickers |

---

## Contrôle des mentions

```python
class MonLayout(ui.LayoutView):
    text = ui.TextDisplay("Bonjour <@123456789> !")  # pinge l'utilisateur

# Désactiver tous les pings
await interaction.response.send_message(
    view=MonLayout(),
    allowed_mentions=discord.AllowedMentions.none(),
)
```

---

## Liens

- [Architecture et LayoutView](02-architecture.md)
- [Container](04-container.md)
- [Retour au README](../README.md)
