# Changelog

Format basé sur [Keep a Changelog](https://keepachangelog.com/fr/1.1.0/).

---

## [2.0.0] — 2026-06-22

### Refonte majeure — discord.py 2.6+

Ce guide a été entièrement revu pour correspondre à discord.py 2.6 (août 2025) et 2.7.1 (mars 2026).

#### Changements d'API documentés

- Point d'entrée : `discord.ui.LayoutView` remplace l'envoi via `components=[...], flags=...`
- Toutes les classes déplacées sous le namespace `discord.ui.*` :
  - `discord.ui.Container` (anciennement `discord.Container`)
  - `discord.ui.TextDisplay` (anciennement `discord.TextDisplay`)
  - `discord.ui.Section` (anciennement `discord.Section`)
  - `discord.ui.Thumbnail` (anciennement `discord.Thumbnail`)
  - `discord.ui.MediaGallery` + `discord.ui.MediaGalleryItem`
  - `discord.ui.File` (anciennement `discord.FileComponent`)
  - `discord.ui.Separator` (anciennement `discord.Separator`)
- Le flag `IS_COMPONENTS_V2` est posé automatiquement par `LayoutView`
- `stickers` et `polls` également incompatibles avec CV2 (ajouté)
- Les mentions dans `TextDisplay` pinguent — `allowed_mentions` nécessaire (ajouté)
- Deux styles d'écriture documentés : déclaratif (attributs de classe) et impératif (`add_item`)
- Version discord.py corrigée : 2.6+ requis (non 2.4)

#### Ajouts

- Viewer web intégré dans Replit (`/api/docs`)
- `docs/02-architecture.md` enrichi avec le détail de `LayoutView`
- `docs/14-bonnes-pratiques.md` : section sur les mentions et `allowed_mentions`
- Callouts GitHub (`> [!NOTE]`, `> [!WARNING]`, `> [!IMPORTANT]`) dans les fichiers appropriés
- Badges shields.io dans le README

---

## [1.0.0] — 2025-06-22

### Initial

- 15 fichiers de documentation couvrant tous les composants CV2
- README avec table des matières et démarrage rapide
- Exemples pratiques, migration V1→V2, bonnes pratiques
