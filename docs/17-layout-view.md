# LayoutView — Le conteneur racine

`discord.ui.LayoutView` est le point d'entrée de **tout** message CV2. C'est lui qui pose automatiquement le flag `IS_COMPONENTS_V2` et qui contient tous les composants du message.

---

## Paramètres

| Paramètre | Type | Défaut | Description |
|---|---|---|---|
| `timeout` | `float \| None` | `180.0` | Durée en secondes avant désactivation. `None` = persistante |

---

## Les deux styles d'écriture

### Style déclaratif — attributs de classe

Idéal pour les layouts fixes, connus à l'avance.

```python
import discord
from discord import ui

class MonLayout(ui.LayoutView):
    # Chaque attribut de classe est un composant ajouté dans l'ordre
    titre = ui.TextDisplay("## Titre du message")
    sep   = ui.Separator(divider=True, spacing=discord.SeparatorSpacingSize.large)
    row   = ui.ActionRow(
        ui.Button(label="OK", style=discord.ButtonStyle.primary, custom_id="ok"),
    )

await interaction.response.send_message(view=MonLayout())
```

### Style impératif — `add_item()`

Indispensable pour les layouts construits dynamiquement (boucles, conditions, données BDD).

```python
view = ui.LayoutView()

container = ui.Container(accent_color=discord.Color.blurple())
container.add_item(ui.TextDisplay("## Titre dynamique"))

for ligne in lignes_depuis_bdd:
    container.add_item(ui.TextDisplay(f"- {ligne}"))

container.add_item(ui.Separator(divider=False, spacing=discord.SeparatorSpacingSize.small))
container.add_item(ui.ActionRow(
    ui.Button(label="Actualiser", custom_id="refresh", style=discord.ButtonStyle.primary)
))

view.add_item(container)
await interaction.response.send_message(view=view)
```

> [!NOTE]
> Les deux styles sont entièrement équivalents. Vous pouvez même les mélanger : définir la structure de base en déclaratif et ajouter des éléments dynamiques en impératif via `add_item()`.

---

## Timeout et désactivation automatique

Par défaut, une LayoutView expire après **180 secondes**. À l'expiration, `on_timeout()` est appelé. Les boutons ne sont **pas** automatiquement désactivés — c'est à vous de le faire.

### Personnaliser le timeout

```python
class MonLayout(ui.LayoutView):
    def __init__(self):
        super().__init__(timeout=60)  # 60 secondes

    text = ui.TextDisplay("Cette vue expire dans 60 secondes.")
    row  = ui.ActionRow(
        ui.Button(label="Cliquer", custom_id="btn", style=discord.ButtonStyle.primary)
    )

    async def on_timeout(self):
        # Le message est déjà envoyé — éditer via self.message si disponible
        if self.message:
            expired = ui.LayoutView()
            expired.add_item(ui.Container(
                ui.TextDisplay("Session expirée. Relancez la commande."),
                accent_color=discord.Color.dark_gray(),
            ))
            try:
                await self.message.edit(view=expired)
            except discord.NotFound:
                pass  # le message a été supprimé
```

### Désactiver le timeout

```python
class MonLayout(ui.LayoutView):
    def __init__(self):
        super().__init__(timeout=None)  # jamais expirée
```

---

## Views persistantes — résistent au redémarrage du bot

Une view avec `timeout=None` et des `custom_id` fixes peut être **réenregistrée** au démarrage pour continuer à recevoir des interactions sur des messages anciens.

```python
class MenuPrincipalLayout(ui.LayoutView):
    def __init__(self):
        super().__init__(timeout=None)

    # Les custom_ids doivent être FIXES (pas de variables d'instance dans le custom_id)
    container = ui.Container(
        ui.TextDisplay("## Menu permanent"),
        ui.ActionRow(
            ui.Button(label="Option A", custom_id="menu:a", style=discord.ButtonStyle.primary),
            ui.Button(label="Option B", custom_id="menu:b", style=discord.ButtonStyle.secondary),
        ),
        accent_color=discord.Color.blurple(),
    )

@bot.event
async def on_ready():
    # Réenregistrer la view au démarrage pour les messages existants
    bot.add_view(MenuPrincipalLayout())
    print(f"Bot connecté : {bot.user}")
```

> [!WARNING]
> Pour qu'une view persistante fonctionne après redémarrage, tous ses `custom_id` doivent être **des chaînes fixes**, pas des valeurs construites avec `f"{user_id}"` ou similaire.
> Si vous avez besoin d'un identifiant dynamique, gérez-le dans `on_interaction` via une base de données.

---

## Accéder aux composants enfants

```python
class MonLayout(ui.LayoutView):
    container = ui.Container(
        ui.TextDisplay("## Bonjour"),
        ui.ActionRow(
            ui.Button(label="Cliquer", custom_id="btn", style=discord.ButtonStyle.primary)
        ),
    )

view = MonLayout()

# Itérer sur les enfants directs de la LayoutView
for item in view.children:
    print(type(item).__name__)  # Container

# Itérer sur les enfants d'un Container
for child in view.children[0].children:
    print(type(child).__name__)  # TextDisplay, ActionRow
```

---

## Compter les composants avant envoi

Discord refuse toute LayoutView dépassant **40 composants** (tous niveaux imbriqués confondus).

```python
def compter_composants(view: ui.LayoutView) -> int:
    total = 0
    for item in view.children:
        total += 1
        if hasattr(item, "children"):
            for child in item.children:
                total += 1
                if hasattr(child, "children"):
                    total += len(child.children)
    return total

view = build_mon_layout()
n = compter_composants(view)
assert n <= 40, f"Trop de composants : {n}/40"
```

---

## `stop()` — désactiver manuellement

```python
async def on_interaction(interaction: discord.Interaction):
    if interaction.data.get("custom_id") == "confirmer":
        self.stop()  # arrête d'écouter les interactions futures
        termine = ui.LayoutView()
        termine.add_item(ui.Container(ui.TextDisplay("Action confirmée.")))
        await interaction.response.edit_message(view=termine)
```

---

## Envoyer une LayoutView — toutes les méthodes

| Contexte | Méthode |
|---|---|
| Répondre à une slash command | `await interaction.response.send_message(view=...)` |
| Réponse éphémère | `await interaction.response.send_message(view=..., ephemeral=True)` |
| Éditer le message en place | `await interaction.response.edit_message(view=...)` |
| Après un `defer()` | `await interaction.followup.send(view=...)` |
| Commande prefix | `await ctx.send(view=...)` |
| Envoyer dans un salon | `await channel.send(view=...)` |
| Envoyer en DM | `await user.send(view=...)` |
| Éditer un message existant | `await message.edit(view=...)` |

---

## Résumé visuel

```
ui.LayoutView(timeout=180 | None)
│
├── Pose automatiquement IS_COMPONENTS_V2
├── Contient 1 à N composants de premier niveau
│   ├── ui.Container        (type 17)
│   ├── ui.TextDisplay      (type 10) — texte direct hors container
│   └── ui.ActionRow        (type 1)  — boutons directs hors container
│
├── timeout=None + add_view() → persistante après redémarrage
├── on_timeout()            → hook de désactivation
└── stop()                  → désactivation manuelle
```

---

## Liens

- [Architecture CV2](02-architecture.md)
- [Container](04-container.md)
- [Action Rows et boutons](11-action-rows.md)
- [Retour au README](../README.md)
