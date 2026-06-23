# Introduction aux Components V2

## Historique

Avant mars 2025, les bots Discord avaient deux façons de structurer un message :

- **Texte brut** (`content`) : une chaîne Markdown limitée, toujours en haut du message
- **Embeds** : blocs colorés avec titre, champs, image, footer — mais non interactifs
- **Components V1** : boutons et menus, uniquement en bas du message via des `ActionRow`

Ces trois éléments étaient indépendants et non composables. Impossible de placer un bouton en face d'un produit, une image en face d'un titre, ou du texte riche à plusieurs endroits dans le même message.

**Discord Components V2** (CV2), publié en mars 2025, résout ce problème. Il unifie tout en un système de blocs composables : texte, images, fichiers et boutons peuvent être placés presque n'importe où, dans presque n'importe quel ordre.

**discord.py 2.6** (août 2025) introduit le support natif de CV2 avec `discord.ui.LayoutView` comme point d'entrée. **discord.py 2.7.1** (mars 2026) est la version stable la plus récente.

---

## Différences V1 vs CV2

| Critère | Components V1 | Components V2 |
|---|---|---|
| Texte principal | `content=` (une seule chaine) | `ui.TextDisplay` (plusieurs blocs Markdown) |
| Embeds | Supportés | Incompatibles avec le flag CV2 |
| Position des boutons | Toujours en bas du message | N'importe où dans le layout |
| Images | Embeds `image` / `thumbnail` | `ui.Thumbnail`, `ui.MediaGallery`, `ui.File` |
| Pièces jointes | S'affichent automatiquement | Doivent passer par `ui.File` |
| Markdown | Limité (pas de H1/H2/H3) | Complet (H1 à H3, listes, code, etc.) |
| Structure | Plates | Hiérarchique (Container > Section > ...) |
| Entry point | `view=ui.View()` | `view=ui.LayoutView()` |
| Flag requis | Non | `IS_COMPONENTS_V2` (automatique avec LayoutView) |
| Stickers | Supportés | Incompatibles |
| Polls | Supportés | Incompatibles |

---

## Ce que CV2 permet

- Afficher un avatar en face du nom d'un membre (Section + Thumbnail)
- Afficher un bouton "Acheter" en face d'un produit (Section + Button accessoire)
- Construire un tableau de bord paginé avec navigation par boutons
- Afficher une galerie de screenshots dans un rapport automatisé
- Envoyer un rapport PDF avec contexte visuel complet
- Structurer un menu multi-niveaux avec des écrans editables

---

## Ce que CV2 ne remplace pas

CV2 n'est pas adapté à tous les cas d'usage. Les situations où les composants V1 classiques restent préférables :

- Messages simples sans mise en page particulière (une ligne de texte)
- Embeds riches avec `author`, `footer`, `timestamp` automatique — CV2 n'a pas ces champs
- Messages avec stickers ou polls
- Bots ciblant discord.py < 2.6 (qui n'ont pas le support natif)

> [!NOTE]
> Les Components V1 ne disparaissent pas. `ui.View` continue de fonctionner normalement. Vous pouvez utiliser V1 et CV2 dans le même bot, sur des messages différents.

---

## Architecture d'un message CV2

Un message CV2 est composé d'une `LayoutView` qui contient des **composants de premier niveau** :

```
LayoutView
├── TextDisplay      — texte Markdown
├── Container        — boite avec bord coloré (contient d'autres composants)
│   ├── TextDisplay
│   ├── Section      — texte + accessoire côte à côte
│   ├── Separator    — ligne ou espace
│   ├── MediaGallery — galerie d'images
│   ├── File         — pièce jointe dans le flux
│   └── ActionRow    — boutons / selects
└── ActionRow        — aussi utilisable directement dans la LayoutView
```

---

## Installation

```bash
# Version stable recommandée (juin 2026)
pip install -U "discord.py>=2.6"

# Dernière version stable (2.7.1, mars 2026)
pip install -U discord.py

# Vérifier la version installée
python -c "import discord; print(discord.__version__)"
```

---

## Premier message CV2

```python
import discord
from discord import ui

bot = discord.Bot(intents=discord.Intents.default())

class AccueilLayout(ui.LayoutView):
    """Layout d'accueil avec titre, texte et bouton."""
    container = ui.Container(
        ui.TextDisplay("## Bienvenue !\nCe message utilise Components V2."),
        ui.Separator(divider=True, spacing=discord.SeparatorSpacingSize.small),
        ui.ActionRow(
            ui.Button(label="Docs Discord", style=discord.ButtonStyle.link,
                      url="https://discord.com/developers/docs/components/reference")
        ),
        accent_color=discord.Color.blurple(),
    )

@bot.tree.command(name="accueil")
async def accueil(interaction: discord.Interaction):
    await interaction.response.send_message(view=AccueilLayout())

bot.run("TOKEN")
```

---

## Liens

- [Architecture](02-architecture.md)
- [Flag IS_COMPONENTS_V2](03-flag-activation.md)
- [Retour au README](../README.md)
