# Select Menus dans CV2

Les Select Menus sont des composants interactifs qui s'insèrent dans une `ActionRow`, exactement comme les boutons. Il en existe cinq types selon la nature des options proposées.

---

## Les 5 types de Select Menu

| Classe | Type API | Ce que l'utilisateur choisit |
|---|---|---|
| `ui.Select` (StringSelect) | `3` | Options texte que vous définissez vous-même |
| `ui.UserSelect` | `5` | Un ou plusieurs membres du serveur |
| `ui.RoleSelect` | `6` | Un ou plusieurs rôles |
| `ui.MentionableSelect` | `7` | Membres ou rôles |
| `ui.ChannelSelect` | `8` | Un ou plusieurs salons |

> [!IMPORTANT]
> Un `ActionRow` contenant un Select Menu **ne peut pas** contenir de boutons. Un Select par `ActionRow`.

---

## StringSelect — Options texte libres

Le plus courant. Vous définissez chaque option avec un `SelectOption`.

### Paramètres

| Paramètre | Type | Défaut | Description |
|---|---|---|---|
| `custom_id` | `str` | obligatoire | Identifiant du callback (max 100 chars) |
| `options` | `list[SelectOption]` | `[]` | Liste des options affichées |
| `placeholder` | `str \| None` | `None` | Texte grisé quand rien n'est sélectionné |
| `min_values` | `int` | `1` | Minimum d'options sélectionnables |
| `max_values` | `int` | `1` | Maximum d'options sélectionnables |
| `disabled` | `bool` | `False` | Désactive le menu |

### `discord.SelectOption`

| Paramètre | Type | Description |
|---|---|---|
| `label` | `str` | Texte affiché (max 100 chars) |
| `value` | `str` | Valeur reçue dans le callback (max 100 chars) |
| `description` | `str \| None` | Sous-titre optionnel (max 100 chars) |
| `emoji` | `PartialEmoji \| str \| None` | Emoji optionnel |
| `default` | `bool` | Pré-sélectionné |

### Exemple — Sélection de rôle (options texte)

```python
import discord
from discord import ui

class RoleMenuLayout(ui.LayoutView):
    container = ui.Container(
        ui.TextDisplay("## Choisissez votre rôle"),
        ui.Separator(divider=True, spacing=discord.SeparatorSpacingSize.small),
        ui.ActionRow(
            ui.Select(
                custom_id="role:choisir",
                placeholder="Sélectionnez un rôle...",
                options=[
                    discord.SelectOption(
                        label="Développeur",
                        value="dev",
                        description="Accès aux salons de développement",
                        emoji=discord.PartialEmoji(name="\U0001f4bb"),
                    ),
                    discord.SelectOption(
                        label="Designer",
                        value="design",
                        description="Accès aux salons créatifs",
                        emoji=discord.PartialEmoji(name="\U0001f3a8"),
                    ),
                    discord.SelectOption(
                        label="Modérateur",
                        value="mod",
                        description="Accès aux outils de modération",
                        emoji=discord.PartialEmoji(name="\U0001f6e1"),
                    ),
                ],
                min_values=1,
                max_values=1,
            )
        ),
        accent_color=discord.Color.blurple(),
    )

@bot.event
async def on_interaction(interaction: discord.Interaction):
    if interaction.type != discord.InteractionType.component:
        return
    if interaction.data.get("custom_id") == "role:choisir":
        valeurs = interaction.data.get("values", [])
        role_choisi = valeurs[0] if valeurs else None
        await interaction.response.send_message(
            f"Role sélectionné : `{role_choisi}`", ephemeral=True
        )
```

### Exemple — Sélection multiple

```python
ui.Select(
    custom_id="notifs:types",
    placeholder="Quelles notifications voulez-vous recevoir ?",
    min_values=1,
    max_values=3,  # l'utilisateur peut en choisir 1, 2 ou 3
    options=[
        discord.SelectOption(label="Annonces",      value="annonces"),
        discord.SelectOption(label="Mises à jour",  value="updates"),
        discord.SelectOption(label="Événements",    value="events"),
    ],
)
```

---

## UserSelect — Sélection de membres

```python
import discord
from discord import ui

class MentionLayout(ui.LayoutView):
    container = ui.Container(
        ui.TextDisplay("## Mentionner un membre"),
        ui.ActionRow(
            ui.UserSelect(
                custom_id="user:mentionner",
                placeholder="Choisissez un membre...",
                min_values=1,
                max_values=1,
            )
        ),
    )

@bot.event
async def on_interaction(interaction: discord.Interaction):
    if interaction.type != discord.InteractionType.component:
        return
    if interaction.data.get("custom_id") == "user:mentionner":
        values = interaction.data.get("resolved", {}).get("users", {})
        user_id = interaction.data.get("values", [None])[0]
        await interaction.response.send_message(
            f"Membre choisi : <@{user_id}>",
            allowed_mentions=discord.AllowedMentions.none(),
            ephemeral=True,
        )
```

---

## RoleSelect — Sélection de rôles

```python
class AssignRoleLayout(ui.LayoutView):
    container = ui.Container(
        ui.TextDisplay("## Assigner un rôle"),
        ui.ActionRow(
            ui.RoleSelect(
                custom_id="admin:role:assigner",
                placeholder="Choisissez un rôle à assigner...",
            )
        ),
    )
```

---

## MentionableSelect — Membres ou rôles

```python
class PingLayout(ui.LayoutView):
    container = ui.Container(
        ui.TextDisplay("## Qui voulez-vous notifier ?"),
        ui.ActionRow(
            ui.MentionableSelect(
                custom_id="ping:cible",
                placeholder="Membre ou rôle...",
                min_values=1,
                max_values=5,
            )
        ),
    )
```

---

## ChannelSelect — Sélection de salons

```python
class LogsLayout(ui.LayoutView):
    container = ui.Container(
        ui.TextDisplay("## Choisissez le salon des logs"),
        ui.ActionRow(
            ui.ChannelSelect(
                custom_id="config:logs:salon",
                placeholder="Salon pour les logs...",
                channel_types=[
                    discord.ChannelType.text,
                    discord.ChannelType.news,
                ],
            )
        ),
    )
```

> [!NOTE]
> `channel_types` permet de filtrer les types de salon affichés. Si omis, tous les types de salon sont proposés.

---

## Callback complet avec routage multi-select

```python
SELECT_HANDLERS = {
    "role:choisir":      handle_role_select,
    "notifs:types":      handle_notifs_select,
    "config:logs:salon": handle_logs_channel_select,
}

@bot.event
async def on_interaction(interaction: discord.Interaction):
    if interaction.type != discord.InteractionType.component:
        return

    cid      = interaction.data.get("custom_id", "")
    values   = interaction.data.get("values", [])
    handler  = SELECT_HANDLERS.get(cid)

    if handler:
        await handler(interaction, values)
```

---

## Boutons vs Select Menus — quand choisir quoi ?

| Situation | Recommandation |
|---|---|
| 2 à 5 choix simples, action immédiate | Boutons |
| Plus de 5 choix | Select Menu |
| Choix d'un utilisateur/rôle/salon | UserSelect / RoleSelect / ChannelSelect |
| Choix multiple (cocher plusieurs cases) | Select avec `max_values > 1` |
| Navigation entre écrans | Boutons |
| Filtre ou configuration | Select Menu |

---

## Règles importantes

- Un `ActionRow` contient **soit** des boutons, **soit** un select — jamais les deux
- Maximum **1 select** par `ActionRow`
- `min_values` doit être inférieur ou égal à `max_values`
- `max_values` ne peut pas dépasser le nombre d'options définies (pour StringSelect)
- `custom_id` : max 100 caractères, unique dans la LayoutView

---

## Liens

- [Action Rows et boutons](11-action-rows.md)
- [Bonnes pratiques](14-bonnes-pratiques.md)
- [Exemples pratiques](12-exemples.md)
- [Retour au README](../README.md)
