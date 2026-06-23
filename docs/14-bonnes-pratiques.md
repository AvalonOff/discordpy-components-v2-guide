# Bonnes pratiques

## Structure des messages

### Hiérarchie recommandée

```
Container
├── TextDisplay (titre H2)
├── Separator (large, avec ligne)
├── Section(s) (contenu principal)
├── MediaGallery (si images)
├── Separator (sans ligne, petit)
└── ActionRow (actions)
```

### Limites à surveiller

- **10 enfants directs** par Container — visez 5-7 pour la lisibilité
- **40 composants** par LayoutView — planifiez avant de construire
- **4000 chars** partagés entre tous les `TextDisplay` de la LayoutView

---

## Nommage des custom_ids

Convention stricte et documentée :

```
namespace:sous-namespace:identifiant

profil:editer:1234567890
rapport:export:pdf
admin:ban:user_987654
notif:toggle:dms
boutique:acheter:prod_42
```

Règles :
- Pas d'espace ni de caractères spéciaux autres que `:` et `_`
- Maximum 100 caractères
- Unique par LayoutView persistante
- Lisible sans documentation

---

## Gestion des interactions

### Toujours répondre dans les 3 secondes

Discord invalide l'interaction si elle n'a pas reçu de réponse dans les 3 secondes. Pour les opérations longues, différez immédiatement :

```python
@bot.event
async def on_interaction(interaction: discord.Interaction):
    if interaction.type != discord.InteractionType.component:
        return

    await interaction.response.defer()  # acquittement immédiat

    # Travail long (requête BDD, API externe...)
    result = await do_long_operation()

    # Répondre via followup
    view = build_result_view(result)
    await interaction.followup.send(view=view)
```

### Réponses éphémères pour les erreurs utilisateur

```python
view = ui.LayoutView()
view.add_item(ui.Container(
    ui.TextDisplay("Permission refusée. Vous n'avez pas les droits nécessaires."),
    accent_color=discord.Color.red(),
))
await interaction.response.send_message(view=view, ephemeral=True)
```

### Éditer le message existant pour les flux interactifs

Quand un bouton déclenche une navigation ou un changement d'état, éditez le message en place :

```python
await interaction.response.edit_message(view=nouveau_layout())
```

---

## Gestion des mentions

Les mentions dans `ui.TextDisplay` pinguent réellement. Utilisez `allowed_mentions` :

```python
# Désactiver tous les pings
await interaction.response.send_message(
    view=view,
    allowed_mentions=discord.AllowedMentions.none(),
)

# Autoriser uniquement les pings utilisateur (pas les rôles)
await interaction.response.send_message(
    view=view,
    allowed_mentions=discord.AllowedMentions(users=True, roles=False, everyone=False),
)
```

---

## Compter les composants

```python
def count_components(view: discord.ui.LayoutView) -> int:
    """Estime le nombre de composants dans une LayoutView."""
    total = 0
    for item in view.children:
        total += 1
        if hasattr(item, "children"):
            for child in item.children:
                total += 1
                if hasattr(child, "children"):
                    total += len(child.children)
    return total

view = build_my_layout()
n = count_components(view)
if n > 38:
    print(f"Attention : {n} composants (limite : 40)")
```

---

## Texte et contenu

### Tronquer proprement

```python
def tronquer(texte: str, max_len: int = 3900, suffixe: str = "\n\n*[Contenu tronqué]*") -> str:
    if len(texte) <= max_len:
        return texte
    return texte[:max_len - len(suffixe)] + suffixe
```

### Échapper les saisies utilisateur

```python
import re

def escape_md(texte: str) -> str:
    return re.sub(r"([\\`*_{}[\]()#+\-.!|])", r"\\\1", texte)

safe_name = escape_md(interaction.user.display_name)
display = ui.TextDisplay(f"Bienvenue, {safe_name} !")
```

### Simuler des tableaux avec monospace

```python
content = (
    "**Statistiques**\n\n"
    "```\n"
    f"{'Membres':<15} {guild.member_count:>6}\n"
    f"{'En ligne':<15} {online_count:>6}\n"
    f"{'Salons texte':<15} {len(guild.text_channels):>6}\n"
    "```"
)
```

---

## Images et médias

### Toujours renseigner la description

Les `description` sur `ui.Thumbnail` et `ui.MediaGalleryItem` servent de texte alternatif pour l'accessibilité. Remplissez-les systématiquement.

### URLs d'avatar Discord

Les URLs d'avatar Discord peuvent expirer. Ne les stockez en base de données qu'avec une TTL courte, ou régénérez-les depuis l'objet `member` au moment de l'envoi.

```python
# Bon : récupéré au moment de l'envoi
url = member.display_avatar.url

# Mauvais : stocké en BDD sans TTL
url = db.get_cached_avatar(member.id)  # peut expirer
```

---

## Construire les layouts hors des callbacks

```python
# A éviter : construction longue dans le callback
@bot.event
async def on_interaction(interaction: discord.Interaction):
    await interaction.response.defer()
    data = await db.fetch_user(interaction.user.id)
    # ...construction longue...

# Mieux : extraire la logique de construction
def build_user_layout(data: dict) -> ui.LayoutView:
    view = ui.LayoutView()
    view.add_item(ui.Container(ui.TextDisplay(f"## {data['name']}")))
    return view

@bot.event
async def on_interaction(interaction: discord.Interaction):
    await interaction.response.defer()
    data = await db.fetch_user(interaction.user.id)
    await interaction.followup.send(view=build_user_layout(data))
```

---

## Patterns à éviter

### Ne pas mélanger des actions sans rapport dans un ActionRow

```python
# A éviter
ui.ActionRow(
    ui.Button(label="Bannir",       custom_id="mod:ban", style=discord.ButtonStyle.danger),
    ui.Button(label="Exporter CSV", custom_id="data:export"),
    ui.Button(label="Documentation", style=discord.ButtonStyle.link, url="https://..."),
)

# Préférer des ActionRows séparées par groupe logique
ui.ActionRow(
    ui.Button(label="Bannir",   custom_id="mod:ban",  style=discord.ButtonStyle.danger),
    ui.Button(label="Expulser", custom_id="mod:kick", style=discord.ButtonStyle.danger),
),
ui.ActionRow(
    ui.Button(label="Exporter CSV",  custom_id="data:export"),
    ui.Button(label="Documentation", style=discord.ButtonStyle.link, url="https://..."),
),
```

### Ne pas injecter du texte utilisateur sans l'échapper

```python
# Dangereux
ui.TextDisplay(f"**Message :** {user_input}")

# Sûr
ui.TextDisplay(f"**Message :** {escape_md(user_input)}")
```

### Ne pas abuser du spoiler Container

`spoiler=True` sur un Container masque **tout**, y compris les boutons. L'utilisateur ne peut pas interagir avant d'avoir révélé le contenu. Réservez ce mode au contenu explicitement sensible.

---

## Liens

- [Exemples pratiques](12-exemples.md)
- [Référence discord.py](15-discord-py.md)
- [Retour au README](../README.md)
