<!-- IGNORE THE HTML BLOCK BELOW, THE INTERESTING PART IS AFTER IT -->

<h1 align="center">Popper. js</h1>

<p align="center">
    <strong>Bibliothèque utilisée pour positionner les pop-out dans les applications Web.</strong>
</p>

<p align="center">
    <a href="https://travis-ci.org/FezVrasta/popper.js/branches" target="_blank"><img src="https://travis-ci.org/FezVrasta/popper.js.svg?branch=master" alt="Build Status"/></a>
    <img src="http://img.badgesize.io/https://unpkg.com/popper.js/dist/popper.min.js?compression=gzip" alt="Stable Release Size"/>
    <a href="https://www.bithound.io/github/FezVrasta/popper.js"><img src="https://www.bithound.io/github/FezVrasta/popper.js/badges/score.svg" alt="bitHound Overall Score"></a>
    <a href="https://codeclimate.com/github/FezVrasta/popper.js/coverage"><img src="https://codeclimate.com/github/FezVrasta/popper.js/badges/coverage.svg" alt="Istanbul Code Coverage"/></a>
    <a href="https://gitter.im/FezVrasta/popper.js" target="_blank"><img src="https://img.shields.io/gitter/room/nwjs/nw.js.svg" alt="Get support or discuss"/></a>
    <br />
    <a href="https://saucelabs.com/u/popperjs" target="_blank"><img src="https://badges.herokuapp.com/browsers?labels=none&googlechrome=latest&firefox=latest&microsoftedge=latest&iexplore=11,10&safari=latest&iphone=latest" alt="SauceLabs Reports"/></a>
</p>

<img src="https://raw.githubusercontent.com/FezVrasta/popper.js/master/popperjs.png" align="right" width=250 />

<!-- 🚨 HEY! HERE BEGINS THE INTERESTING STUFF 🚨 -->

## <a name="wut-poppers"></a>Wut? Pop-up?

Un Popper est un élément à l'écran qui «sort» du flux naturel de votre application.  
Des info-bulles, des popovers et des listes déroulantes sont des exemples courants de pop-up.


## <a name="so-yet-another-tooltip-library"></a>Alors, une autre bibliothèque d'info-bulles?

Eh bien, fondamentalement, **non**.  
Popper. js étant un **moteur de positionnement**, son objectif est de calculer la position d'un élément de sorte qu'il soit possible de le positionner à proximité d'un élément de référence donné.  

Le moteur est entièrement modulaire et la plupart de ses fonctionnalités sont implémentées en tant que **modificateurs** (semblables aux middlewares ou aux plugins).  
L'intégralité de la base de code est écrite dans ES2015 et ses fonctionnalités sont testées automatiquement sur les navigateurs réels grâce à [SauceLabs](https://saucelabs.com/) et [TravisCI](https://travis-ci.org/).

Popper. js n'a aucune dépendance. Pas jQuery, no LoDash, Nothing.  
Elle est utilisée par les grandes entreprises comme [Twitter dans bootstrap v4](https://getbootstrap.com/), [Microsoft dans webclipper](https://github.com/OneNoteDev/WebClipper) et [Atlassian dans AtlasKit](https://aui-cdn.atlassian.com/atlaskit/registry/).

### <a name="popperjs"></a>Popper. js

Il s'agit du moteur, la bibliothèque qui calcule et, éventuellement, applique les styles aux pop-up.

Voici quelques-uns des points clés:

- Placez les éléments en les conservant dans leur contexte DOM d'origine (ne fonctionne pas avec votre DOM!);
- Permet d'exporter les informations calculées afin de les intégrer à des bibliothèques de révisions et d'autres bibliothèques d'affichage;
- Prend en charge les éléments Shadow DOM;
- Entièrement personnalisable grâce à la structure basée sur les modificateurs;

Visitez notre [page de projet](https://fezvrasta.github.io/popper.js) pour voir un grand nombre d'exemples de ce que vous pouvez faire avec Popper. js!

Trouvez [la documentation ici](/docs/_includes/popper-documentation.md).


### <a name="tooltipjs"></a>ToolTip. js

Étant donné que de nombreux utilisateurs ont simplement besoin d'une méthode simple pour intégrer des info-bulles puissantes dans leurs projets, nous avons créé **ToolTip. js**.  
Il s'agit d'une petite bibliothèque qui facilite la création automatique d'info-bulles à l'aide d'un moteur Popper. js.  
Son API est quasiment identique au système de démarrage de démarrage célèbre, de cette manière, il est facile de l'intégrer dans vos projets.  
Les info-bulles générées par ToolTip. js sont accessibles grâce `aria` aux balises.

Trouvez [la documentation ici](/docs/_includes/tooltip-documentation.md).


## <a name="installation"></a>Installation
Popper. js est disponible sur les gestionnaires de package et CDN suivants:

| Source |                                                                                  |
|:-------|:---------------------------------------------------------------------------------|
| npm    | `npm install popper.js --save`                                                   |
| ténacité   | `yarn add popper.js`                                                             |
| NuGet  | `PM> Install-Package popper.js`                                                  |
| Bower  | `bower install popper.js --save`                     |
| unpkg  | [`https://unpkg.com/popper.js`](https://unpkg.com/popper.js)                     |
| cdnjs  | [`https://cdnjs.com/libraries/popper.js`](https://cdnjs.com/libraries/popper.js) |

ToolTip. js également:

| Source |                                                                                  |
|:-------|:---------------------------------------------------------------------------------|
| npm    | `npm install tooltip.js --save`                                                  |
| ténacité   | `yarn add tooltip.js`                                                            |
| Bower * | `bower install tooltip.js=https://unpkg.com/tooltip.js --save`                   |
| unpkg  | [`https://unpkg.com/tooltip.js`](https://unpkg.com/tooltip.js)                   |
| cdnjs  | [`https://cdnjs.com/libraries/popper.js`](https://cdnjs.com/libraries/popper.js) |

\*: Bower n'est pas officiellement pris en charge, il peut être utilisé pour installer ToolTip. js uniquement en creux le CDN unpkg.com. Cette méthode a la limitation de ne pas pouvoir définir une version spécifique de la bibliothèque. Bower et Popper. js suggèrent d'utiliser des NPM ou des fils pour vos projets.  
Pour plus d'informations, [Lisez le problème connexe](https://github.com/FezVrasta/popper.js/issues/390).

### <a name="dist-targets"></a>Cibles de dist

Popper. js est actuellement fourni avec 3 cibles à l'esprit: UMD, ESM et ESNext.

- UMD-définition de module universel: AMD, RequireJS et Globals;
- Modules ESM-ES: pour webpacking/ROLLUP ou pour les navigateurs prenant en charge les spécifications;
- ESNext: disponible dans `dist/`, peut être utilisé avec WebPack et `babel-preset-env`;

Veillez à utiliser la bonne pour répondre à vos besoins. Si vous souhaitez l'importer avec une `<script>` balise, utilisez UMD.

## <a name="usage"></a>Utilisation

À partir d'un nœud DOM Popper existant, demandez à Popper. js de le placer près de son bouton.

```js
var reference = document.querySelector('.my-button');
var popper = document.querySelector('.my-popper');
var anotherPopper = new Popper(
    reference,
    popper,
    {
        // popper options here
    }
);
```

### <a name="callbacks"></a>Rappels

Popper. js prend en charge deux types de rappels `onCreate` , le rappel est appelé après l'initialisation de l'Popper. Le `onUpdate` premier est appelé sur toute mise à jour ultérieure.

```js
const reference = document.querySelector('.my-button');
const popper = document.querySelector('.my-popper');
new Popper(reference, popper, {
    onCreate: (data) => {
        // data is an object containing all the informations computed
        // by Popper.js and used to style the popper and its arrow
        // The complete description is available in Popper.js documentation
    },
    onUpdate: (data) => {
        // same as `onCreate` but called on subsequent updates
    }
});
```

### <a name="writing-your-own-modifiers"></a>Écriture de vos propres modificateurs

Popper. js est basé sur une architecture «de plug-in», la plupart de ses fonctionnalités sont entièrement encapsulées «modificateurs».  
Un modificateur est une fonction qui est appelée chaque fois que Popper. js doit calculer la position de l'Popper. Pour cette raison, les modificateurs doivent être très performants pour éviter les goulets d'étranglement.  

Pour en savoir plus sur la création d'un modificateur, [consultez la documentation sur les modificateurs](docs/_includes/popper-documentation.md#modifiers--object)


### <a name="react-vuejs-angular-angularjs-emberjs-etc-integration"></a>Intégration de REACT, vue. js, angulaire, AngularJS, Ember. js (etc...)

L'intégration de bibliothèques tierces dans les bibliothèques REACT ou d'autres bibliothèques peut être une douleur, car elles modifient généralement le DOM et rendent les bibliothèques fou.  
Popper. js limite toutes les modifications apportées `applyStyle` au modèle DOM à l'intérieur du modificateur, vous pouvez simplement le désactiver et appliquer manuellement les coordonnées Popper à l'aide de votre bibliothèque de choix.  

Pour obtenir la liste complète des bibliothèques qui vous permettent d'utiliser Popper. js dans des infrastructures existantes, [](/MENTIONS.md) visitez la page mentions.

Vous pouvez également remplacer votre propre par votre propre `applyStyles` et intégrer Popper. js par vous-même!

```js
function applyReactStyle(data) {
    // export data in your framework and use its content to apply the style to your popper
};

const reference = document.querySelector('.my-button');
const popper = document.querySelector('.my-popper');
new Popper(reference, popper, {
    modifiers: {
        applyStyle: { enabled: false },
        applyReactStyle: {
            enabled: true,
            fn: applyReactStyle,
            order: 800,
        },
    },
});

```

### <a name="migration-from-popperjs-v0"></a>Migration à partir de Popper. js v0

Étant donné que l'API a changé, nous avons préparé des instructions de migration pour faciliter la mise à niveau vers Popper. js v1.  

https://github.com/FezVrasta/popper.js/issues/62

N'hésitez pas à commenter le problème si vous avez des questions.

### <a name="performances"></a>Performance

Popper. js est très performant. Il faut généralement 0,5 ms pour calculer la position d'une Popper (sur un iMac avec 3,5 G Intel Core i5).  
Cela signifie qu'elle n'entraîne aucune [Jank](https://www.chromium.org/developers/how-tos/trace-event-profiling-tool/anatomy-of-jank), ce qui se traduit par une expérience utilisateur sans complication.

## <a name="notes"></a>Notes

### <a name="libraries-using-popperjs"></a>Bibliothèques à l'aide de Popper. js

L'objectif de Popper. js est de fournir un moteur de positionnement stable et puissant prêt à être utilisé dans les bibliothèques tierces.  

Visitez la [](/MENTIONS.md) page des mentions pour obtenir une liste mise à jour des projets.


### <a name="credits"></a>Crédits
Je souhaite remercier des amis et des projets pour le travail qu'il a effectué:

- [@AndreaScn](https://github.com/AndreaScn) de son travail sur la page GitHub et le test manuel qu'il a fait lors du développement;
- [@vampolo](https://github.com/vampolo) de l'idée d'origine et du nom de la bibliothèque;
- [Sysdig](https://github.com/Draios) pour toutes les choses géniales que j'ai apprises au cours de ces années, qui m'a permis d'écrire cette bibliothèque;
- [Attache. js](http://github.hubspot.com/tether/) pour m'avoir inspiré lors de l'écriture d'une bibliothèque de positionnement prête pour le monde réel;
- [Les contributeurs](https://github.com/FezVrasta/popper.js/graphs/contributors) de leurs requêtes d'extraction et de bogues très appréciées;
- **vous** pour l'étoile que vous allez donner à ce projet et, si vous le souhaitez, donner à ce projet une tentative🙂

### <a name="copyright-and-license"></a>Copyright et licence
Code et documentation Copyright 2016 **Federico Zivolo**. Code publié dans le cadre de la [licence MIT](LICENSE.md). Documents publiés sous Creative (Creative).
