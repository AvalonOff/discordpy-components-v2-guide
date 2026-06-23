# Action Row et composants interactifs

## Description

Les `ActionRow` dans CV2 fonctionnent comme dans le système classique, mais avec un avantage clé : **ils peuvent être placés à n'importe quelle position dans un Container**. Les boutons ne sont plus contraints d'être en bas du message — ils peuvent être exactement en face du contenu qu'ils contrôlent.

---

## Règles de composition

| Élément | Limite |
|---|---|
| Boutons par ActionRow | 5 maximum |
| Select menus par ActionRow | 1 (exclusif des boutons) |
| Composants totaux par LayoutView | 40 (tous types confondus) |

---

## Boutons

### Styles disponibles

| Style | Constante | Couleur | Usage |
|---|---|---|---|
| Primary | `ButtonStyle.primary` | Bleu | Action principale |
| Secondary | `ButtonStyle.secondary` | Gris | Action secondaire |
| Success | `ButtonStyle.success` | Vert | Confirmation |
| Danger | `ButtonStyle.danger` | Rouge | Action destructive |
| Link | `ButtonStyle.link` | Gris + icône externe | Lien URL |

### Paramètres Button

| Paramètre | Type | Description |
|---|---|---|
| `label` | `str` | Texte du bouton (max 80 chars) |
| `style` | `ButtonStyle` | Style visuel |
| `custom_id` | `str` | Identifiant pour les callbacks (max 100 chars) |
| `url` | `str` | URL (uniquement pour `style=link`) |
| `disabled` | `bool` | Désactive le bouton |
| `emoji` | `PartialEmoji \| str` | Emoji optionnel |

---

## Exemples de boutons

### Style déclaratif

```python
import discord
from discord import ui

class AdminLayout(ui.LayoutView):

    class AdminContainer(ui.Container):
        titre = ui.TextDisplay("## Panneau d'administration")
        sep   = ui.Separator(divider=True, spacing=discord.SeparatorSpacingSize.small)

        actions_membres = ui.ActionRow(
            ui.Button(label="Voir membres", custom_id="admin:membres:voir", style=discord.ButtonStyle.primary),
            ui.Button(label="Bannir",       custom_id="admin:membres:bannir", style=discord.ButtonStyle.danger),
            ui.Button(label="Expulser",     custom_id="admin:membres:kick",   style=discord.ButtonStyle.danger),
        )

        sep2 = ui.Separator(divider=False, spacing=discord.SeparatorSpacingSize.small)
        sep3 = ui.TextDisplay("**Gestion des salons**")

        actions_salons = ui.ActionRow(
            ui.Button(label="Verrouiller", custom_id="admin:salons:lock",    style=discord.ButtonStyle.danger),
            ui.Button(label="Archiver",    custom_id="admin:salons:archiver", style=discord.ButtonStyle.secondary),
        )

    container = AdminContainer(accent_color=discord.Color.dark_red())
```

### Style impératif avec boutons dynamiques

```python
def build_pagination(current: int, total: int, prefix: str) -> ui.ActionRow:
    row = ui.ActionRow(
        ui.Button(
            label="Précédent",
            custom_id=f"{prefix}:prev:{current}",
            style=discord.ButtonStyle.secondary,
            disabled=current <= 1,
        ),
        ui.Button(
            label=f"Page {current}/{total}",
            custom_id=f"{prefix}:current",
            style=discord.ButtonStyle.secondary,
            disabled=True,
        ),
        ui.Button(
            label="Suivant",
            custom_id=f"{prefix}:next:{current}",
            style=discord.ButtonStyle.secondary,
            disabled=current >= total,
        ),
    )
    return row
```

### Boutons avec emojis

```python
class ConfirmLayout(ui.LayoutView):
    container = ui.Container(
        ui.TextDisplay("Confirmez-vous la suppression ?"),
        ui.ActionRow(
            ui.Button(label="Confirmer", custom_id="confirm", style=discord.ButtonStyle.danger,
                      emoji=discord.PartialEmoji(name="\U0001f5d1")),
            ui.Button(label="Annuler",   custom_id="cancel",  style=discord.ButtonStyle.secondary,
                      emoji=discord.PartialEmoji(name="\u274c")),
        ),
    )
```

---

## Select Menus

### StringSelect dans un Container CV2

```python
class LangueLayout(ui.LayoutView):
    container = ui.Container(
        ui.TextDisplay("## Choisissez votre langue"),
        ui.ActionRow(
            ui.Select(
                custom_id="langue:select",
                placeholder="Sélectionnez une langue",
                options=[
                    discord.SelectOption(label="Français", value="fr", description="Interface en français"),
                    discord.SelectOption(label="English",  value="en", description="English interface"),
                    discord.SelectOption(label="Español",  value="es"),
                ],
            )
        ),
    )
```

---

## Positionnement stratégique des ActionRows

L'avantage CV2 est de placer les boutons exactement en face du contenu qu'ils contrôlent :

```
Container
├── TextDisplay "## Paramètres du compte"
├── Separator
├── Section (nom + avatar)
├── ActionRow [Modifier le profil] [Changer l'avatar]   <- collé au profil
├── Separator
├── TextDisplay "Notifications"
├── ActionRow [Activer tout] [Désactiver tout]           <- collé aux notifs
├── Separator
└── ActionRow [Sauvegarder] [Annuler]                   <- actions globales
```

---

## Gestion des callbacks

### Via on_interaction avec routage par préfixe

```python
@bot.event
async def on_interaction(interaction: discord.Interaction):
    if interaction.type != discord.InteractionType.component:
        return

    cid = interaction.data.get("custom_id", "")
    parts = cid.split(":")

    if not parts:
        return

    namespace = parts[0]

    if namespace == "admin":
        await handle_admin(interaction, parts[1:])
    elif namespace == "plan":
        await handle_plan(interaction, parts[1:])


async def handle_admin(interaction: discord.Interaction, parts: list[str]):
    action = ":".join(parts)
    if action == "membres:bannir":
        if not interaction.user.guild_permissions.ban_members:
            await interaction.response.send_message("Permission refusée.", ephemeral=True)
            return
        await interaction.response.send_message("Ouvre le panneau de bannissement.", ephemeral=True)
```

### Via LayoutView avec @button décorateur

```python
class MenuLayout(ui.LayoutView):
    container = ui.Container(
        ui.TextDisplay("## Menu principal"),
        ui.ActionRow(),
        accent_color=discord.Color.blurple(),
    )

    # Le bouton est déclaré sur la ActionRow
    @container.children[-1].button(label="Ouvrir", custom_id="menu:ouvrir", style=discord.ButtonStyle.primary)
    async def ouvrir(self, interaction: discord.Interaction, button: ui.Button):
        await interaction.response.send_message("Menu ouvert.", ephemeral=True)
```

### Via View persistante (timeout=None)

```python
class MonMenuView(ui.LayoutView):
    def __init__(self):
        super().__init__(timeout=None)  # persistante après redémarrage

    @ui.button(label="Ouvrir", custom_id="menu:ouvrir", style=discord.ButtonStyle.primary)
    async def ouvrir(self, interaction: discord.Interaction, button: ui.Button):
        await interaction.response.send_message("Menu ouvert.", ephemeral=True)

@bot.event
async def on_ready():
    bot.add_view(MonMenuView())
```

---

## Convention de nommage des custom_ids

```
namespace:sous-namespace:identifiant

profil:editer:1234567890
boutique:acheter:prod_42
admin:ban:user_987654
notif:toggle:dms
rapport:actualiser
```

Règles :
- Maximum 100 caractères
- Uniques dans le contexte d'une LayoutView persistante
- Pas d'espace ni de caractères spéciaux autres que `:` et `_`
- Lisible sans documentation

---

## Payload API brut

```json
{
  "type": 1,
  "components": [
    { "type": 2, "label": "Confirmer", "style": 3, "custom_id": "confirm" },
    { "type": 2, "label": "Annuler",   "style": 4, "custom_id": "cancel"  }
  ]
}
```

---

## Liens

- [Retour au README](../README.md)
- [Bonnes pratiques](14-bonnes-pratiques.md)
