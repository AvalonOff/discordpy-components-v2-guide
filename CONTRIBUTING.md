# Contribuer à ce guide

Les contributions sont bienvenues : corrections, exemples supplémentaires, nouvelles sections.

---

## Règles générales

- Ce guide est constitué **uniquement de fichiers Markdown** (`.md`). Aucun fichier Python autonome n'est accepté.
- Le contenu doit être factuel et vérifié contre la documentation officielle Discord : https://discord.com/developers/docs/components/reference
- Pas d'emojis Unicode dans les fichiers Markdown.
- Le guide est optimisé pour l'affichage GitHub (GitHub Flavored Markdown).
- Tous les exemples de code Python doivent fonctionner avec discord.py 2.6+.
- Utilisez les callouts GitHub (`> [!NOTE]`, `> [!WARNING]`, `> [!IMPORTANT]`) pour les avertissements.

---

## Comment contribuer

### Signaler une erreur

Si vous constatez une information incorrecte (limite erronée, classe mal documentée, comportement changé avec une nouvelle version), ouvrez une issue en précisant :
- Le fichier et la section concernés
- Ce qui est incorrect
- La source qui contredit l'information (documentation officielle, release notes discord.py...)

### Proposer une correction mineure

Pour une faute de frappe, un lien cassé, ou une formulation confuse : ouvrez une Pull Request directement avec la correction.

### Proposer une nouvelle section

Avant d'écrire une section complète, ouvrez une issue pour discuter :
- Du sujet proposé
- De la section existante dans laquelle il s'intègre, ou de la nouvelle section à créer
- Des sources utilisées

### Ajouter un exemple

Les exemples supplémentaires sont bienvenus dans `docs/12-exemples.md` ou dans la section du composant concerné. Ils doivent :
- Être testables (code fonctionnel avec discord.py 2.6+)
- Représenter un cas d'usage réel
- Suivre les conventions du guide

---

## Structure des fichiers

```
README.md                  index principal avec badges et TOC
CHANGELOG.md               journal des modifications
CONTRIBUTING.md            ce fichier
LICENSE                    licence MIT
docs/
  01-introduction.md
  02-architecture.md
  03-flag-activation.md
  04-container.md
  05-section.md
  06-text-display.md
  07-thumbnail.md
  08-separator.md
  09-media-gallery.md
  10-file.md
  11-action-rows.md
  12-exemples.md
  13-migration.md
  14-bonnes-pratiques.md
  15-discord-py.md
```

### Numérotation des fichiers

Les fichiers `docs/` sont préfixés par un numéro à deux chiffres (`01-`, `02-`...) pour garantir l'ordre dans les explorateurs de fichiers GitHub. Si vous ajoutez une section, choisissez le prochain numéro disponible.

---

## Style de rédaction

- Titres H2 (`##`) pour les sections principales d'un fichier
- Titres H3 (`###`) pour les sous-sections
- Blocs de code avec indication du langage (` ```python `, ` ```json `, ` ```bash `)
- Tableaux Markdown pour les paramètres (colonnes : Paramètre, Type, Défaut, Description)
- Callouts GitHub pour les avertissements :
  - `> [!NOTE]` — information utile
  - `> [!WARNING]` — attention requise
  - `> [!IMPORTANT]` — règle fondamentale
- Liens vers les autres fichiers en relatif (`[Container](04-container.md)`)
- Section "Liens" en fin de chaque fichier pointant vers les sections connexes et le README

---

## Mise à jour du CHANGELOG

Chaque contribution notable doit être mentionnée dans `CHANGELOG.md`.

---

## Vérification avant soumission

- Les exemples Python fonctionnent avec discord.py 2.6+
- Les liens internes pointent vers des fichiers existants
- Le Markdown s'affiche correctement sur GitHub
- Aucune information non vérifiée sur les limites ou comportements API
- Pas d'emojis Unicode dans le texte

---

## Ressources

- [Documentation officielle Discord Components](https://discord.com/developers/docs/components/reference)
- [Discord Change Log](https://docs.discord.com/developers/change-log)
- [discord.py changelog](https://discordpy.readthedocs.io/en/latest/whats_new.html)
- [discord.py documentation](https://discordpy.readthedocs.io)
- [Visual builder Discord](https://discord.builders)
