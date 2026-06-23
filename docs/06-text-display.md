# Text Display (type 10)

## Description

`discord.ui.TextDisplay` est le bloc de texte fondamental de CV2. Il remplace le champ `content` du message et les champs d'embed, avec un support Markdown nettement plus riche. Il peut apparaître directement à la racine d'une `LayoutView`, dans un `Container`, ou dans une `Section`.

---

## Paramètres

| Paramètre | Type | Défaut | Description |
|---|---|---|---|
| `content` | `str` | requis | Texte Markdown (max 4000 chars partagés) |
| `id` | `int \| None` | `None` | Identifiant optionnel |

> [!WARNING]
> La limite de 4000 caractères est **partagée entre tous les TextDisplay** de la même `LayoutView`. Si vous avez 4 TextDisplay de 1200 chars chacun, vous dépassez la limite (4800 chars > 4000).

---

## Markdown supporté

### Titres

```markdown
# Titre H1
## Titre H2
### Titre H3
```

### Formatage inline

```markdown
**gras**
*italique*
__souligné__
~~barré~~
`code inline`
```

### Blocs de code

````markdown
```python
def bonjour():
    print("Hello, CV2")
```

```json
{"cle": "valeur", "nombre": 42}
```

```bash
pip install -U discord.py
```
````

### Listes

```markdown
- Élément 1
- Élément 2
  - Sous-élément

1. Premier
2. Deuxième
3. Troisième
```

### Citation

```markdown
> Ceci est une citation
> sur plusieurs lignes
```

### Liens masqués

```markdown
[Texte du lien](https://discordpy.readthedocs.io)
```

### Mentions et éléments Discord

```markdown
<@USER_ID>          mention utilisateur (pinge !)
<@&ROLE_ID>         mention rôle (pinge !)
<#CHANNEL_ID>       mention salon
<:emoji:ID>         emoji personnalisé
<a:emoji:ID>        emoji animé
```

> [!WARNING]
> Les mentions dans un `TextDisplay` **pinguent réellement** les utilisateurs et rôles, même si le TextDisplay est dans un Container. Utilisez `allowed_mentions=discord.AllowedMentions.none()` pour désactiver les pings.

### Timestamps dynamiques

```markdown
<t:1700000000>      affichage par défaut
<t:1700000000:d>    date courte  → 22/06/2025
<t:1700000000:D>    date longue  → 22 juin 2025
<t:1700000000:t>    heure courte → 14:32
<t:1700000000:T>    heure longue → 14:32:00
<t:1700000000:f>    date + heure → 22 juin 2025 14:32
<t:1700000000:F>    date + heure complète
<t:1700000000:R>    temps relatif → il y a 2 jours
```

---

## Exemples

### Texte simple

```python
from discord import ui

texte = ui.TextDisplay("Bonjour depuis Components V2 !")
```

### Texte Markdown riche

```python
import discord
from discord import ui

class StatutLayout(ui.LayoutView):
    texte = ui.TextDisplay(
        "## Rapport de statut\n\n"
        "**Statut :** En ligne\n"
        "**Uptime :** 99,98 %\n"
        "**Dernière vérification :** <t:1750000000:R>\n\n"
        "> Tous les systèmes sont opérationnels."
    )
```

### Texte avec bloc de code

```python
class CmdLayout(ui.LayoutView):
    texte = ui.TextDisplay(
        "**Commande exécutée :**\n"
        "```bash\n"
        "git push origin main --force\n"
        "```\n"
        "**Résultat :** 3 fichiers modifiés, 150 insertions."
    )
```

### Construction dynamique

```python
def build_log_display(logs: list[dict]) -> ui.TextDisplay:
    lines = ["## Logs récents", ""]
    for log in logs[:10]:
        ts = f"<t:{int(log['timestamp'])}:T>"
        level = f"**[{log['level'].upper()}]**" if log["level"] == "warn" else f"[{log['level'].upper()}]"
        lines.append(f"{ts} {level} {log['message']}")
    return ui.TextDisplay(content="\n".join(lines))
```

### Informations membre avec timestamps

```python
def build_member_text(member: discord.Member) -> ui.TextDisplay:
    joined  = f"<t:{int(member.joined_at.timestamp())}:D>" if member.joined_at else "Inconnu"
    created = f"<t:{int(member.created_at.timestamp())}:D>"
    roles   = " ".join(f"`{r.name}`" for r in member.roles[1:][:5]) or "Aucun"

    return ui.TextDisplay(
        f"## {member.display_name}\n"
        f"**Tag :** {member}\n"
        f"**ID :** `{member.id}`\n"
        f"**Rôles :** {roles}\n"
        f"**Rejoint le :** {joined}\n"
        f"**Compte créé le :** {created}"
    )
```

---

## Gestion de la limite de caractères

```python
def safe_text(content: str, max_len: int = 3900) -> str:
    """Tronque le contenu si nécessaire."""
    if len(content) <= max_len:
        return content
    return content[:max_len] + "\n\n*[Contenu tronqué...]*"

texte = ui.TextDisplay(safe_text(long_text))
```

---

## Échapper le Markdown

```python
import re

def escape_markdown(text: str) -> str:
    """Échappe les caractères spéciaux Markdown."""
    return re.sub(r"([\\`*_{}[\]()#+\-.!|])", r"\\\1", text)

# Protéger une saisie utilisateur
safe_name = escape_markdown(user_input)
texte = ui.TextDisplay(f"Nom saisi : {safe_name}")
```

---

## Payload API brut

```json
{
  "type": 10,
  "content": "## Titre\nCorps du texte avec **markdown**."
}
```

---

## Ce que TextDisplay ne supporte pas

- HTML — uniquement le Markdown Discord
- Citations imbriquées (`>> citation`)
- Tables Markdown
- Images inline — utilisez `ui.MediaGallery` ou `ui.Thumbnail`

---

## Liens

- [Section](05-section.md)
- [Thumbnail](07-thumbnail.md)
- [Retour au README](../README.md)
