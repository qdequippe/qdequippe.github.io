---
date: 2022-10-04
linktitle: Des emojis dans vos URLs avec Symfony
type:
- post
- posts
title: Des emojis dans vos URLs avec Symfony
weight: 10
draft: true
tags:
- emoji
- symfony
---

Avec le support des emojis dans le componsant [Intl](https://symfony.com/doc/6.2/components/intl.html#emoji-transliteration) de Symfony il est maintenant possible d'effectuer une "traduction" des emojis dans un alphabet "classique".

Un très bon article explique cela je vous laisse y jeter un oeil avant d'aller plus loin (en anglais 🇬🇧) : https://medium.com/the-sensiolabs-tech-blog/emojis-are-new-symfonys-best-friends-7c084760c056

---

Et si nos URLs devenaient plus sympas ? Et si nous pouvions faire passer des émotions via des URLs tout en pouvant les traiter textuellement ? Et tout ca bien sur automatiquement !

Nous allons imaginer devoir passer des emojis directement dans une URL, par exemple pour donner notre avis sur un produit ou un service.

Par exemple nous aurions `https://mon-site.fr/feedback/☺️` ou encore `https://mon-site.fr/feedback/😡` .

Nous pourrions enregistrer directement les emojis dans notre base de données, mais si je fais ca mon article s'arrête la 🤣

Nous allons donc devoir "traduire" ces emojis dans un alphabet "classique", pour cela nous allons utilisé le ~~EmojiTerminator~~ `EmojiTransliterator` ! c'est parti ...
