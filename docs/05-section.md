# Section (type 9)

## Description

`discord.ui.Section` est le composant de layout le plus puissant de CV2. Il affiche du contenu texte à gauche et un **accessoire optionnel à droite** — soit un `ui.Thumbnail` (image miniature), soit un `ui.Button`. C'est ce qui permet de placer un avatar exactement en face d'un nom, ou un bouton "Acheter" exactement en face d'un produit.

---

## Paramètres

| Paramètre | Type | Défaut | Description |
|---|---|---|---|
| `*items` | `TextDisplay...` | requis | Un ou plusieurs TextDisplay (positionnels) |
| `accessory` | `Thumbnail \| Button \| None` | `None` | Composant affiché à droite |
| `id` | `int \| None` | `None` | Identifiant optionnel |

---

## Accessoires disponibles

### Thumbnail comme accessoire

```python
import discord
from discord import ui

class ProfilLayout(ui.LayoutView):
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

### Button comme accessoire

```python
class BoutiqueLayout(ui.LayoutView):
    container = ui.Container(
        ui.Section(
            ui.TextDisplay("**Plan Pro** — 9,99 EUR/mois\nAccès illimité, 5 utilisateurs"),
            accessory=ui.Button(
                label="S'abonner",
                style=discord.ButtonStyle.success,
                custom_id="plan:pro:subscribe",
            ),
        ),
    )
```

> [!NOTE]
> Le `Button` en accessoire de Section ne peut **pas** avoir `style=link`. Il doit obligatoirement avoir un `custom_id`.

---

## Section avec plusieurs TextDisplay

Une Section peut contenir **plusieurs** `TextDisplay` :

```python
class SectionMulti(ui.LayoutView):
    container = ui.Container(
        ui.Section(
            ui.TextDisplay("**Titre de la section**"),
            ui.TextDisplay("Description sur une ou plusieurs lignes."),
            ui.TextDisplay("*Note supplémentaire en italique.*"),
            accessory=ui.Thumbnail(
                media=discord.UnfurledMediaItem(url="https://exemple.com/icone.png"),
            ),
        ),
    )
```

---

## Section sans accessoire

```python
class ReglementLayout(ui.LayoutView):
    container = ui.Container(
        ui.Section(
            ui.TextDisplay(
                "## Règlement\n\n"
                "1. Respectez tous les membres\n"
                "2. Pas de spam\n"
                "3. Restez dans les bons salons"
            ),
        ),
    )
```

Sans accessoire, une Section fonctionne comme un TextDisplay simple. Préférez directement un `TextDisplay` dans ce cas.

---

## Schéma visuel

```
+------------------------------------------+
|                            |              |
|  [TextDisplay]             | [Thumbnail]  |
|  Texte principal           | ou           |
|  sur plusieurs lignes      | [Button]     |
|                            |              |
+------------------------------------------+
```

L'accessoire est toujours aligné verticalement au centre du bloc texte.

---

## Exemples avancés

### Liste de membres avec avatars (style impératif)

```python
def build_equipe(members_data: list[dict]) -> ui.LayoutView:
    view = ui.LayoutView()
    container = ui.Container(accent_color=discord.Color.blurple())
    container.add_item(ui.TextDisplay("## Équipe de modération"))

    for i, m in enumerate(members_data):
        if i > 0:
            container.add_item(ui.Separator(divider=True, spacing=discord.SeparatorSpacingSize.small))
        section = ui.Section(
            ui.TextDisplay(f"**{m['name']}**\n{m['role']}"),
            accessory=ui.Thumbnail(media=discord.UnfurledMediaItem(url=m["avatar"])),
        )
        container.add_item(section)

    view.add_item(container)
    return view

# Usage
members = [
    {"name": "Alice", "role": "Admin", "avatar": "https://cdn.discordapp.com/.../alice.png"},
    {"name": "Bob",   "role": "Modérateur", "avatar": "https://cdn.discordapp.com/.../bob.png"},
]
await interaction.response.send_message(view=build_equipe(members))
```

### Catalogue de plans avec boutons contextuels

```python
PLANS = [
    {"id": "starter",    "name": "Starter",    "price": "Gratuit",     "style": discord.ButtonStyle.secondary},
    {"id": "pro",        "name": "Pro",        "price": "9,99 EUR/mois","style": discord.ButtonStyle.primary},
    {"id": "enterprise", "name": "Enterprise", "price": "Sur devis",   "style": discord.ButtonStyle.success},
]

def build_boutique() -> ui.LayoutView:
    view = ui.LayoutView()
    container = ui.Container(accent_color=discord.Color.gold())
    container.add_item(ui.TextDisplay("## Nos offres"))
    container.add_item(ui.Separator(divider=True, spacing=discord.SeparatorSpacingSize.large))

    for i, p in enumerate(PLANS):
        if i > 0:
            container.add_item(ui.Separator(divider=True, spacing=discord.SeparatorSpacingSize.small))
        container.add_item(ui.Section(
            ui.TextDisplay(f"**{p['name']}** — {p['price']}"),
            accessory=ui.Button(label="Choisir", style=p["style"], custom_id=f"plan:choisir:{p['id']}"),
        ))

    view.add_item(container)
    return view
```

### Profil membre avec avatar Discord

```python
@bot.tree.command(name="profil")
@discord.app_commands.describe(membre="Membre à afficher")
async def profil(interaction: discord.Interaction, membre: discord.Member = None):
    m = membre or interaction.user

    view = ui.LayoutView()
    container = ui.Container(
        accent_color=m.color if m.color.value else discord.Color.blurple()
    )
    container.add_item(ui.Section(
        ui.TextDisplay(
            f"## {m.display_name}\n"
            f"**ID :** `{m.id}`\n"
            f"**Rejoint le :** <t:{int(m.joined_at.timestamp())}:D>"
        ),
        accessory=ui.Thumbnail(
            media=discord.UnfurledMediaItem(url=m.display_avatar.url),
            description=f"Avatar de {m.display_name}",
        ),
    ))
    view.add_item(container)
    await interaction.response.send_message(view=view)
```

---

## Payload API brut

```json
{
  "type": 9,
  "components": [
    { "type": 10, "content": "**Alice Martin**\nAdministratrice" }
  ],
  "accessory": {
    "type": 11,
    "media": { "url": "https://exemple.com/alice.png" },
    "description": "Avatar d'Alice"
  }
}
```

---

## Notes importantes

- Le `Button` en accessoire ne peut **pas** avoir de `url` (pas de `style=link`)
- L'accessoire est toujours à **droite** — non configurable
- Sans accessoire, préférez directement un `TextDisplay` dans le Container

---

## Liens

- [TextDisplay](06-text-display.md)
- [Thumbnail](07-thumbnail.md)
- [Retour au README](../README.md)
