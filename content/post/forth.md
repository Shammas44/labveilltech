---
author: Sébastien Traber
title: Le language de programmation forth
date: 2023-01-27
math: true
---

Sur mon temps libre j'ai commancé la lecture d'une livre intitulé Strange Code écris par Ronald T. Kneusel. Ce livre s'intéresse à décortiquer des langages de progrmmation étrange.

<!--more-->

Il s'agit pour moi d'une bonne façon d'approfondir mes connaisances en programmation en étudiant d'autres languages parfois même très bas niveau. C'est ainsi que j'ai découvert le langage nommé `forth` 

Forth est un langage de programmation impératif, procédural et réflexif. Il est basé sur un modèle de pile, ce qui signifie que les données et les instructions sont stockées dans une pile et manipulées à partir de celle-ci. Les instructions Forth sont généralement très courtes et peuvent être combinées pour créer des programmes plus complexes.

Les principales caractéristiques de Forth sont sa flexibilité, sa simplicité et sa puissance. Il est encore utilisé aujaurd'hui dans certaines applications embarquées. Une des particularité de `forth` c'est que l'utilisateur va intéragir directement avec la `stackpile` comme nous les verons dans les exemples suivants.

__installer gforth__

```bash
sudo apt-get install gforth
```

__Quitter gforth__

```forth
bye
```

__Exemple basic__

```forth
cr 1 2 3 .
```
- `cr`: ordonne au prompte de se déplacer sur la ligne suivante
- `1 2 3`: insére consécutivement `1`, `2` puis `3` dans la `stackpil`
- `.`: affiche la valeur de la stack supérieur

__Supprimer toute la pile__

```forth
clear
```

## Manipuler la pile

| Word  | Effect             | Description                         |
| :---: | :----------------: | :---------------------------------: |
| dup   | ( a -- a a)        | Duplicate the top stack item        |
| drop  | ( a b -- a  )      | Drop the top of stack item          |
| swap  | ( a b -- b a )     | Swap the top two stack items        |
| 2dup  | ( a b -- a b a b ) | Duplicate the top two stack items   |
| 2drop | ( a b c  --  a )   | Drop the top two stack items        |
| over  | ( a b -- a b a )   | Copy the next to top of stack items |
| rot   | ( a b c -- b c a ) | Rotate the top three stack items    |
| nip   | ( a b c -- a c )   | Drop the next to top of stack item  |
| .s    | ( -- )             | Print the stack without altering it |

## Opérations mathématiques

> 📚 __Ordre des opérations__ <br>
Forth ne respecte pas l'ordre standart des opérations mathématiques. Au contraire les calculs sont efféctué de gauche à droite, cependant les opérateurs sont toujours placé avec les nombres. Ainsi `2 + 3` devient `2 3 +` avec forth
>

- (1200 * 3) / 4 -> `1200 3 * 4 /`
- 8 * (127 - 9) / 11 -> `127 9 - 8 * 11 /`
- 8 * (127 - 9) mod 11 -> `127 9 - 8 * 11 mod`
- ((33 - 45) / (7 + 9)) * 3 -> `33 45 - 7 9 + / 3 *`

## Définir notre propre mots clé

La commande `:` permet de créer un nouveau mot-clé qui peut être utilisé pour exécuter un ensemble d'instructions. 

__Exemple 1: addition__

Par exemple, définir un mot-clé `add` qui prend deux nombres en entrée `n1` et `n2` et renvoie leur somme. 

```forth
: add ( n1 n2 -- sum ) + ;
2 3 add . <1> 5 ok
```

- `:` sers à définir un nouveau mot clé 
- `add` le nom de mot clé
- `( n1 n2 -- sum )` commentaire _facultatif_ `( input1 -- returnValue )`
- `+` le corps de la fonction, dans ce cas effectue une addition
- `;` indique la fin de la défintion du mot clé

__Exemple 2: hello world__

```forth
( affiche Hello, world! )
: hi ( -- ) ." Hello, world!" cr ;
( affiche 10 fois Hello, world! dans la console )
: hello ( -- ) 10 0 do hi loop ;
```
- 📚 `."` début de la compilation d'une chaine de charactère
- ⚠️  l'espace avant Hello world est requis

## boucle

Le mot clé `loop` ne peut être utilisé que dans une définition de mot clé

__Syntaxe d'une boucle__

```
<end> <start> do <body> loop
```

__Boucle for en c__

```c
for (int i=0; i < 10; i++)
  printf("Hello, world!\n")
```

__Boucle for en forth__

```forth
: counter ( -- ) 10 0 do i . loop ;
counter 0 1 2 3 4 5 6 7 8 9 ok
```

__Boucle for en forth incrémenté de 3__

```forth
: counter ( -- ) 20 0 do i . 3 +loop ;
counter 0 3 6 9 12 15 18 ok
```

__Boucle for en forth utilisant l'index comme incrément__

```forth
: counter ( -- ) 100 0 do i dup . +loop ;
counter 1 2 4 8 16 32 64 ok
```

__Double boucle en forth__

```forth
: nested ( -- )
  3 0 do
    3 0 do  
      j . i . space
    loop
  loop ;
```
- 📚: `space` affiche un espace

## Boolean

- `0` -> `false`
- `-1` -> `true`
- `>` -> plus grand que
- `<` -> plus petit que
- `=` -> égal
- `<>` -> pas égal

```forth
1 2 < . -1 
-123 321 < . -1 
45 3 > . -1 
3 45 = . 0
3 45 <> . -1
```

## Conditions

__Syntaxe__

```
<condition> if <true_instructions> then
<condition> if <true_instructions> else <false_instructions> then
```

```python
def porridge(n):
  print("The porridge is ", end="")
  if (n < 90):
    print("too cold")  
  elsif (n > 100):
    print("too hot")  
  else:
    print("just right")
```

__Exemple de condition__

```forth
: porridge ( n -- )
  ." The porridge is "
  dup 90 < if drop ." too cold" else
     100 < if ." just right"    else
    ." too hot"
  then then cr ;
  
```

```
80 porridge The porridge is to cold
ok
99 porridge The porridge is just right
ok
111 porridge The porridge is too hot
```

__Explications__

```forth
dup 90 < if drop ." too cold" else
```
1. `dup` duplique la température, 
- 📚 Nécéssaire car `<` consomme `n` et `90` pour tester que `n < 90`
- 📚 Si le premier `if` échoue, le second `if` nécéssite une copy de `n`
2. Le résultat `0` ou `-1` est ensuite poussé au dessus de la pile
- 📚 A ce stade la pile contient `n` et le résultat de la condition
3. le prochain `if` consomme le résultat de la condition
4. Si le premier test passe, `drop` supprime la duplication de `n`
- 📚 Autrement ce dernier reste dans la pile

### Swtich

Certaines version de forth dont nottament `gforth` inclue une forme de switch similaire au `c`

```forth
: menu ( n -- )
  case
    0 of ." option 0" endof
    1 of ." option 1" endof
    2 of ." option 2" endof
    ." bad option"
  endcase ;
```

```
0 menu option 0 ok
1 menu option 1 ok
2 menu option 2 ok
3 menu bad option ok
```

## Autres formes de boucles

__Syntaxes__

```forth
(  boucle while )
begin <condition> while <body> repeat
(  do while )
begin <body> <condition> until
(  boucle infinie )
begin <body> again
```

```forth
( boucle infinie qui affiche tout les nombres de 0 à infinie)
: infinity ( -- ) 0 begin dup . cr 1+ again ;
```

```forth
( calcul la racine carré de n )
: dsqr ( n -- ) 0 begin 2dup dup * > while 1+ repeat nip ; 
4 dsqr .s 
<1> 2 ok ( affiche bien 2 )
```

__Explications__

| Etapes | Pile      |
| :----: | :-------: |
| 4 dsqr | 4         |
| 0      | 4 0       |
| begin  | 4 0       |
| 2dup   | 4 0 4 0   |
| dup    | 4 0 4 0 0 |
| *      | 4 0 4 0   |
| >      | 4 0 -1    |
| while  | 4 0       |
| 1+     | 4 1       |
| repeat | 4 1       |

- 📚 `nip` éxécuté dès que la condition est fausse, supprimer `n` de la pile

## Variables et constantes

```forth
variable f
variable c 
32 constant b
: c2f c @ 9 * 5 / b + f ! ; ( celsius to farenheit)
: f2c f @ b - 5 * 9 / c ! ; ( farenheit to celsius)
212 f ! f2c c @ . 100 ok
22 c ! c2f f @ . 71 ok
32 f ! f2c c @ . 0 ok
```

- `variable f` et `variable c` définissent des variables
- `32 constant b` définis une constant `b` égal au haut de la pile soit `32`
- 📚 `c` ajoute l'addresse de la variable `c` à la pile
- 📚 `c @` ajoute la valeur de la variable `c`  à la pile 
- 📚 `!` enregistre la valeur précédante à l'adresse en haut de la pile

__Incrémenter une variable__

```forth
( forme standart)
x @ 1+ x !
( forme racourcie )
1 x +!
```
## Allouer de la mémoire 

| Word   | Effect     | Description                             |
| :----: | :--------: | :-------------------------------------: |
| @      | ( a -- a ) | Get the 64-bit integer at address a     |
| !      | ( n a -- ) | Store a 64-bit integer in address a     |
| ,      | ( n -- )   | Compile an integer into the dictionary  |
| c@     | ( a -- b ) | Get the byte at address a               |
| c!     | ( b a -- ) | Store a byte at address a               |
| c,     | ( b -- )   | Compile a byte to the dictionary        |
| create | ( -- )     | Create a new word                       |
| does>  | ( -- a )   | Define the word's behavior              |
| allot  | ( b -- )   | Allocate dictionary space (bytes)       |
| cells  | ( n -- b ) | Convert cells to bytes                  |

__Allouer de l'espace mémoire à un mot du dictionnaire de forth__

```forth
( Crée un mot nommé buf contenant 1000 bytes d'espace mémoire )
create buf 1000 allot ok
( Ajoute 1 au premier byte de buf )
1 buf c! ok
( Ajoute 2 au deuxième byte de buf )
2 buf 1+ c! ok
( Ajoute 3 au troisième byte de buf )
3 buf 2 + c! ok
( Affiche la valeur du premier byte de buf )
buf c@ . 1 ok
( Affiche la valeur du deuxième byte de buf )
buf 1+ c@ . 2 ok
( Affiche la valeur du troisième byte de buf )
buf 2 + c@ . 3 ok
( Stocke une valeur de 64-bit dans le 4ème byte de buf )
123456789 buf 3 + ! ok
( Affiche la valeur du quatrième byte de buf )
buf 3 + @ . 123456789 ok
```

__Créer un tableau de bytes__

```forth
: bArray ( n -- ) create allot does> + ; ok
100 bArray x ok
123 0 x c! ok
124 1 x c! ok
1 x c@ . 124 ok
0 x c@ . 123 ok
2 x c@ . 125 ok
```

La première ligne définis `bArray` qui attend comme paramètre un nombre sur la pile, correspondant au nombre de bytes à allouer a ce mot.

le mot clé `does>` est appelé a chaque fois que `bArray` est référencé. C'est lui qui définis les étapes suivantes.

Lorsque le mot `bArray` est exécuté, il va rajouter son addrese mémoire en hau t de la pile. puis `does>` prend la relève.

Le `+` défnis dans `does>` sigifie que l'adresse mémoire de `bArray` présent dans la pile et la valeur précédante dans la pile vont être additioné pour formé une nouvelle addresse qui sera poussé sur la pile.

Ainsi `123 0 x c!` signifie que `123` va être poussé dans la pile puis `0`, puis `x` pousse son addresse mémoire dans la pile. C'est alors `does>` qui est éxécuté et qui va consommer l'addresse de x ainsi que l'index `0` pour faire une addition. le résultat est poussé sur la pile et correspond à l'adresse da la cellule `0` du tableau `x`. C'est alors `c!` qui est exécuté pour enregistrer la valeur `123` dans la cellule `0` de `x`

Par conséquent la notation `125 2 x c!` est équivalent à `x[2] = 125` en javascript. de même que `2 x c@` est la forme équivalente de `console.log(x[2])` toujours en javascript 

__Créer un tableau d'entiers__

```forth
: array ( n -- ) create cells allot does> swap cells + ; ok
100 array y ok
111111 66 y ! ok
222222 67 y ! ok
333333 68 y ! ok
66 y @ . 111111 ok
68 y @ . 333333 ok
67 y @ . 222222 ok
```

L'unité de mémoire de base dans `forth` est appelé `cell`. Il s'agit de `64 bits` ou `8 bytes`. Ainsi pour stocker `100` valeur, `800 bytes` sont requis. 

Pour indexer un tableau, il nous faut une addresse de base multiplié par la taille de la cellule.

Le mot `cell` après `create` permet de calculer le nombre de bytes que `allot` doit réserver à la fin du dictionnaire.

```forth
( Affiche le nombre de bytes pour stocker 1 valeur dans une cellule )
1 cells . 8 ok
( Affiche le nombre de bytes pour stocker 100 valeurs dans des cellules )
100 cells . 800 ok
```

Lorsque l'on index le tableau, la partie après `does>`, de devons dabord convertir l'index depuis la cellule vers bytes et ajouter le résultat à l'addresse de base. C'est ce que fait `swap cells +`

Après ces changement, `array` fonctionne comme `bArray` mais il faut utiliser `!` et `@` pour `set` et `get` les valeurs du tableau

__Initialiser un tableau avec des valeurs__

```forth
create ABCDEF 65 c, 66 c, 67 c, 68 c, 69 c, 70 c,
```
Crée le mot `ABCDEF` et aloue 6 valeurs correspondant à la valuer `ASCII` des 6 première lettre de l'alphabet grâce à `c,` on peut faire la même chose pour stocker des entiers avec juste `,`

```forth
: one ." one" ; ok
: two ." two" ; ok
: three ." three" ; ok
create tbl ' one , ' two , ' three , ok
tbl 2 cells +  @ execute three ok
tbl 1 cells +  @ execute two ok
tbl @ execute one ok
```

Cette exemple introduit deux nouveau mots `'` et `execute`. le premier prend l'addresse d'execution du mot suivant et le place dans la pile. le mot `execute` execute le mot dont l'addresse se trouve au dessus de la pile. Ainsi `tbl` est un tableau de 3 `function pointers`

## Conclusion

En conclusion, j'ai beaucoup apprécier le language et son principe de de manipulaton de la `stackpile`, cela nous force à avoir une très bonne comprhéension de ce mécanisme et de la gestion de la mémoire. Cependant cela fait aussi de `forth` un language très difficile à lire et qui demande beaucoup de concentration. Ainsi je n'ai pas l'intention de me servir de `forth` dans mon quotidien mais l'exploration de ce language m'as permis de mieux appréhender les concept de programmation tels que la `stackpile` et l'addrésage mémoire.
