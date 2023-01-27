---
author: Sébastien Traber
title: Les web components
date: 2023-01-27
math: true
---

Grâce à l'un de mes abonnements YouTube, en l'occurrence le youtubeur [Web Dev Simplified](https://www.youtube.com/watch?v=2I7uX8m0Ta0&t=720s) j'ai découvert le principe des web composants <!--more--> qui permet structurer le code de ses app avec une approche composant. Ceci permet ainsi de se passer des grandes libraires/framework front-end tels que react, vue ou angular.

Les web composants sont une collection de technologies qui permettent aux développeurs de créer des composants personnalisés pour les navigateurs web. Ces composants peuvent être réutilisés à travers plusieurs pages et applications web, offrant ainsi une meilleure modularité et une meilleure maintenance du code.

Il existe plusieurs technologies qui composent les web components, notamment :

- Les Custom Éléments, qui permettent de créer des éléments HTML personnalisés avec leur propre comportement et style.
- L'API Shadow DOM, qui permet de créer un arbre DOM (Document Object Model) isolé pour chaque composant, ce qui permet de séparer le contenu d'un composant de sa présentation.
- L'API HTML Templates, qui permet de définir un modèle de code HTML qui peut être utilisé pour créer des éléments répétitifs.

Voici un exemple de code qui utilise les Custom Éléments pour créer un composant de bouton :

```html
<button-component>Mon bouton</button-component>

<script>
class ButtonComponent extends HTMLElement {
  constructor() {
    super();
    this.attachShadow({ mode: 'open' });
    this.shadowRoot.innerHTML = `
      <style>
        button {
          background-color: blue;
          color: white;
          padding: 8px 16px;
          border: none;
          cursor: pointer;
        }
      </style>
      <button><slot></slot></button>
    `;
  }
}
customElements.define('button-component', ButtonComponent);
</script>
```

Ce code crée un nouvel élément HTML <button-component> qui affiche un bouton bleu avec du texte blanc. La propriété <slot> permet de spécifier où le contenu de l'élément sera affiché, dans ce cas-ci c'est dans le bouton.

Voici un autre exemple qui utilise l'API Shadow DOM pour créer un composant de carte :

```html
<card-component>
  <h1 slot="header">Titre de la carte</h1>
  <p slot="content">Contenu de la carte</p>
</card-component>

<script>
class CardComponent extends HTMLElement {
  constructor() {
    super();
    this.attachShadow({ mode: 'open' });
    this.shadowRoot.innerHTML = `
      <style>
        .card {
          background-color: white;
          border: 1px solid gray;
          padding: 16px;
        }
        .card h1 {
          margin-top: 0;
        }
      </style>
      <div class="card">
        <slot name="header"></slot>
        <slot name="content"></slot>
      </div>
    `;
  }
}
customElements.define('card-component', CardComponent);
</script>
```

## Stylisation des web composants

L'encapsulation du style dans les web components permet de séparer le contenu d'un composant de sa présentation. Cela est rendu possible grâce à l'utilisation de l'API Shadow DOM, qui permet de créer un arbre DOM (Document Object Model) isolé pour chaque composant.

Lorsqu'un composant utilise le Shadow DOM, tous les styles définis à l'intérieur de celui-ci n'affecteront que les éléments de ce composant et n'interféreront pas avec les styles des autres éléments de la page. De plus, les styles définis à l'extérieur du composant ne seront pas appliqués aux éléments à l'intérieur du composant.
Cela permet de créer des composants qui sont indépendants de leur environnement, ce qui facilite la réutilisation de ces composants dans différentes pages et applications.

Voici un exemple de code qui utilise l'API Shadow DOM pour encapsuler les styles d'un composant :

```html
<div id="app">
  <my-component>Contenu de mon composant</my-component>
  <style>
    /* styles qui s'appliquent à toute la page */
  </style>
</div>

<script>
class MyComponent extends HTMLElement {
  constructor() {
    super();
    this.attachShadow({ mode: 'open' });
    this.shadowRoot.innerHTML = `
      <style>
        /* styles qui s'appliquent uniquement à mon composant */
      </style>
      <div>
        <slot></slot>
      </div>
    `;
  }
}
customElements.define('my-component', MyComponent);
</script>
```

Dans cet exemple, nous avons défini un composant `my-component`, qui possède un arbre DOM isolé grâce à l'utilisation de la méthode attachShadow. Les styles définis à l'intérieur de ce composant n'affecteront que les éléments à l'intérieur de celui-ci, et ne seront pas affectés par les styles définis à l'extérieur de ce composant, dans la page d'accueil par exemple.

En encapsulant les styles de cette façon, les composants sont plus faciles à maintenir et à réutiliser, car ils ne dépendent pas des styles définis dans leur environnement. Cela permet également de limiter les conflits de styles entre différents composants et de faciliter l'application de styles uniques à chaque composant.

## Conclusion 

En conclusion, les web components offrant une meilleure modularité et une meilleure maintenance du code en permettant de réutiliser des composants à travers plusieurs pages et applications web. Les Custom Éléments, l'API Shadow DOM et l'API HTML Templates sont les principales technologies qui composent les web composants et peuvent être utilisées ensemble pour créer des composants plus complets et avancés. Ces technologies sont en cours d'évolution constante et de plus en plus de navigateurs les prennent en charge, il est donc important de suivre les dernières tendances pour pouvoir en tirer le meilleur parti.
