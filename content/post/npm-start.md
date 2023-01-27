---
author: Sébastien Traber
title: Npm run xxx
date: 2023-01-27
math: true
---

Lors de mes nombreuses recherche sur le web, je suis tombé sur un article qui à retenue mon attention. Ce dernier s'institule __What happened to "npm run xxx"__ et explique ce que fait vraiment la commande `npm run xxx`

<!--more-->

En tant que développeur javascript, je travaille constamment avec `npm` et l'utilisation de la commande `npm run xxx` fait partis de mon quotidien. Cependant je ne m'était jamais demandé ce que faisait réellement cette commande. Cette article à ainsi pu répondre à cette nouvelle interogation.

Cet article est un petit résumé de l'article [What happened to "npm run xxx"][link1] L'article décrit une expérience d'entretien d'embauche d'un développeur front-end. L'intervieweur a posé des questions sur la commande `npm run xxx` et le développeur a admis ne pas la comprendre complètement.

la première question de l'intervieweur est la suivante. Pourquoi ne pas utiliser directement la commande `vue-cli-service serve` au lieu de `npm run serve` si le but de `npm run serve` est justement d'éxécuté `vue-cli-service serve` ?

On pourrait être tenté de répondre que ainsi la ccommande est plus rapide a tapper et à mémoriser. Cependant bien que cela ne soit pas faux ce n'est pas la bonne explications.

En réalité la commande `vue-cli-service serve` n'existe pas sur le système d'exploitation, il n'est donc pas possible de l'éxécuté de cette manière, raison pour laquelle on est obligé de passer par `npm` car `@vue/cli` est un outil installé au moyen de `npm`

L'intervieweur a ensuite demandé pourquoi `npm run serve` fonctionne sans erreur, même si `vue-cli-service serve` n'existe pas dans le système d'exploitation. Le développeur n'a pas su répondre tout de suite, mais a finalement découvert que lorsque `npm` installe une dépendance, il crée un fichier exécutable dans le répertoire `node_modules/.bin`. La commande `npm run` peut trouver et exécuter ce fichier en tant que script, même si la commande globale n'est pas installée. 

En conslusion cette article démontre l'importance de connaitre le fonctionnement des outils que nous utilisons au quotidien et de ne surtout pas hésiter à demander de l'aide lorsque c'est nécéssaire.

[link1]:https://javascript.plainenglish.io/interviewer-what-happened-to-npm-run-xxx-cdcb37dbaf44

