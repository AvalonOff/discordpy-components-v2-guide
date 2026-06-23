# Separator (type 14)

## Description

`discord.ui.Separator` crée une séparation visuelle entre les blocs de contenu dans un Container. Il peut afficher une ligne horizontale visible et/ou ajouter de l'espace vertical entre les éléments. C'est l'équivalent d'une balise HTML `<hr>` enrichie.

---

## Paramètres

| Paramètre | Type | Défaut | Description |
|---|---|---|---|
| `divider` | `bool` | `True` | Affiche une ligne horizontale visible |
| `spacing` | `SeparatorSpacingSize` | `small` | Taille de l'espace autour |
| `id` | `int \| None` | `None` | Identifiant optionnel |

---

## Valeurs de spacing

| Constante discord.py | Valeur API | Effet |
|---|---|---|
| `discord.SeparatorSpacingSize.small` | `1` | Espace réduit — éléments proches |
| `discord.SeparatorSpacingSize.large` | `2` | Espace généreux — sections majeures |

---

## Exemples

### Style déclaratif

```python
import discord
from discord import ui

class RapportLayout(ui.LayoutView):

    class RapportContainer(ui.Container):
        titre    = ui.TextDisplay("## Rapport mensuel")
        sep_maj  = ui.Separator(divider=True, spacing=discord.SeparatorSpacingSize.large)
        section1 = ui.TextDisplay("**Section A** : contenu de la première partie")
        sep_min  = ui.Separator(divider=True, spacing=discord.SeparatorSpacingSize.small)
        section2 = ui.TextDisplay("**Section B** : contenu de la deuxième partie")
        gap      = ui.Separator(divider=False, spacing=discord.SeparatorSpacingSize.small)
        actions  = ui.ActionRow(
            ui.Button(label="Exporter", custom_id="export", style=discord.ButtonStyle.primary)
        )

    container = RapportContainer(accent_color=discord.Color.blurple())
```

### Style impératif dans une boucle

```python
def build_liste(items: list[str]) -> ui.LayoutView:
    view = ui.LayoutView()
    container = ui.Container(accent_color=discord.Color.blurple())
    container.add_item(ui.TextDisplay("## Ma liste"))

    for i, item in enumerate(items):
        container.add_item(ui.TextDisplay(item))
        if i < len(items) - 1:
            container.add_item(ui.Separator(divider=True, spacing=discord.SeparatorSpacingSize.small))

    view.add_item(container)
    return view
```

---

## Guide d'utilisation

| Cas | Paramètres recommandés |
|---|---|
| Séparation entre deux grandes sections | `divider=True, spacing=large` |
| Séparation entre éléments d'une liste | `divider=True, spacing=small` |
| Espace vide sans ligne | `divider=False, spacing=small` |
| Espace vide large sans ligne | `divider=False, spacing=large` |

---

## Guide visuel

```
+------------------------------------------+
|  Titre                                   |
|                                          |
|  ════════════════════════════════════    |  divider=True,  spacing=large
|                                          |
|  Section A                               |
|  ————————————————————————————————————    |  divider=True,  spacing=small
|  Section B                               |
|                                          |  divider=False, spacing=small
|  [Bouton]                                |
+------------------------------------------+
```

---

## Payload API brut

```json
{ "type": 14, "divider": true,  "spacing": 1 }
{ "type": 14, "divider": true,  "spacing": 2 }
{ "type": 14, "divider": false, "spacing": 1 }
```

---

## Notes importantes

- Un Separator compte comme **1 enfant** du Container (dans la limite de 10)
- `divider=False` + `spacing=small` = espace invisible, sans ligne
- Un Separator ne peut **pas** être placé dans une `Section`
- Les deux valeurs de `spacing` (`small` et `large`) sont les seules disponibles

---

## Liens

- [Container](04-container.md)
- [Retour au README](../README.md)
