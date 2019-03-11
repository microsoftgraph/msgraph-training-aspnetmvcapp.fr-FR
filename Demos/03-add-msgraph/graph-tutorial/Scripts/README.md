<!-- IGNORE THE HTML BLOCK BELOW, THE INTERESTING PART IS AFTER IT -->

<h1 align="center"><span data-ttu-id="82e76-101">Popper. js</span><span class="sxs-lookup"><span data-stu-id="82e76-101">Popper.js</span></span></h1>

<p align="center"><span data-ttu-id="82e76-102">
    <strong>Bibliothèque utilisée pour positionner les pop-out dans les applications Web.</strong>
</span><span class="sxs-lookup"><span data-stu-id="82e76-102">
    <strong>A library used to position poppers in web applications.</strong>
</span></span></p>

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

## <a name="wut-poppers"></a><span data-ttu-id="82e76-103">Wut?</span><span class="sxs-lookup"><span data-stu-id="82e76-103">Wut?</span></span> <span data-ttu-id="82e76-104">Pop-up?</span><span class="sxs-lookup"><span data-stu-id="82e76-104">Poppers?</span></span>

<span data-ttu-id="82e76-105">Un Popper est un élément à l'écran qui «sort» du flux naturel de votre application.</span><span class="sxs-lookup"><span data-stu-id="82e76-105">A popper is an element on the screen which "pops out" from the natural flow of your application.</span></span>  
<span data-ttu-id="82e76-106">Des info-bulles, des popovers et des listes déroulantes sont des exemples courants de pop-up.</span><span class="sxs-lookup"><span data-stu-id="82e76-106">Common examples of poppers are tooltips, popovers and drop-downs.</span></span>


## <a name="so-yet-another-tooltip-library"></a><span data-ttu-id="82e76-107">Alors, une autre bibliothèque d'info-bulles?</span><span class="sxs-lookup"><span data-stu-id="82e76-107">So, yet another tooltip library?</span></span>

<span data-ttu-id="82e76-108">Eh bien, fondamentalement, **non**.</span><span class="sxs-lookup"><span data-stu-id="82e76-108">Well, basically, **no**.</span></span>  
<span data-ttu-id="82e76-109">Popper. js étant un **moteur de positionnement**, son objectif est de calculer la position d'un élément de sorte qu'il soit possible de le positionner à proximité d'un élément de référence donné.</span><span class="sxs-lookup"><span data-stu-id="82e76-109">Popper.js is a **positioning engine**, its purpose is to calculate the position of an element to make it possible to position it near a given reference element.</span></span>  

<span data-ttu-id="82e76-110">Le moteur est entièrement modulaire et la plupart de ses fonctionnalités sont implémentées en tant que **modificateurs** (semblables aux middlewares ou aux plugins).</span><span class="sxs-lookup"><span data-stu-id="82e76-110">The engine is completely modular and most of its features are implemented as **modifiers** (similar to middlewares or plugins).</span></span>  
<span data-ttu-id="82e76-111">L'intégralité de la base de code est écrite dans ES2015 et ses fonctionnalités sont testées automatiquement sur les navigateurs réels grâce à [SauceLabs](https://saucelabs.com/) et [TravisCI](https://travis-ci.org/).</span><span class="sxs-lookup"><span data-stu-id="82e76-111">The whole code base is written in ES2015 and its features are automatically tested on real browsers thanks to [SauceLabs](https://saucelabs.com/) and [TravisCI](https://travis-ci.org/).</span></span>

<span data-ttu-id="82e76-112">Popper. js n'a aucune dépendance.</span><span class="sxs-lookup"><span data-stu-id="82e76-112">Popper.js has zero dependencies.</span></span> <span data-ttu-id="82e76-113">Pas jQuery, no LoDash, Nothing.</span><span class="sxs-lookup"><span data-stu-id="82e76-113">No jQuery, no LoDash, nothing.</span></span>  
<span data-ttu-id="82e76-114">Elle est utilisée par les grandes entreprises comme [Twitter dans bootstrap v4](https://getbootstrap.com/), [Microsoft dans webclipper](https://github.com/OneNoteDev/WebClipper) et [Atlassian dans AtlasKit](https://aui-cdn.atlassian.com/atlaskit/registry/).</span><span class="sxs-lookup"><span data-stu-id="82e76-114">It's used by big companies like [Twitter in Bootstrap v4](https://getbootstrap.com/), [Microsoft in WebClipper](https://github.com/OneNoteDev/WebClipper) and [Atlassian in AtlasKit](https://aui-cdn.atlassian.com/atlaskit/registry/).</span></span>

### <a name="popperjs"></a><span data-ttu-id="82e76-115">Popper. js</span><span class="sxs-lookup"><span data-stu-id="82e76-115">Popper.js</span></span>

<span data-ttu-id="82e76-116">Il s'agit du moteur, la bibliothèque qui calcule et, éventuellement, applique les styles aux pop-up.</span><span class="sxs-lookup"><span data-stu-id="82e76-116">This is the engine, the library that computes and, optionally, applies the styles to the poppers.</span></span>

<span data-ttu-id="82e76-117">Voici quelques-uns des points clés:</span><span class="sxs-lookup"><span data-stu-id="82e76-117">Some of the key points are:</span></span>

- <span data-ttu-id="82e76-118">Placez les éléments en les conservant dans leur contexte DOM d'origine (ne fonctionne pas avec votre DOM!);</span><span class="sxs-lookup"><span data-stu-id="82e76-118">Position elements keeping them in their original DOM context (doesn't mess with your DOM!);</span></span>
- <span data-ttu-id="82e76-119">Permet d'exporter les informations calculées afin de les intégrer à des bibliothèques de révisions et d'autres bibliothèques d'affichage;</span><span class="sxs-lookup"><span data-stu-id="82e76-119">Allows to export the computed informations to integrate with React and other view libraries;</span></span>
- <span data-ttu-id="82e76-120">Prend en charge les éléments Shadow DOM;</span><span class="sxs-lookup"><span data-stu-id="82e76-120">Supports Shadow DOM elements;</span></span>
- <span data-ttu-id="82e76-121">Entièrement personnalisable grâce à la structure basée sur les modificateurs;</span><span class="sxs-lookup"><span data-stu-id="82e76-121">Completely customizable thanks to the modifiers based structure;</span></span>

<span data-ttu-id="82e76-122">Visitez notre [page de projet](https://fezvrasta.github.io/popper.js) pour voir un grand nombre d'exemples de ce que vous pouvez faire avec Popper. js!</span><span class="sxs-lookup"><span data-stu-id="82e76-122">Visit our [project page](https://fezvrasta.github.io/popper.js) to see a lot of examples of what you can do with Popper.js!</span></span>

<span data-ttu-id="82e76-123">Trouvez [la documentation ici](/docs/_includes/popper-documentation.md).</span><span class="sxs-lookup"><span data-stu-id="82e76-123">Find [the documentation here](/docs/_includes/popper-documentation.md).</span></span>


### <a name="tooltipjs"></a><span data-ttu-id="82e76-124">ToolTip. js</span><span class="sxs-lookup"><span data-stu-id="82e76-124">Tooltip.js</span></span>

<span data-ttu-id="82e76-125">Étant donné que de nombreux utilisateurs ont simplement besoin d'une méthode simple pour intégrer des info-bulles puissantes dans leurs projets, nous avons créé **ToolTip. js**.</span><span class="sxs-lookup"><span data-stu-id="82e76-125">Since lots of users just need a simple way to integrate powerful tooltips in their projects, we created **Tooltip.js**.</span></span>  
<span data-ttu-id="82e76-126">Il s'agit d'une petite bibliothèque qui facilite la création automatique d'info-bulles à l'aide d'un moteur Popper. js.</span><span class="sxs-lookup"><span data-stu-id="82e76-126">It's a small library that makes it easy to automatically create tooltips using as engine Popper.js.</span></span>  
<span data-ttu-id="82e76-127">Son API est quasiment identique au système de démarrage de démarrage célèbre, de cette manière, il est facile de l'intégrer dans vos projets.</span><span class="sxs-lookup"><span data-stu-id="82e76-127">Its API is almost identical to the famous tooltip system of Bootstrap, in this way it will be easy to integrate it in your projects.</span></span>  
<span data-ttu-id="82e76-128">Les info-bulles générées par ToolTip. js sont accessibles grâce `aria` aux balises.</span><span class="sxs-lookup"><span data-stu-id="82e76-128">The tooltips generated by Tooltip.js are accessible thanks to the `aria` tags.</span></span>

<span data-ttu-id="82e76-129">Trouvez [la documentation ici](/docs/_includes/tooltip-documentation.md).</span><span class="sxs-lookup"><span data-stu-id="82e76-129">Find [the documentation here](/docs/_includes/tooltip-documentation.md).</span></span>


## <a name="installation"></a><span data-ttu-id="82e76-130">Installation</span><span class="sxs-lookup"><span data-stu-id="82e76-130">Installation</span></span>
<span data-ttu-id="82e76-131">Popper. js est disponible sur les gestionnaires de package et CDN suivants:</span><span class="sxs-lookup"><span data-stu-id="82e76-131">Popper.js is available on the following package managers and CDNs:</span></span>

| <span data-ttu-id="82e76-132">Source</span><span class="sxs-lookup"><span data-stu-id="82e76-132">Source</span></span> |                                                                                  |
|:-------|:---------------------------------------------------------------------------------|
| <span data-ttu-id="82e76-133">npm</span><span class="sxs-lookup"><span data-stu-id="82e76-133">npm</span></span>    | `npm install popper.js --save`                                                   |
| <span data-ttu-id="82e76-134">ténacité</span><span class="sxs-lookup"><span data-stu-id="82e76-134">yarn</span></span>   | `yarn add popper.js`                                                             |
| <span data-ttu-id="82e76-135">NuGet</span><span class="sxs-lookup"><span data-stu-id="82e76-135">NuGet</span></span>  | `PM> Install-Package popper.js`                                                  |
| <span data-ttu-id="82e76-136">Bower</span><span class="sxs-lookup"><span data-stu-id="82e76-136">Bower</span></span>  | `bower install popper.js --save`                     |
| <span data-ttu-id="82e76-137">unpkg</span><span class="sxs-lookup"><span data-stu-id="82e76-137">unpkg</span></span>  | [`https://unpkg.com/popper.js`](https://unpkg.com/popper.js)                     |
| <span data-ttu-id="82e76-138">cdnjs</span><span class="sxs-lookup"><span data-stu-id="82e76-138">cdnjs</span></span>  | [`https://cdnjs.com/libraries/popper.js`](https://cdnjs.com/libraries/popper.js) |

<span data-ttu-id="82e76-139">ToolTip. js également:</span><span class="sxs-lookup"><span data-stu-id="82e76-139">Tooltip.js as well:</span></span>

| <span data-ttu-id="82e76-140">Source</span><span class="sxs-lookup"><span data-stu-id="82e76-140">Source</span></span> |                                                                                  |
|:-------|:---------------------------------------------------------------------------------|
| <span data-ttu-id="82e76-141">npm</span><span class="sxs-lookup"><span data-stu-id="82e76-141">npm</span></span>    | `npm install tooltip.js --save`                                                  |
| <span data-ttu-id="82e76-142">ténacité</span><span class="sxs-lookup"><span data-stu-id="82e76-142">yarn</span></span>   | `yarn add tooltip.js`                                                            |
| <span data-ttu-id="82e76-143">Bower \*</span><span class="sxs-lookup"><span data-stu-id="82e76-143">Bower\*</span></span> | `bower install tooltip.js=https://unpkg.com/tooltip.js --save`                   |
| <span data-ttu-id="82e76-144">unpkg</span><span class="sxs-lookup"><span data-stu-id="82e76-144">unpkg</span></span>  | [`https://unpkg.com/tooltip.js`](https://unpkg.com/tooltip.js)                   |
| <span data-ttu-id="82e76-145">cdnjs</span><span class="sxs-lookup"><span data-stu-id="82e76-145">cdnjs</span></span>  | [`https://cdnjs.com/libraries/popper.js`](https://cdnjs.com/libraries/popper.js) |

<span data-ttu-id="82e76-146">\*: Bower n'est pas officiellement pris en charge, il peut être utilisé pour installer ToolTip. js uniquement en creux le CDN unpkg.com.</span><span class="sxs-lookup"><span data-stu-id="82e76-146">\*: Bower isn't officially supported, it can be used to install Tooltip.js only trough the unpkg.com CDN.</span></span> <span data-ttu-id="82e76-147">Cette méthode a la limitation de ne pas pouvoir définir une version spécifique de la bibliothèque.</span><span class="sxs-lookup"><span data-stu-id="82e76-147">This method has the limitation of not being able to define a specific version of the library.</span></span> <span data-ttu-id="82e76-148">Bower et Popper. js suggèrent d'utiliser des NPM ou des fils pour vos projets.</span><span class="sxs-lookup"><span data-stu-id="82e76-148">Bower and Popper.js suggests to use npm or Yarn for your projects.</span></span>  
<span data-ttu-id="82e76-149">Pour plus d'informations, [Lisez le problème connexe](https://github.com/FezVrasta/popper.js/issues/390).</span><span class="sxs-lookup"><span data-stu-id="82e76-149">For more info, [read the related issue](https://github.com/FezVrasta/popper.js/issues/390).</span></span>

### <a name="dist-targets"></a><span data-ttu-id="82e76-150">Cibles de dist</span><span class="sxs-lookup"><span data-stu-id="82e76-150">Dist targets</span></span>

<span data-ttu-id="82e76-151">Popper. js est actuellement fourni avec 3 cibles à l'esprit: UMD, ESM et ESNext.</span><span class="sxs-lookup"><span data-stu-id="82e76-151">Popper.js is currently shipped with 3 targets in mind: UMD, ESM and ESNext.</span></span>

- <span data-ttu-id="82e76-152">UMD-définition de module universel: AMD, RequireJS et Globals;</span><span class="sxs-lookup"><span data-stu-id="82e76-152">UMD - Universal Module Definition: AMD, RequireJS and globals;</span></span>
- <span data-ttu-id="82e76-153">Modules ESM-ES: pour webpacking/ROLLUP ou pour les navigateurs prenant en charge les spécifications;</span><span class="sxs-lookup"><span data-stu-id="82e76-153">ESM - ES Modules: For webpack/Rollup or browser supporting the spec;</span></span>
- <span data-ttu-id="82e76-154">ESNext: disponible dans `dist/`, peut être utilisé avec WebPack et `babel-preset-env`;</span><span class="sxs-lookup"><span data-stu-id="82e76-154">ESNext: Available in `dist/`, can be used with webpack and `babel-preset-env`;</span></span>

<span data-ttu-id="82e76-155">Veillez à utiliser la bonne pour répondre à vos besoins.</span><span class="sxs-lookup"><span data-stu-id="82e76-155">Make sure to use the right one for your needs.</span></span> <span data-ttu-id="82e76-156">Si vous souhaitez l'importer avec une `<script>` balise, utilisez UMD.</span><span class="sxs-lookup"><span data-stu-id="82e76-156">If you want to import it with a `<script>` tag, use UMD.</span></span>

## <a name="usage"></a><span data-ttu-id="82e76-157">Utilisation</span><span class="sxs-lookup"><span data-stu-id="82e76-157">Usage</span></span>

<span data-ttu-id="82e76-158">À partir d'un nœud DOM Popper existant, demandez à Popper. js de le placer près de son bouton.</span><span class="sxs-lookup"><span data-stu-id="82e76-158">Given an existing popper DOM node, ask Popper.js to position it near its button</span></span>

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

### <a name="callbacks"></a><span data-ttu-id="82e76-159">Rappels</span><span class="sxs-lookup"><span data-stu-id="82e76-159">Callbacks</span></span>

<span data-ttu-id="82e76-160">Popper. js prend en charge deux types de rappels `onCreate` , le rappel est appelé après l'initialisation de l'Popper.</span><span class="sxs-lookup"><span data-stu-id="82e76-160">Popper.js supports two kinds of callbacks, the `onCreate` callback is called after the popper has been initalized.</span></span> <span data-ttu-id="82e76-161">Le `onUpdate` premier est appelé sur toute mise à jour ultérieure.</span><span class="sxs-lookup"><span data-stu-id="82e76-161">The `onUpdate` one is called on any subsequent update.</span></span>

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

### <a name="writing-your-own-modifiers"></a><span data-ttu-id="82e76-162">Écriture de vos propres modificateurs</span><span class="sxs-lookup"><span data-stu-id="82e76-162">Writing your own modifiers</span></span>

<span data-ttu-id="82e76-163">Popper. js est basé sur une architecture «de plug-in», la plupart de ses fonctionnalités sont entièrement encapsulées «modificateurs».</span><span class="sxs-lookup"><span data-stu-id="82e76-163">Popper.js is based on a "plugin-like" architecture, most of its features are fully encapsulated "modifiers".</span></span>  
<span data-ttu-id="82e76-164">Un modificateur est une fonction qui est appelée chaque fois que Popper. js doit calculer la position de l'Popper.</span><span class="sxs-lookup"><span data-stu-id="82e76-164">A modifier is a function that is called each time Popper.js needs to compute the position of the popper.</span></span> <span data-ttu-id="82e76-165">Pour cette raison, les modificateurs doivent être très performants pour éviter les goulets d'étranglement.</span><span class="sxs-lookup"><span data-stu-id="82e76-165">For this reason, modifiers should be very performant to avoid bottlenecks.</span></span>  

<span data-ttu-id="82e76-166">Pour en savoir plus sur la création d'un modificateur, [consultez la documentation sur les modificateurs](docs/_includes/popper-documentation.md#modifiers--object)</span><span class="sxs-lookup"><span data-stu-id="82e76-166">To learn how to create a modifier, [read the modifiers documentation](docs/_includes/popper-documentation.md#modifiers--object)</span></span>


### <a name="react-vuejs-angular-angularjs-emberjs-etc-integration"></a><span data-ttu-id="82e76-167">Intégration de REACT, vue. js, angulaire, AngularJS, Ember. js (etc...)</span><span class="sxs-lookup"><span data-stu-id="82e76-167">React, Vue.js, Angular, AngularJS, Ember.js (etc...) integration</span></span>

<span data-ttu-id="82e76-168">L'intégration de bibliothèques tierces dans les bibliothèques REACT ou d'autres bibliothèques peut être une douleur, car elles modifient généralement le DOM et rendent les bibliothèques fou.</span><span class="sxs-lookup"><span data-stu-id="82e76-168">Integrating 3rd party libraries in React or other libraries can be a pain because they usually alter the DOM and drive the libraries crazy.</span></span>  
<span data-ttu-id="82e76-169">Popper. js limite toutes les modifications apportées `applyStyle` au modèle DOM à l'intérieur du modificateur, vous pouvez simplement le désactiver et appliquer manuellement les coordonnées Popper à l'aide de votre bibliothèque de choix.</span><span class="sxs-lookup"><span data-stu-id="82e76-169">Popper.js limits all its DOM modifications inside the `applyStyle` modifier, you can simply disable it and manually apply the popper coordinates using your library of choice.</span></span>  

<span data-ttu-id="82e76-170">Pour obtenir la liste complète des bibliothèques qui vous permettent d'utiliser Popper. js dans des infrastructures existantes, [](/MENTIONS.md) visitez la page mentions.</span><span class="sxs-lookup"><span data-stu-id="82e76-170">For a comprehensive list of libraries that let you use Popper.js into existing frameworks, visit the [MENTIONS](/MENTIONS.md) page.</span></span>

<span data-ttu-id="82e76-171">Vous pouvez également remplacer votre propre par votre propre `applyStyles` et intégrer Popper. js par vous-même!</span><span class="sxs-lookup"><span data-stu-id="82e76-171">Alternatively, you may even override your own `applyStyles` with your custom one and integrate Popper.js by yourself!</span></span>

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

### <a name="migration-from-popperjs-v0"></a><span data-ttu-id="82e76-172">Migration à partir de Popper. js v0</span><span class="sxs-lookup"><span data-stu-id="82e76-172">Migration from Popper.js v0</span></span>

<span data-ttu-id="82e76-173">Étant donné que l'API a changé, nous avons préparé des instructions de migration pour faciliter la mise à niveau vers Popper. js v1.</span><span class="sxs-lookup"><span data-stu-id="82e76-173">Since the API changed, we prepared some migration instructions to make it easy to upgrade to Popper.js v1.</span></span>  

https://github.com/FezVrasta/popper.js/issues/62

<span data-ttu-id="82e76-174">N'hésitez pas à commenter le problème si vous avez des questions.</span><span class="sxs-lookup"><span data-stu-id="82e76-174">Feel free to comment inside the issue if you have any questions.</span></span>

### <a name="performances"></a><span data-ttu-id="82e76-175">Performance</span><span class="sxs-lookup"><span data-stu-id="82e76-175">Performances</span></span>

<span data-ttu-id="82e76-176">Popper. js est très performant.</span><span class="sxs-lookup"><span data-stu-id="82e76-176">Popper.js is very performant.</span></span> <span data-ttu-id="82e76-177">Il faut généralement 0,5 ms pour calculer la position d'une Popper (sur un iMac avec 3,5 G Intel Core i5).</span><span class="sxs-lookup"><span data-stu-id="82e76-177">It usually takes 0.5ms to compute a popper's position (on an iMac with 3.5G GHz Intel Core i5).</span></span>  
<span data-ttu-id="82e76-178">Cela signifie qu'elle n'entraîne aucune [Jank](https://www.chromium.org/developers/how-tos/trace-event-profiling-tool/anatomy-of-jank), ce qui se traduit par une expérience utilisateur sans complication.</span><span class="sxs-lookup"><span data-stu-id="82e76-178">This means that it will not cause any [jank](https://www.chromium.org/developers/how-tos/trace-event-profiling-tool/anatomy-of-jank), leading to a smooth user experience.</span></span>

## <a name="notes"></a><span data-ttu-id="82e76-179">Notes</span><span class="sxs-lookup"><span data-stu-id="82e76-179">Notes</span></span>

### <a name="libraries-using-popperjs"></a><span data-ttu-id="82e76-180">Bibliothèques à l'aide de Popper. js</span><span class="sxs-lookup"><span data-stu-id="82e76-180">Libraries using Popper.js</span></span>

<span data-ttu-id="82e76-181">L'objectif de Popper. js est de fournir un moteur de positionnement stable et puissant prêt à être utilisé dans les bibliothèques tierces.</span><span class="sxs-lookup"><span data-stu-id="82e76-181">The aim of Popper.js is to provide a stable and powerful positioning engine ready to be used in 3rd party libraries.</span></span>  

<span data-ttu-id="82e76-182">Visitez la [](/MENTIONS.md) page des mentions pour obtenir une liste mise à jour des projets.</span><span class="sxs-lookup"><span data-stu-id="82e76-182">Visit the [MENTIONS](/MENTIONS.md) page for an updated list of projects.</span></span>


### <a name="credits"></a><span data-ttu-id="82e76-183">Crédits</span><span class="sxs-lookup"><span data-stu-id="82e76-183">Credits</span></span>
<span data-ttu-id="82e76-184">Je souhaite remercier des amis et des projets pour le travail qu'il a effectué:</span><span class="sxs-lookup"><span data-stu-id="82e76-184">I want to thank some friends and projects for the work they did:</span></span>

- <span data-ttu-id="82e76-185">[@AndreaScn](https://github.com/AndreaScn) de son travail sur la page GitHub et le test manuel qu'il a fait lors du développement;</span><span class="sxs-lookup"><span data-stu-id="82e76-185">[@AndreaScn](https://github.com/AndreaScn) for his work on the GitHub Page and the manual testing he did during the development;</span></span>
- <span data-ttu-id="82e76-186">[@vampolo](https://github.com/vampolo) de l'idée d'origine et du nom de la bibliothèque;</span><span class="sxs-lookup"><span data-stu-id="82e76-186">[@vampolo](https://github.com/vampolo) for the original idea and for the name of the library;</span></span>
- <span data-ttu-id="82e76-187">[Sysdig](https://github.com/Draios) pour toutes les choses géniales que j'ai apprises au cours de ces années, qui m'a permis d'écrire cette bibliothèque;</span><span class="sxs-lookup"><span data-stu-id="82e76-187">[Sysdig](https://github.com/Draios) for all the awesome things I learned during these years that made it possible for me to write this library;</span></span>
- <span data-ttu-id="82e76-188">[Attache. js](http://github.hubspot.com/tether/) pour m'avoir inspiré lors de l'écriture d'une bibliothèque de positionnement prête pour le monde réel;</span><span class="sxs-lookup"><span data-stu-id="82e76-188">[Tether.js](http://github.hubspot.com/tether/) for having inspired me in writing a positioning library ready for the real world;</span></span>
- <span data-ttu-id="82e76-189">[Les contributeurs](https://github.com/FezVrasta/popper.js/graphs/contributors) de leurs requêtes d'extraction et de bogues très appréciées;</span><span class="sxs-lookup"><span data-stu-id="82e76-189">[The Contributors](https://github.com/FezVrasta/popper.js/graphs/contributors) for their much appreciated Pull Requests and bug reports;</span></span>
- <span data-ttu-id="82e76-190">**vous** pour l'étoile que vous allez donner à ce projet et, si vous le souhaitez, donner à ce projet une tentative🙂</span><span class="sxs-lookup"><span data-stu-id="82e76-190">**you** for the star you'll give to this project and for being so awesome to give this project a try 🙂</span></span>

### <a name="copyright-and-license"></a><span data-ttu-id="82e76-191">Copyright et licence</span><span class="sxs-lookup"><span data-stu-id="82e76-191">Copyright and license</span></span>
<span data-ttu-id="82e76-192">Code et documentation Copyright 2016 **Federico Zivolo**.</span><span class="sxs-lookup"><span data-stu-id="82e76-192">Code and documentation copyright 2016 **Federico Zivolo**.</span></span> <span data-ttu-id="82e76-193">Code publié dans le cadre de la [licence MIT](LICENSE.md).</span><span class="sxs-lookup"><span data-stu-id="82e76-193">Code released under the [MIT license](LICENSE.md).</span></span> <span data-ttu-id="82e76-194">Documents publiés sous Creative (Creative).</span><span class="sxs-lookup"><span data-stu-id="82e76-194">Docs released under Creative Commons.</span></span>
