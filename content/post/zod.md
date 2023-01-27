---
author: Sébastien Traber
title: Validation de données en JavaScript avec Zod
date: 2023-01-27
math: true
---

Grâce à l'un de mes abonnements YouTube, en l'occurrence le youtubeur [Web Dev Simplified](https://www.youtube.com/watch?v=9UVPk0Ulm6U) j'ai découvert une librairie JavaScript très utile <!--more--> que je pourrais être susceptible d'utiliser dans un de mes projets pour la validation de données des formulaires.

Zod est une librairie JavaScript open source pour la validation de données. Il permet de définir des schémas de validation pour vos objets JavaScript et de vérifier si des objets donnés respectent ces schémas. Cela peut être utile pour vous assurer que les données d'entrée d'une API ou d'une application respectent les attentes de votre application.

## Installation

Pour installer Zod, vous pouvez utiliser npm ou yarn :

```bash
npm install zod
```

ou

```bash
yarn add zod
```

## Utilisation

Voici un exemple d'utilisation de Zod pour valider un objet contenant un nom et un âge :

```javascript
const z = require('zod');

const schema = z.object({
  name: z.string().nonempty(),
  age: z.number().positive()
});

const data = { name: 'John', age: 30 };

try {
  schema.validate(data);
  console.log('Validation réussie !');
} catch (err) {
  console.log(err.message);
}
```

Dans cet exemple, nous avons défini un schéma qui attend un objet avec une propriété `name` qui est une chaîne de caractères non vide et une propriété `age` qui est un nombre positif. Nous avons ensuite utilisé ce schéma pour valider un objet avec les valeurs `name: 'John', age: 30`. Si l'objet respecte le schéma, la validation réussit et un message est imprimé. Sinon, une erreur est levée et le message d'erreur est imprimé.

## Types de données pris en charge

Zod prend en charge les types de données suivants :

- number
- string
- boolean
- object
- array
- date

Il fournit également des validateurs pour vérifier si un nombre est positif ou négatif, si une chaîne de caractères est vide ou non, etc.

## Conclusion

Zod est un outil pratique pour valider les données d'entrée de votre application. Il permet de définir des schémas de validation clairs et de vérifier facilement si des objets respectent ces schémas. Il est facile à installer et à utiliser, et prend en charge un grand nombre de types de données.
