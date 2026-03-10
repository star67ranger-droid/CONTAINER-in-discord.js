# 📦 Guide Complet — ContainerBuilder Discord.js v14 (Components V2)

> Tout ce qu'il faut savoir pour construire des messages riches et structurés avec les containers Components V2.

---

## Table des matières

- [C'est quoi un Container ?](#cest-quoi-un-container-)
- [Prérequis](#prérequis)
- [Structure de base](#structure-de-base)
- [AccentColor — la barre colorée](#accentcolor--la-barre-colorée)
- [TextDisplay — afficher du texte](#textdisplay--afficher-du-texte)
- [Separator — diviser le contenu](#separator--diviser-le-contenu)
- [Section — texte + accessoire](#section--texte--accessoire)
- [Thumbnail — petite image](#thumbnail--petite-image)
- [Button dans une Section](#button-dans-une-section)
- [MediaGallery — galerie d'images](#mediagallery--galerie-dimages)
- [File — pièce jointe](#file--pièce-jointe)
- [ChannelSelectMenu dans un Container](#channelselectmenu-dans-un-container)
- [Exemple complet](#exemple-complet)
- [Règles et limites](#règles-et-limites)
- [Règles d'or](#règles-dor)

---

## C'est quoi un Container ?

Un `ContainerBuilder` est un composant Components V2 qui regroupe plusieurs éléments visuels dans un seul bloc structuré. Il remplace avantageusement les embeds classiques : plus flexible, plus moderne, et capable d'accueillir des médias, des boutons, des menus et du texte dans un seul message cohérent.

La clé pour l'utiliser : ajouter `flags: MessageFlags.IsComponentsV2` dans ta réponse, sinon Discord ignorera les composants V2.

---

## Prérequis

```js
const {
  ContainerBuilder,
  TextDisplayBuilder,
  SeparatorBuilder,
  SeparatorSpacingSize,
  SectionBuilder,
  ThumbnailBuilder,
  ButtonBuilder,
  ButtonStyle,
  MediaGalleryBuilder,
  MediaGalleryItemBuilder,
  FileBuilder,
  AttachmentBuilder,
  ActionRowBuilder,
  ChannelSelectMenuBuilder,
  MessageFlags
} = require('discord.js');
```

---

## Structure de base

```js
const container = new ContainerBuilder();

await interaction.reply({
  flags: MessageFlags.IsComponentsV2,
  components: [container]
});
```

Un container vide ne fait rien de visible. Tu lui ajoutes des composants avec les méthodes `add...Components()`.

---

## AccentColor — la barre colorée

Tu peux ajouter une barre colorée sur le côté gauche du container, comme les embeds. Tu passes une valeur hexadécimale en nombre entier.

```js
const container = new ContainerBuilder()
  .setAccentColor(0x5865F2); // Blurple Discord
```

Pour utiliser une couleur depuis une string hex :

```js
const couleur = '#FF5733';
const container = new ContainerBuilder()
  .setAccentColor(parseInt(couleur.replace('#', ''), 16));
```

---

## TextDisplay — afficher du texte

`TextDisplayBuilder` affiche du texte statique avec support du Markdown complet.

```js
const container = new ContainerBuilder()
  .addTextDisplayComponents(
    new TextDisplayBuilder().setContent('## Titre principal'),
    new TextDisplayBuilder().setContent('Voici une description en **gras** et en *italique*.'),
    new TextDisplayBuilder().setContent('> Une citation mise en avant'),
    new TextDisplayBuilder().setContent('`du code inline` ou un lien : https://discord.com')
  );
```

Tu peux chaîner autant de `TextDisplayBuilder` que tu veux dans un seul appel. Tout le Markdown Discord fonctionne : titres (`##`, `###`), gras, italique, listes, blocs de code, liens, etc.

---

## Separator — diviser le contenu

`SeparatorBuilder` crée un espace visuel ou une ligne de séparation entre les composants.

```js
const separator = new SeparatorBuilder()
  .setDivider(true)                        // affiche une ligne visible
  .setSpacing(SeparatorSpacingSize.Small); // Small ou Large
```

Sans `.setDivider(true)`, le separator crée juste un espace vide. Avec, il affiche une ligne horizontale. Très utile pour séparer les sections visuellement.

```js
const container = new ContainerBuilder()
  .addTextDisplayComponents(new TextDisplayBuilder().setContent('Partie 1'))
  .addSeparatorComponents(
    new SeparatorBuilder().setDivider(true).setSpacing(SeparatorSpacingSize.Small)
  )
  .addTextDisplayComponents(new TextDisplayBuilder().setContent('Partie 2'));
```

---

## Section — texte + accessoire

`SectionBuilder` est le composant le plus puissant du container. Il combine du texte à gauche avec un accessoire à droite (thumbnail ou bouton).

```js
const section = new SectionBuilder()
  .addTextDisplayComponents(
    new TextDisplayBuilder().setContent('## Titre de la section'),
    new TextDisplayBuilder().setContent('Description de la section.')
  );
```

Une section **doit obligatoirement** avoir un accessoire : soit un `ThumbnailBuilder`, soit un `ButtonBuilder`. Une section sans accessoire génère une erreur.

---

## Thumbnail — petite image

`ThumbnailBuilder` s'utilise uniquement comme accessoire d'une `SectionBuilder`. Il affiche une petite image à droite du texte.

```js
const section = new SectionBuilder()
  .addTextDisplayComponents(
    new TextDisplayBuilder().setContent('## Mon profil'),
    new TextDisplayBuilder().setContent('Voici mes informations.')
  )
  .setThumbnailAccessory(
    new ThumbnailBuilder({ media: { url: 'https://example.com/avatar.png' } })
  );

const container = new ContainerBuilder()
  .addSectionComponents(section);
```

L'URL doit être une image publiquement accessible. Tu peux utiliser les URLs d'avatars Discord directement.

---

## Button dans une Section

Un `ButtonBuilder` peut aussi être l'accessoire d'une section. Il apparaît alors à droite du texte.

```js
const section = new SectionBuilder()
  .addTextDisplayComponents(
    new TextDisplayBuilder().setContent('📖 **Documentation**'),
    new TextDisplayBuilder().setContent('Consulte la doc officielle Discord.js.')
  )
  .setButtonAccessory(
    new ButtonBuilder()
      .setLabel('Ouvrir')
      .setURL('https://discord.js.org')
      .setStyle(ButtonStyle.Link)
  );
```

Pour les boutons avec interaction (non-Link), utilise un `customId` à la place de l'URL :

```js
.setButtonAccessory(
  new ButtonBuilder()
    .setLabel('Confirmer')
    .setCustomId('confirm_action')
    .setStyle(ButtonStyle.Primary)
)
```

---

## MediaGallery — galerie d'images

`MediaGalleryBuilder` affiche une ou plusieurs images en carrousel dans le container.

```js
const gallery = new MediaGalleryBuilder()
  .addItems(
    new MediaGalleryItemBuilder().setURL('https://example.com/image1.png'),
    new MediaGalleryItemBuilder().setURL('https://example.com/image2.png'),
    new MediaGalleryItemBuilder().setURL('https://example.com/image3.png')
  );

const container = new ContainerBuilder()
  .addMediaGalleryComponents(gallery);
```

Tu peux ajouter une description sur chaque item :

```js
new MediaGalleryItemBuilder()
  .setURL('https://example.com/image1.png')
  .setDescription('Légende de l\'image')
```

---

## File — pièce jointe

`FileBuilder` permet de référencer un fichier attaché au message directement depuis le container.

```js
const attachment = new AttachmentBuilder('./mon-fichier.json').setName('mon-fichier.json');
const fileComponent = new FileBuilder().setURL('attachment://mon-fichier.json');

const container = new ContainerBuilder()
  .addTextDisplayComponents(
    new TextDisplayBuilder().setContent('📎 Voici le fichier joint :')
  )
  .addFileComponents(fileComponent);

await interaction.reply({
  flags: MessageFlags.IsComponentsV2,
  components: [container],
  files: [attachment]        // ← ne pas oublier d'attacher le fichier ici aussi
});
```

Le nom dans `attachment://` doit correspondre exactement au `.setName()` de l'`AttachmentBuilder`.

---

## ChannelSelectMenu dans un Container

Les select menus (`ActionRowBuilder`) ne peuvent pas être mis directement dans un container. Ils doivent être passés en dehors, en tant que composant de niveau supérieur dans `components: []`.

```js
const container = new ContainerBuilder()
  .addTextDisplayComponents(
    new TextDisplayBuilder().setContent('Choisissez un salon ci-dessous :')
  );

const selectRow = new ActionRowBuilder().addComponents(
  new ChannelSelectMenuBuilder()
    .setCustomId('channel_select')
    .setPlaceholder('Sélectionner un salon...')
);

await interaction.reply({
  flags: MessageFlags.IsComponentsV2,
  components: [container, selectRow]  // ← le selectRow est en dehors du container
});
```

---

## Exemple complet

Voici un exemple concret d'une commande `/profil` qui utilise toutes les fonctionnalités du container :

```js
const {
  SlashCommandBuilder,
  MessageFlags,
  ContainerBuilder,
  TextDisplayBuilder,
  SeparatorBuilder,
  SeparatorSpacingSize,
  SectionBuilder,
  ThumbnailBuilder,
  ButtonBuilder,
  ButtonStyle,
  MediaGalleryBuilder,
  MediaGalleryItemBuilder
} = require('discord.js');

module.exports = {
  data: new SlashCommandBuilder()
    .setName('profil')
    .setDescription("Afficher le profil d'un utilisateur")
    .addUserOption(opt => opt
      .setName('user')
      .setDescription('Utilisateur cible (toi par défaut)')
    ),

  async execute(interaction) {
    const user      = interaction.options.getUser('user') ?? interaction.user;
    const member    = interaction.guild.members.cache.get(user.id);
    const avatar    = user.displayAvatarURL({ extension: 'png', size: 256 });
    const joinDate  = member?.joinedAt?.toLocaleDateString('fr-FR') ?? 'Inconnu';
    const createDate = user.createdAt.toLocaleDateString('fr-FR');
    const roles     = member?.roles.cache
      .filter(r => r.id !== interaction.guild.id)
      .map(r => `<@&${r.id}>`)
      .join(' ') || 'Aucun rôle';

    const container = new ContainerBuilder()
      .setAccentColor(0x5865F2)

      // Entête avec avatar
      .addSectionComponents(
        new SectionBuilder()
          .addTextDisplayComponents(
            new TextDisplayBuilder().setContent(`## 👤 ${user.username}`),
            new TextDisplayBuilder().setContent(`ID : \`${user.id}\``)
          )
          .setThumbnailAccessory(
            new ThumbnailBuilder({ media: { url: avatar } })
          )
      )

      .addSeparatorComponents(
        new SeparatorBuilder().setDivider(true).setSpacing(SeparatorSpacingSize.Small)
      )

      // Informations
      .addTextDisplayComponents(
        new TextDisplayBuilder().setContent('### 📋 Informations'),
        new TextDisplayBuilder().setContent(`📅 **Compte créé le** : ${createDate}`),
        new TextDisplayBuilder().setContent(`📥 **A rejoint le** : ${joinDate}`),
        new TextDisplayBuilder().setContent(`🏷️ **Rôles** : ${roles}`)
      )

      .addSeparatorComponents(
        new SeparatorBuilder().setDivider(true).setSpacing(SeparatorSpacingSize.Small)
      )

      // Galerie
      .addMediaGalleryComponents(
        new MediaGalleryBuilder().addItems(
          new MediaGalleryItemBuilder().setURL(avatar).setDescription('Avatar')
        )
      )

      .addSeparatorComponents(
        new SeparatorBuilder().setDivider(true).setSpacing(SeparatorSpacingSize.Small)
      )

      // Lien vers l'avatar
      .addSectionComponents(
        new SectionBuilder()
          .addTextDisplayComponents(
            new TextDisplayBuilder().setContent('🔗 **Voir l\'avatar en plein écran**')
          )
          .setButtonAccessory(
            new ButtonBuilder()
              .setLabel('Ouvrir')
              .setURL(avatar)
              .setStyle(ButtonStyle.Link)
          )
      );

    await interaction.reply({
      flags: MessageFlags.IsComponentsV2,
      components: [container]
    });
  }
};
```

---

## Règles et limites

**Ce qui va DANS le container via `add...Components()` :**
- `TextDisplayBuilder` → `addTextDisplayComponents()`
- `SeparatorBuilder` → `addSeparatorComponents()`
- `SectionBuilder` → `addSectionComponents()`
- `MediaGalleryBuilder` → `addMediaGalleryComponents()`
- `FileBuilder` → `addFileComponents()`

**Ce qui va EN DEHORS du container dans `components: []` :**
- `ActionRowBuilder` (select menus, boutons classiques)

**Ce qui s'utilise uniquement comme accessoire d'une Section :**
- `ThumbnailBuilder` → `.setThumbnailAccessory()`
- `ButtonBuilder` (dans une section) → `.setButtonAccessory()`

**Limites importantes :**
- Une `SectionBuilder` doit toujours avoir un accessoire (thumbnail ou bouton), sinon erreur.
- Le nom dans `attachment://nom` doit correspondre exactement au `.setName()` de l'attachment.
- Tu dois toujours passer `flags: MessageFlags.IsComponentsV2` dans la réponse, sinon les composants V2 ne s'affichent pas.
- Les URLs des médias doivent être publiquement accessibles.

---

## Règles d'or

- 📦 Groupe tout ce qui est lié dans un seul container pour un rendu cohérent.
- 🎨 Utilise `setAccentColor()` pour donner une identité visuelle à chaque type de message.
- ✂️ Sépare les blocs logiques avec des `SeparatorBuilder` pour aérer la lecture.
- 🖼️ Utilise les sections avec thumbnail pour les profils, cartes, résultats de recherche.
- 🔗 Utilise les sections avec bouton pour les liens d'action directs.
- 📋 Ne mets jamais un `ActionRowBuilder` dans un container — il va en dehors.
- ⚠️ Toujours ajouter `MessageFlags.IsComponentsV2` dans la réponse sinon rien ne s'affiche.

---

## Ressources

- [Discord Developer Docs — Components Overview](https://discord.com/developers/docs/components/overview)
- [Component Types Reference](https://discord.com/developers/docs/components/reference)
- [Discord.js Docs](https://discord.js.org/)


# By claude AI
