---
date: 2022-10-04
linktitle: Des emojis dans vos URLs avec Symfony
type:
- post
- posts
title: Des emojis dans vos URLs avec Symfony
tags:
- emoji
- symfony
- param converter
---

![Example image](/images/domingo-alvarez-e-Cs3y8Mn6-Gk-unsplash.jpg)

Et si nos URLs devenaient plus sympas ? Et si nous pouvions ajouter des emojis dans nos URLs tout en pouvant les 
stocker sous forme de texte ? Et tout ça bien sûr automatiquement ?

Nous allons imaginer devoir passer des emojis directement dans une URL pour donner notre avis sur un 
produit ou un service.

Par exemple nous aurions `https://mon-site.fr/feedback/👍` ou encore `https://mon-site.fr/feedback/😡`.

C'est parti 🚀

## Emoji + URL = c'est possible

Et oui c'est possible d'avoir des emojis dans une URL, commençons donc par créer notre controller et notre route :

```php
<?php

namespace App\Controller;

...

class EmojiController extends AbstractController
{
    #[Route('/feedback/{emoji}', name: 'app_emoji')]
    public function index(string $emoji): Response
}
```

Si on dump ici ce qui se trouve dans la variable `$emoji` nous allons obtenir l'emoji tel quel, par exemple
avec l'url `https://mon-site.fr/feedback/👍`, le dump affichera `👍`

Comment maintenant "convertir" ce `👍` en texte ?

## Emoji + Symfony = 💙

Grâce au super travail de [Grégoire Pineau](https://twitter.com/lyrixx) (membre de la core team de Symfony) nous allons
pouvoir "convertir"/"traduire" notre emoji vers un alphabet plus "classique", tout ça grâce au service
[`EmojiTransliterator`](https://symfony.com/doc/6.2/components/intl.html#emoji-transliteration) (je vous laisse chercher
dans le code source de Symfony comment ça marche 😉).

Reprenons donc notre controller et utilisons ce nouveau service, il faut tout d'abord lui définir un "id" en 1er argument,
"id" qui est en quelque sorte un catalogue des traductions/conversions disponibles. Dans notre cas, ce sera le catalogue
francais "fr" (mais ça marche aussi en anglais avec "en" et même avec "github" 🤩).

Appelons ensuite la méthode `transliterate` et voyons ce que ca donne.

```php
<?php

namespace App\Controller;

...
use Symfony\Component\Intl\Transliterator\EmojiTransliterator;

class EmojiController extends AbstractController
{
    #[Route('/feedback/{emoji}', name: 'app_emoji')]
    public function index(string $emoji): Response
    {
        $transliterator = EmojiTransliterator::create('fr');
        
        $translation = $transliterator->transliterate($emoji);
        
        ...
    }
}
```

Et 🎉 `$translation` ici va contenir (avec l'url `https://mon-site.fr/feedback/👍`) : `thumbs up`

Tout fonctionne bien, maintenant comment obtenir automatiquement cette "traduction" en argument de notre fonction ?

## Param Converter à la rescousse !

Connaissez-vous les [Param Converter](https://symfony.com/bundles/SensioFrameworkExtraBundle/current/annotations/converters.html) ?
Ces services permettent de convertir des paramètres de la requete directement
en objet (ou autre) et de les obtenir en argument de votre fonction par exemple.

Nous pouvons même créer notre propre Param Converter et c'est ce que nous allons faire. Créons un 
`EmojiConverter` qui va nous permettre d'automatiser la "traduction" de notre emoji passé dans l'url.

Tout est bien expliqué dans la doc de Symfony donc allons y, je vous explique tout juste après étape par étape :

```php
<?php

namespace App;

use Sensio\Bundle\FrameworkExtraBundle\Configuration\ParamConverter;
use Sensio\Bundle\FrameworkExtraBundle\Request\ParamConverter\ParamConverterInterface;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\Intl\Transliterator\EmojiTransliterator;

final class EmojiConverter implements ParamConverterInterface
{
    public function apply(Request $request, ParamConverter $configuration): bool
    {
        $transliterator = EmojiTransliterator::create('en');

        $param = $request->attributes->get($configuration->getName());

        $translation = $transliterator->transliterate($param);
        
        $request->attributes->set($configuration->getName(), $translation);

        return true;
    }

    public function supports(ParamConverter $configuration): bool
    {
        return $configuration->getConverter() === 'emoji';
    }
}
```

La 1re étape va être d'écrire la méthode `supports` qui permet de dire à Symfony s'il peut appliquer notre Param Converter
`EmojiConverter` au paramètre présent dans l'url. Ici simplement nous lui disons que le nom du converter appelé doit être "emoji"
(Nous verrons par la suite comment appeler notre service).

2ème étape, nous allons reprendre ce que nous avions fait dans notre controller et l'appliquer dans la méthode `apply`, méthode
qui va être appelée si `supports` renvoi `true`, afin d'effectuer la transformation.

Notre emoji se trouve dans les `attributes` de la requete, nous allons donc récupérer ici sa valeur 

```php
$request->attributes->get($configuration->getName())
```

`$configuration->getName()` nous donne ici le nom de l'attribut présent dans l'url que l'on souhaite convertir. 
Dans notre cas, c'est `emoji` (nous verrons par la suite comment le définir).

Enfin nous effectuons la "traduction" et nous allons stocker ce résultat dans le même attribut par soucis de simplicité.

```php
$translation = $transliterator->transliterate($param);
$request->attributes->set($configuration->getName(), $translation);
```

## Utiliser EmojiConverter

C'est bien beau tout ça mais pour l'instant rien ne se passe, comment utiliser notre Param Converter personnalisé ?

Tout d'abord nous allons faire un poil de config, il va falloir configurer ce nouveau Param Converter pour pouvoir
l'utiliser comme nous le souhaitons.

```yaml
# services.yml

App\EmojiConverter:
    tags:
        - { name: request.param_converter, converter: emoji }
```

Maintenant que notre service est configuré nous allons pouvoir l'utiliser.

```php
<?php

namespace App\Controller;

...

class EmojiController extends AbstractController
{
    #[Route('/feedback/{emoji}', name: 'app_emoji')]
    #[ParamConverter('emoji', converter: 'emoji')]
    public function index(string $emoji): Response
}
```

En utilisant l'attribut (ou l'annotation pour PHP < 8) `ParamConverter` et en lui définissant le nom de l'attribut (de la requête)
présent dans l'url (ici `emoji`) et le converter souhaité la "traduction" se fera automatiquement 🎉.

### Bonus

Si vous souhaitez convertir plusieurs paramètres vous pouvez le faire comme ceci :

```php
<?php

namespace App\Controller;

...

class EmojiController extends AbstractController
{
    #[Route('/feedback/{emoji1}/{emoji2}', name: 'app_emoji')]
    #[ParamConverter('emoji1', converter: 'emoji')]
    #[ParamConverter('emoji2', converter: 'emoji')]
    public function index(string $emoji1, string $emoji2): Response
}
```

---

## N'oubliez pas Jobbsy !

Vous êtes à la recherche d'un job de développeur/développeuse Symfony ?

Alors jetez un œil du côté de [Jobbsy](https://jobbsy.dev/) le job board dédié à Symfony.
