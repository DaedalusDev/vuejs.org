---
title: Slots
type: guide
order: 104
---

> Cette page suppose que vous avez déjà lu les principes de base des [composants](components.html). Lisez-les en premier si les composants sont quelque chose de nouveau pour vous.

## Les « Slot content » ou « Contenu de slot »

Vue implémente une API de distribution de contenu inspiré du [Brouillon de spécification des WebComponents](https://github.com/w3c/webcomponents/blob/gh-pages/proposals/Slots-Proposal.md) utilisant l'élément `<slot>` comme zone d'atterissage pour la distribution du contenu.

Celà vous permet de composer vos composant ainsi :
``` html
<navigation-link url="/profile">
  Mon profil
</navigation-link>
```

Dans le template `<navigation-link>`, nous aurons :

``` html
<a
  v-bind:href="url"
  class="nav-link"
>
  <slot></slot>
</a>
```

Lors du cycle de rendu du composent, l'élément `<slot>` est remplacé par « Mon profil ». Les éléments `<slot>` peuvent contenir n'importe quel code de template, incluant le HTML :

``` html
<navigation-link url="/profile">
  <!-- Mon icone -->
  <span class="fa fa-user"></span>
  Mon profil
</navigation-link>
```

Ou encore faire appel à d'autres composants :

``` html
<navigation-link url="/profile">
  <!-- Utilisation d'un composant dédié à l'ajout d'une icone -->
  <font-awesome-icon name="user"></font-awesome-icon>
  Mon profil
</navigation-link>
```

Si `<navigation-link>` ne contient **pas** d'élément `<slot>`, le contenu enfant sera simplement ignoré.

## Les Slots nommés

Dans certains cas, il peut être interessant d'avoir plusieurs éléments `<slot>`. Dans un exemple hypothetique, voici le template d'un composant `<base-layout>`:

``` html
<div class="container">
  <header>
    <!-- Ici le contenu de l'entête -->
  </header>
  <main>
    <!-- Ici le contenu courant -->
  </main>
  <footer>
    <!-- Ici le pied de page -->
  </footer>
</div>
```

Dans le cas suivant, l'élément `<slot>` à un l'attribut spécial `name` , qui peut être utilisé pour désigner des slots additionnels :

``` html
<div class="container">
  <header>
    <slot name="header"></slot>
  </header>
  <main>
    <slot></slot>
  </main>
  <footer>
    <slot name="footer"></slot>
  </footer>
</div>
```

Afin de distribuer le contenu dans les éléments `<slot>` appropriés, il suffit d'utiliser l'attribut réservé `slot` sur un élément `<template>` du composant parent :

```html
<base-layout>
  <template slot="header">
    <h1>Le titre de ma page</h1>
  </template>

  <p>Un paragraphe pour le slot par défaut.</p>
  <p>Un autre paragraphe.</p>

  <template slot="footer">
    <p>Ici les infos de contact</p>
  </template>
</base-layout>
```

Ou utiliser l'attribut `slot` directement sur un élément normal:

``` html
<base-layout>
  <h1 slot="header">Le titre de ma page</h1>

  <p>Un paragraphe pour le slot par défaut.</p>
  <p>Un autre paragraphe.</p>

  <p slot="footer">Ici les infos de contact</p>
</base-layout>
```

Il ne peut y avoir qu'un seul **default slot** qui recevra tous les éléments qui ne correspondent à aucun des slots nommés. Dans l'exemple précédent, le rendu HTML produit sera :

``` html
<div class="container">
  <header>
    <h1>Le titre de ma page</h1>
  </header>
  <main>
    <p>Un paragraphe pour le slot par défaut.</p>
    <p>Un autre paragraphe.</p>
  </main>
  <footer>
    <p>Ici les infos de contact</p>
  </footer>
</div>
```

## Les slots avec contenu par défaut

Dans certains cas, il peut être interessant de fournir un contenu par défaut pour vos slots. Par exemple, qu'un composant `<submit-button>` contienne le texte 'Envoyer' par défaut et que ce contenu soit facilement modifié par 'Sauvegarder', 'Télécharger', ou autre.

Pour procéder de la sorte, il est possible de spécifier le contenu par défaut d'une balise `<slot>`.

```html
<button type="submit">
  <slot>Envoyer</slot>
</button>
```

Si le `<slot>` est rempli dans le composant parent, le contenu par défaut est remplacé.

## Scope de compilation du template

Quand on utilise de la donnée dans le slot, comme dans l'exemple suivant :

``` html
<navigation-link url="/profile">
  Connecté en tant que {{ user.name }}
</navigation-link>
```

Le `<slot>` bénéficie des mêmes propriétés d'instance (même « scope » ou « contexte ») que le reste du template parent. Le `<slot>` **ne** bénéficie en aucun cas des éléments du « scope » de l'instance de `<navigation-link>`. Par exemple, tenter d'accéder à la props `url` ne fonctionnera pas. A titre de règle générale, souvenez vous que :

> Tout ce qui est dans le `<template>` parent est compilé dans le contexte parent. Tout ce qui est dans le `<template>` enfant est compilé dans le contexte enfant.

## Les slots scopés (ou slots avec accès au child scope)

> Nouveauté en 2.1.0+

Dans certains cas, il peut être interessant lors de l'utilisation d'un composant avec éléments `<slot>` réutilisables, d'accéder à de la donnée du scope du composant enfant. Voici l'exemple d'un composant `<todo-list>` dont le `<template>` est :

```html
<ul>
  <li
    v-for="todo in todos"
    v-bind:key="todo.id"
  >
    {{ todo.text }}
  </li>
</ul>
```

Dans certaines parties de l'application, il serait interessant de personnaliser le rendu pour chaque éléments de la liste, plutôt que d'afficher seulement `todo.text`. C'est possible avec les slots scopés.

Pour utiliser cette fonctionnalité, nous allons englober le contenu du rendu d'élément de liste avec un élément `<slot>`, et fournir un jeu de donnée provenant de son contexte : dans notre cas, l'objet `todo`) :

```html
<ul>
  <li
    v-for="todo in todos"
    v-bind:key="todo.id"
  >
    <!-- Un slot est créé pour chaque éléments de "todos" -->
    <!-- Chaque objet `todo` va bénéfichier d'un "slot prop" -->
    <slot v-bind:todo="todo">
      <!-- Ici le contenu par défaut du slot -->
      {{ todo.text }}
    </slot>
  </li>
</ul>
```

Lorsqu'on utiliser le composant `<todo-list>`, il devient alors possible de fournir de façon optionnelle un `<template>` alternatif de rendu des éléments de la liste, et d'accèder à la donnée du `slot` enfant avec l'attribut `slot-scope` :

```html
<todo-list v-bind:todos="todos">
  <!-- Définir `slotProps` pour assigner la donnée du slot dans une variable -->
  <template slot-scope="lesPropsDuSlot">
    <!-- Ici, nous allons personnaliser le rendu du slot -->
    <!-- `lesPropsDuSlot` contiendra les props fournies par v-bind dans le slot enfant -->
    <span v-if="lesPropsDuSlot.todo.isComplete">✓</span>
    {{ lesPropsDuSlot.todo.text }}
  </template>
</todo-list>
```

> Dans 2.5.0+, `slot-scope` n'est plus limité aux éléments  `<template>` et peut être utilisé sur n'importe quel élément ou composant (`<div>`, `<my-component>`, ...)

### Affectation par décomposition du `slot-scope`

La valeur de l'attribut `slot-scope` peut actuellement recevoir une expression JavaScript valide qui pourrait apparaitre en argument de définition d'une fonction. Attention cependant, cette fonctionnalité est soumise à l'utilisation des environnements suivants : [Composants monofichiers](single-file-components.html) ou [Navigateurs modernes](https://developer.mozilla.org/fr/docs/Web/JavaScript/Reference/Op%C3%A9rateurs/Affecter_par_d%C3%A9composition#Compatibilité_des_navigateurs) vous pouvez alors utiliser [Affecter par décomposition (ES2015)](https://developer.mozilla.org/fr/docs/Web/JavaScript/Reference/Op%C3%A9rateurs/Affecter_par_d%C3%A9composition) dans l'expression, de cette manière :

```html
<todo-list v-bind:todos="todos">
  <template slot-scope="{ todo }">
    <span v-if="todo.isComplete">✓</span>
    {{ todo.text }}
  </template>
</todo-list>
```

Celà peut être pratique pour rendre vos slot scopés un peu plus lisibles.
