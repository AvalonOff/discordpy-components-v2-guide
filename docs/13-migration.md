# Migration V1 vers CV2

## Pourquoi migrer ?

Les Components V1 restent supportés. La migration vers CV2 n'est justifiée que si vous avez besoin de :
- Placer des boutons en face du contenu qu'ils contrôlent (pas en bas)
- Afficher des images dans le flux du message (pas en embed)
- Utiliser du Markdown riche (H1/H2/H3, listes, blocs de code) dans le texte
- Construire des layouts visuellement structurés

> [!NOTE]
> Les Components V1 (`ui.View`) ne disparaissent pas. Vous pouvez utiliser V1 et CV2 dans le même bot sur des messages différents. La migration n'est pas tout-ou-rien.

---

## Ce qui change et ce qui ne change pas

| Élément | V1 | CV2 |
|---|---|---|
| Boutons | `ui.View` + `@ui.button` | `ui.ActionRow` dans `ui.LayoutView` |
| Texte | `content=` | `ui.TextDisplay` dans `ui.LayoutView` |
| Embeds | Supportés | Incompatibles |
| Images | Embed `image/thumbnail` | `ui.Thumbnail`, `ui.MediaGallery` |
| Pièces jointes | Automatiques | Via `ui.File` |
| Position des boutons | Bas du message | Partout dans le layout |
| Entry point | `view=ui.View()` | `view=ui.LayoutView()` |
| Flag CV2 | Non | Automatique avec LayoutView |
| Callbacks | Identiques | Identiques |
| Views persistantes | Supportées | Supportées |
| discord.py requis | 2.0+ | 2.6+ |

---

## Équivalences directes

### Texte simple

```python
# V1
await ctx.send("Bonjour tout le monde !")

# CV2
class BonjourLayout(ui.LayoutView):
    text = ui.TextDisplay("Bonjour tout le monde !")

await ctx.send(view=BonjourLayout())
```

### Embed basique

```python
# V1
embed = discord.Embed(title="Rapport", description="Contenu du rapport", color=discord.Color.blurple())
embed.add_field(name="Champ A", value="Valeur A")
await ctx.send(embed=embed)

# CV2
class RapportLayout(ui.LayoutView):
    container = ui.Container(
        ui.TextDisplay(
            "## Rapport\n"
            "Contenu du rapport\n\n"
            "**Champ A**\nValeur A"
        ),
        accent_color=discord.Color.blurple(),
    )

await ctx.send(view=RapportLayout())
```

### Embed avec image et thumbnail

```python
# V1
embed = discord.Embed(title="Article", description="Description")
embed.set_image(url="https://exemple.com/grande.png")
embed.set_thumbnail(url="https://exemple.com/miniature.png")
await ctx.send(embed=embed)

# CV2
class ArticleLayout(ui.LayoutView):
    container = ui.Container(
        ui.Section(
            ui.TextDisplay("## Article\nDescription"),
            accessory=ui.Thumbnail(
                media=discord.UnfurledMediaItem(url="https://exemple.com/miniature.png")
            ),
        ),
        ui.MediaGallery(
            ui.MediaGalleryItem(media=discord.UnfurledMediaItem(url="https://exemple.com/grande.png"))
        ),
    )

await ctx.send(view=ArticleLayout())
```

### Message avec boutons

```python
# V1
class ActionView(ui.View):
    @ui.button(label="Confirmer", style=discord.ButtonStyle.success)
    async def confirmer(self, interaction, button):
        await interaction.response.send_message("Confirmé !", ephemeral=True)

await ctx.send("Confirmez-vous l'action ?", view=ActionView())

# CV2
class ConfirmLayout(ui.LayoutView):
    container = ui.Container(
        ui.TextDisplay("Confirmez-vous l'action ?"),
        ui.ActionRow(
            ui.Button(label="Confirmer", style=discord.ButtonStyle.success, custom_id="confirmer"),
            ui.Button(label="Annuler",   style=discord.ButtonStyle.danger,   custom_id="annuler"),
        ),
    )

@bot.event
async def on_interaction(interaction: discord.Interaction):
    if interaction.type != discord.InteractionType.component:
        return
    cid = interaction.data.get("custom_id", "")
    if cid == "confirmer":
        await interaction.response.send_message("Confirmé !", ephemeral=True)
    elif cid == "annuler":
        await interaction.response.send_message("Annulé.", ephemeral=True)

await ctx.send(view=ConfirmLayout())
```

### Embed avec champs et boutons

```python
# V1
embed = discord.Embed(title="Commande", color=discord.Color.blurple())
embed.add_field(name="Produit", value="Widget Pro", inline=True)
embed.add_field(name="Prix",    value="19,99 EUR",  inline=True)
embed.add_field(name="Stock",   value="12 unités",  inline=False)
await ctx.send(embed=embed, view=AcheterView())

# CV2
class CommandeLayout(ui.LayoutView):
    container = ui.Container(
        ui.TextDisplay(
            "## Commande\n\n"
            "**Produit :** Widget Pro\n"
            "**Prix :** 19,99 EUR\n"
            "**Stock :** 12 unités"
        ),
        ui.Separator(divider=False, spacing=discord.SeparatorSpacingSize.small),
        ui.ActionRow(
            ui.Button(label="Acheter", custom_id="acheter", style=discord.ButtonStyle.success),
            ui.Button(label="Voir le produit", style=discord.ButtonStyle.link,
                      url="https://boutique.exemple.com/widget-pro"),
        ),
        accent_color=discord.Color.blurple(),
    )

await ctx.send(view=CommandeLayout())
```

---

## Migration progressive

Si vous avez une large base de code, migrez progressivement commande par commande :

```python
def make_layout(content: str, color: discord.Color = discord.Color.blurple()) -> ui.LayoutView:
    """Wrappeur pour remplacer rapidement ctx.send('text', embed=...)."""
    view = ui.LayoutView()
    view.add_item(ui.Container(ui.TextDisplay(content), accent_color=color))
    return view

# Remplacement direct
await ctx.send(view=make_layout("## Succès\nOpération effectuée.", discord.Color.green()))
```

---

## Migration des Views persistantes

Les Views persistantes (`timeout=None`) fonctionnent de la même façon en CV2. Les `custom_id` restent inchangés.

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

## Checklist de migration

- Remplacer `content=` par `ui.TextDisplay` dans un `ui.Container`
- Remplacer `embeds=[embed]` par des composants CV2 équivalents
- Passer de `view=ui.View()` à `view=ui.LayoutView()`
- Migrer les boutons de `@ui.button` vers `ui.ActionRow` + `ui.Button`
- Remplacer les images embed par `ui.Thumbnail` ou `ui.MediaGallery`
- Adapter les pièces jointes avec `ui.File`
- Tester tous les chemins interactifs (boutons, selects, edits)
- Vérifier que les views persistantes sont enregistrées au démarrage
- Vérifier que les mentions sont gérées avec `allowed_mentions`

---

## Liens

- [Flag IS_COMPONENTS_V2](03-flag-activation.md)
- [Exemples pratiques](12-exemples.md)
- [Bonnes pratiques](14-bonnes-pratiques.md)
- [Retour au README](../README.md)
