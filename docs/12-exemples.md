# Exemples pratiques

Cas d'usage réels combinant plusieurs composants CV2.

---

## 1. Carte de profil utilisateur

```python
import discord
from discord import ui

@bot.tree.command(name="profil")
@discord.app_commands.describe(membre="Membre à afficher")
async def profil(interaction: discord.Interaction, membre: discord.Member = None):
    m = membre or interaction.user

    roles = [r for r in reversed(m.roles) if r.name != "@everyone" and not r.managed]
    roles_text = " ".join(r.mention for r in roles[:5]) or "Aucun"
    joined  = f"<t:{int(m.joined_at.timestamp())}:D>" if m.joined_at else "Inconnu"
    created = f"<t:{int(m.created_at.timestamp())}:D>"

    view = ui.LayoutView()
    container = ui.Container(accent_color=m.color if m.color.value else discord.Color.blurple())
    container.add_item(ui.Section(
        ui.TextDisplay(
            f"## {m.display_name}\n"
            f"**Rôles :** {roles_text}\n"
            f"**Rejoint le :** {joined}\n"
            f"**Compte créé le :** {created}\n"
            f"**ID :** `{m.id}`"
        ),
        accessory=ui.Thumbnail(
            media=discord.UnfurledMediaItem(url=m.display_avatar.url),
            description=f"Avatar de {m.display_name}",
        ),
    ))

    if interaction.user.guild_permissions.manage_messages and m != interaction.user:
        container.add_item(ui.Separator(divider=False, spacing=discord.SeparatorSpacingSize.small))
        container.add_item(ui.ActionRow(
            ui.Button(label="Avertir",  custom_id=f"profil:warn:{m.id}",  style=discord.ButtonStyle.secondary),
            ui.Button(label="Expulser", custom_id=f"profil:kick:{m.id}",  style=discord.ButtonStyle.danger),
            ui.Button(label="Bannir",   custom_id=f"profil:ban:{m.id}",   style=discord.ButtonStyle.danger),
        ))

    view.add_item(container)
    await interaction.response.send_message(
        view=view,
        allowed_mentions=discord.AllowedMentions.none(),
    )
```

---

## 2. Catalogue produits avec boutons contextuels

```python
PRODUCTS = [
    {"id": "starter",    "name": "Starter",    "price": "Gratuit",        "style": discord.ButtonStyle.secondary},
    {"id": "pro",        "name": "Pro",        "price": "9,99 EUR/mois",  "style": discord.ButtonStyle.primary},
    {"id": "enterprise", "name": "Enterprise", "price": "Sur devis",      "style": discord.ButtonStyle.success},
]

@bot.tree.command(name="boutique")
async def boutique(interaction: discord.Interaction):
    view = ui.LayoutView()
    container = ui.Container(accent_color=discord.Color.gold())
    container.add_item(ui.TextDisplay("## Nos offres"))
    container.add_item(ui.Separator(divider=True, spacing=discord.SeparatorSpacingSize.large))

    for i, p in enumerate(PRODUCTS):
        if i > 0:
            container.add_item(ui.Separator(divider=True, spacing=discord.SeparatorSpacingSize.small))
        container.add_item(ui.Section(
            ui.TextDisplay(f"**{p['name']}** — {p['price']}"),
            accessory=ui.Button(label="Choisir", style=p["style"], custom_id=f"plan:choisir:{p['id']}"),
        ))

    view.add_item(container)
    await interaction.response.send_message(view=view)
```

---

## 3. Tableau de bord serveur actualisable

```python
import time

def build_dashboard(guild: discord.Guild) -> ui.LayoutView:
    now    = int(time.time())
    online = sum(1 for m in guild.members if not m.bot and m.status != discord.Status.offline)
    icon   = guild.icon.url if guild.icon else "https://cdn.discordapp.com/embed/avatars/0.png"

    view = ui.LayoutView()
    container = ui.Container(accent_color=discord.Color.blurple())
    container.add_item(ui.Section(
        ui.TextDisplay(
            f"## {guild.name}\n"
            f"**Membres :** {guild.member_count}\n"
            f"**En ligne :** {online}\n"
            f"**Salons :** {len(guild.text_channels)} texte, {len(guild.voice_channels)} voix\n"
            f"**Actualisé :** <t:{now}:T>"
        ),
        accessory=ui.Thumbnail(media=discord.UnfurledMediaItem(url=icon)),
    ))
    container.add_item(ui.Separator(divider=False, spacing=discord.SeparatorSpacingSize.small))
    container.add_item(ui.ActionRow(
        ui.Button(label="Actualiser",       custom_id=f"dash:refresh:{guild.id}", style=discord.ButtonStyle.primary),
        ui.Button(label="Membres en ligne", custom_id=f"dash:online:{guild.id}",  style=discord.ButtonStyle.secondary),
    ))
    view.add_item(container)
    return view

@bot.tree.command(name="dashboard")
async def dashboard(interaction: discord.Interaction):
    await interaction.response.send_message(view=build_dashboard(interaction.guild))
```

---

## 4. Menu multi-étapes (navigation entre écrans)

```python
def screen_accueil() -> ui.LayoutView:
    view = ui.LayoutView()
    container = ui.Container(
        ui.TextDisplay("## Menu principal\nQue souhaitez-vous faire ?"),
        ui.Separator(divider=True, spacing=discord.SeparatorSpacingSize.small),
        ui.ActionRow(
            ui.Button(label="Compte",    style=discord.ButtonStyle.primary,   custom_id="menu:compte"),
            ui.Button(label="Paramètres", style=discord.ButtonStyle.secondary, custom_id="menu:params"),
            ui.Button(label="Fermer",    style=discord.ButtonStyle.danger,     custom_id="menu:fermer"),
        ),
        accent_color=discord.Color.blurple(),
    )
    view.add_item(container)
    return view

def screen_compte() -> ui.LayoutView:
    view = ui.LayoutView()
    container = ui.Container(
        ui.TextDisplay("## Gestion du compte"),
        ui.Section(
            ui.TextDisplay("**Modifier le profil**\nChanger votre nom et votre bio."),
            accessory=ui.Button(label="Modifier", custom_id="menu:compte:edit", style=discord.ButtonStyle.primary),
        ),
        ui.Separator(divider=False, spacing=discord.SeparatorSpacingSize.small),
        ui.ActionRow(
            ui.Button(label="Retour", style=discord.ButtonStyle.secondary, custom_id="menu:accueil"),
        ),
        accent_color=discord.Color.blurple(),
    )
    view.add_item(container)
    return view

SCREENS = {
    "menu:accueil": screen_accueil,
    "menu:compte":  screen_compte,
}

@bot.event
async def on_interaction(interaction: discord.Interaction):
    if interaction.type != discord.InteractionType.component:
        return
    cid = interaction.data.get("custom_id", "")
    if cid in SCREENS:
        await interaction.response.edit_message(view=SCREENS[cid]())
    elif cid == "menu:fermer":
        ferme = ui.LayoutView()
        ferme.add_item(ui.Container(ui.TextDisplay("Menu fermé.")))
        await interaction.response.edit_message(view=ferme)
```

---

## 5. Rapport d'erreur formaté

```python
import traceback, time

async def log_error(bot: discord.Client, channel_id: int, error: Exception, ctx=None):
    tb = "".join(traceback.format_exception(type(error), error, error.__traceback__))
    if len(tb) > 1500:
        tb = "...[tronqué]\n" + tb[-1500:]

    now = int(time.time())

    view = ui.LayoutView()
    container = ui.Container(accent_color=discord.Color.red())
    container.add_item(ui.TextDisplay(
        f"## Erreur : `{type(error).__name__}`\n"
        + (f"**Commande :** `{ctx.command}`\n**Auteur :** {ctx.author}\n" if ctx else "")
        + f"**Heure :** <t:{now}:F>\n\n"
        f"**Message :** {str(error) or 'Aucun'}\n\n"
        f"**Traceback :**\n```python\n{tb}\n```"
    ))
    container.add_item(ui.Separator(divider=False, spacing=discord.SeparatorSpacingSize.small))

    row = ui.ActionRow(
        ui.Button(label="Marquer résolu", custom_id=f"err:ok:{now}",   style=discord.ButtonStyle.success),
        ui.Button(label="Ignorer",        custom_id=f"err:skip:{now}", style=discord.ButtonStyle.secondary),
    )
    if ctx and hasattr(ctx, "message"):
        row.add_item(ui.Button(label="Aller au message", style=discord.ButtonStyle.link, url=ctx.message.jump_url))
    container.add_item(row)
    view.add_item(container)

    channel = bot.get_channel(channel_id)
    if channel:
        await channel.send(view=view, allowed_mentions=discord.AllowedMentions.none())
```

---

## 6. Galerie depuis des pièces jointes

```python
@bot.tree.command(name="galerie")
async def galerie(interaction: discord.Interaction):
    if not interaction.message or not interaction.message.attachments:
        await interaction.response.send_message("Joignez des images à votre commande.", ephemeral=True)
        return

    images = [
        a for a in interaction.message.attachments
        if a.content_type and a.content_type.startswith("image/")
    ][:10]

    if not images:
        await interaction.response.send_message("Aucune image valide.", ephemeral=True)
        return

    view = ui.LayoutView()
    container = ui.Container(
        ui.TextDisplay(f"## Galerie\n{len(images)} image(s) partagée(s) par {interaction.user.mention}"),
        ui.MediaGallery(
            *[ui.MediaGalleryItem(media=discord.UnfurledMediaItem(url=a.url), description=a.filename) for a in images]
        ),
    )
    view.add_item(container)
    await interaction.response.send_message(view=view, allowed_mentions=discord.AllowedMentions.none())
```

---

## 7. Fichier de log joint dans un message

```python
import io

@bot.tree.command(name="log_fichier")
async def log_fichier(interaction: discord.Interaction):
    log_content = "2026-06-22 14:00:00 [INFO] Bot démarré\n2026-06-22 14:01:00 [WARN] Mémoire élevée\n"
    file = discord.File(io.BytesIO(log_content.encode()), filename="bot.log")

    view = ui.LayoutView()
    container = ui.Container(
        ui.TextDisplay("## Logs du bot\nFichier joint ci-dessous."),
        ui.File(file=discord.UnfurledMediaItem(url="attachment://bot.log")),
        accent_color=discord.Color.dark_gray(),
    )
    view.add_item(container)
    await interaction.response.send_message(view=view, files=[file])
```

---

## Liens

- [Migration V1 → CV2](13-migration.md)
- [Bonnes pratiques](14-bonnes-pratiques.md)
- [Retour au README](../README.md)
