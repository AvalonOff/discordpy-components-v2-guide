# Architecture des Components V2

## Le point d'entrée : LayoutView

`discord.ui.LayoutView` est la classe de base de tout message CV2 en discord.py 2.6+. Elle remplace `discord.ui.View` pour les messages qui utilisent des composants V2.

```python
import discord
from discord import ui

class MonLayout(ui.LayoutView):
    # Les composants de premier niveau deviennent des attributs de classe
    container = ui.Container(...)
    texte     = ui.TextDisplay("...")
    row       = ui.ActionRow(...)

# Envoi — le flag IS_COMPONENTS_V2 est posé automatiquement
await interaction.response.send_message(view=MonLayout())
```

Différences avec `ui.View` :

| Critère | `ui.View` (V1) | `ui.LayoutView` (CV2) |
|---|---|---|
| Namespace | `discord.ui.View` | `discord.ui.LayoutView` |
| Mise en page | Automatique (ActionRows auto-générées) | Manuelle |
| Composants | Boutons, selects uniquement | Tous les composants CV2 |
| `content` + `embeds` | Compatibles | Incompatibles |
| Flag CV2 | Non | Posé automatiquement |

---

## Les trois couches

CV2 classe ses composants en trois couches selon leur rôle :

### Couche Layout

Les composants qui structurent l'espace :

| Composant | Type API | Rôle |
|---|---|---|
| `ui.Container` | `17` | Boite avec bord coloré, contient les autres composants |
| `ui.Section` | `9` | Bloc texte à gauche + accessoire à droite |
| `ui.ActionRow` | `1` | Ligne de boutons ou d'un menu |
| `ui.Separator` | `14` | Ligne horizontale ou espace vertical |

### Couche Content

Les composants qui affichent du contenu :

| Composant | Type API | Rôle |
|---|---|---|
| `ui.TextDisplay` | `10` | Bloc de texte Markdown |
| `ui.Thumbnail` | `11` | Image miniature (uniquement en accessoire de Section) |
| `ui.MediaGallery` | `12` | Galerie de 1 à 10 images/vidéos |
| `ui.File` | `13` | Pièce jointe dans le flux du message |

### Couche Interactive

Les composants qui répondent aux clics :

| Composant | Type API | Rôle |
|---|---|---|
| `ui.Button` | `2` | Bouton cliquable |
| `ui.StringSelect` | `3` | Menu déroulant avec options textuelles |
| `ui.UserSelect` | `5` | Sélection d'utilisateurs |
| `ui.RoleSelect` | `6` | Sélection de rôles |
| `ui.MentionableSelect` | `7` | Sélection utilisateurs ou rôles |
| `ui.ChannelSelect` | `8` | Sélection de salons |

---

## Hiérarchie et imbrication

```
LayoutView (racine)
│
├── ui.TextDisplay          — niveau racine OK
├── ui.Container            — niveau racine OK
│   ├── ui.TextDisplay      — dans Container OK
│   ├── ui.Section          — dans Container OK
│   │   ├── ui.TextDisplay  — dans Section OK (plusieurs possibles)
│   │   └── accessory:
│   │       ├── ui.Thumbnail — accessoire de Section uniquement
│   │       └── ui.Button    — accessoire de Section (pas de style=link)
│   ├── ui.Separator        — dans Container OK
│   ├── ui.MediaGallery     — dans Container OK
│   ├── ui.File             — dans Container OK
│   └── ui.ActionRow        — dans Container OK
│       └── ui.Button(s) ou SelectMenu
│
└── ui.ActionRow            — niveau racine OK
```

Règles d'imbrication :
- Un `Container` ne peut **pas** contenir un autre `Container`
- Un `Thumbnail` ne peut **pas** être placé directement dans un Container — uniquement en `Section.accessory`
- Un `Button` en accessoire de Section ne peut **pas** avoir `style=link` (pas d'URL)
- `ui.TextInput` est réservé aux modals — pas utilisable dans un message CV2

---

## Limite de 40 composants

La limite de 40 composants s'applique à l'ensemble de la `LayoutView`, imbrication incluse. Chaque élément compte pour 1 :

```
LayoutView
├── Container                   → 1
│   ├── TextDisplay             → 1  (total: 2)
│   ├── Separator               → 1  (total: 3)
│   ├── Section                 → 1  (total: 4)
│   │   └── TextDisplay         → 1  (total: 5)
│   ├── MediaGallery            → 1  (total: 6)  [pas 1 par image — la galerie est 1]
│   └── ActionRow               → 1  (total: 7)
│       ├── Button              → 1  (total: 8)
│       └── Button              → 1  (total: 9)
└── ActionRow                   → 1  (total: 10)
    └── Button                  → 1  (total: 11)
```

Les items d'une `MediaGallery` (les `MediaGalleryItem`) ne comptent pas comme composants séparés.

---

## Limite de 4000 caractères

La limite de 4000 caractères s'applique à l'ensemble des `TextDisplay` de la même `LayoutView`, pas par composant. Si vous avez 5 `TextDisplay` contenant chacun 1000 chars, vous avez atteint 5000 chars et l'API rejettera le message.

---

## Deux styles d'écriture

### Style déclaratif

Les composants sont définis comme attributs de classe. Lisible pour les layouts statiques.

```python
class RapportLayout(ui.LayoutView):
    en_tete  = ui.TextDisplay("## Rapport mensuel")
    sep      = ui.Separator(divider=True, spacing=discord.SeparatorSpacingSize.large)
    contenu  = ui.TextDisplay("Contenu du rapport...")
    actions  = ui.ActionRow(
        ui.Button(label="Exporter", custom_id="export", style=discord.ButtonStyle.primary)
    )
```

### Style impératif

Les composants sont créés et ajoutés dynamiquement. Indispensable pour les layouts conditionnels ou générés en boucle.

```python
def build_rapport(items: list[str]) -> ui.LayoutView:
    view = ui.LayoutView()
    container = ui.Container(accent_color=discord.Color.blurple())
    container.add_item(ui.TextDisplay("## Rapport dynamique"))
    container.add_item(ui.Separator())
    for item in items:
        container.add_item(ui.TextDisplay(f"- {item}"))
    view.add_item(container)
    return view
```

---

## Liens

- [Flag IS_COMPONENTS_V2](03-flag-activation.md)
- [Container](04-container.md)
- [Retour au README](../README.md)
