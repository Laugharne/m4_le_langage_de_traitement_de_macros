# M4 le langage de traitement de macros

> Ce texte est la republication du même article issu de mon vieux blog,
> afin d'obtenir un peu plus de visibilité sur le l'usage du langage M4
> Lien : https://laugharne.me/post/13489092551/m4-le-langage-de-traitement-de-macros


Je ne reviendrai pas sur le rôle et l'usage des[ macro-définitions](https://secure.wikimedia.org/wikipedia/fr/wiki/Macro-d%C3%A9finition) plus communément appelées **macros**.

De même, attention aux pièges que peuvent être les macros ! Si on ne vérifie pas que certains paramètres ne sont pas le résultat d'expressions arithmétiques ou de traitements de caractères. Ayant pour résultats, effets de bords et dégradations des performances liées aux appels répétés par la macro. Un grand classique, mais qu'il convient de rappeler tant il est simple de se faire prendre !

**M4** est un langage de programmation[ complet](https://secure.wikimedia.org/wikipedia/fr/wiki/Turing-complet) au sens[ Turing](https://secure.wikimedia.org/wikipedia/fr/wiki/Machine_de_Turing) du terme apparu en 1977 développer par[ Kernighan](https://secure.wikimedia.org/wikipedia/fr/wiki/Brian_Kernighan) et[ Ritchie](https://secure.wikimedia.org/wikipedia/fr/wiki/Dennis_Ritchie).

Les exemples que vous trouverez ici, sont pour certains issus ou adaptés des sites que vous trouverez en lien au bas de l'article.

## Définition et utilisation :

Les macros ne nécessitent pas de caractères spéciaux ou de préfixes quels qu'ils soient et n'a pas été conçu pour le traitement d'un langage informatique en particulier. Pour utiliser M4 il suffit simplement de fournir le nom du (ou des) fichier(s) à utiliser en entrée en paramètre de l'appel. Celui-ci sera lu, l'expansion des macros effectuée et le résultat généré sur la sortie standard. Partout ou le nom de la macro (ou des macros) apparaîtra, celui-ci sera substitué par le texte prédéfini.

Avec le fichier “ *exemple.m4* ” suivant :

```php
define(`salut', `Salut tout le monde') salut, voici un exemple de script M4.
```

Si nous lançons la commande suivante :

```bash
m4 exemple.m4 > exemple.txt
```

Le fichier “ *exemple.txt* ” généré aura pour contenu :
```
Salut tout le monde, voici un exemple de script M4.
```
## Possibilités :

- remplacement de texte (comme dans l'exemple précédent)
- substitution de paramètre (des paramètres peuvent être passés à la macro)
- inclusion de fichier (comme en C ou en PHP)
- manipulation de chaînes de caractères (troncature, assemblage, substitution, extraction, expressions régulières, etc.)
- évaluation conditionnelle (branchements “if”, “else”, etc.)
- itérations (par des appels récursifs)
- expressions arithmétiques (simples)
- interfaçage avec le système (appel de commande système)
- diagnostics pour le programmeur (M4 est debuggable en lui-même et utile pour le debuggage des programmes)

## Difficultés :

Un des soucis que peut poser M4 est la confusion lors de l'interprétation des noms des macros.

Tout d'abord les noms de macros qui se trouvent dans les commentaires M4 sont ignorés, exemple :

```
# Dans un commentaire M4, je peux librement utiliser des noms de macros précédemment
# définies ou des noms d'instructions M4 comme define, divert ou ifelse
```

L’instruction **divert** permet d'éviter aussi ce genre de problèmes, mais contrairement aux noms contenus dans les commentaires, les macros sont effectivement exécutées, mais leurs résultats ne sont pas répercutés vers la sortie standard.

```php
divert(-1) <definitions...> divert(0)dnl
```

La dernière ligne, avec dnl empêche la ligne suivante d'être répercutée sur la sortie standard, elle aurait pu également être écrite sous cette forme

divert`'dnl

**dnl** signifie **d**elete to **n**ew **l**ine et élimine tout écho vers la sortie standard jusqu'à la fin de ligne comprise. Si un espace blanc se trouvait entre les deux instructions, celui-ci serait propagé vers la sortie standard.

Il existe une manière simple de résoudre le problème de la confusion entre le define de M4 et de certains mots-clef propre au langage traité via M4 (par exemple le **define** de **PHP**) Une alternative nous est proposée si nous spécifions l'option **-P** en ligne de commande ainsi toutes les instructions natives de M4 seront préfixées par “ *m4\_* ”, comme illustré ci-dessous.

```php
define(`M1',`text1')M1     # -> define(M1,text1)M1
m4_define(`M1',`text1')M1 # -> text1
```

**Exemples simples :**

Le code suivant est un exemple d'utilisation de **M4** en conjugaison avec du **HTML**. Il permet de créer automatiquement des sections numérotées :

```
divert(-1)


La commande `divert' permet d'ignorer ce texte. Notez cependant que j'ai dû mettre entre guillemets le mot `divert' pour qu'il ne soit pas pris en compte.

# Initialise le compteur à 1.
define(`H2_COUNT', 1)

# La macro H2_COUNT est redéfinie à chaque fois que la macro H2 est utilisée.

define(`H2', `<h2>H2_COUNT. $1</h2>define(`H2_COUNT', incr(H2_COUNT))')

divert(0)dnl Remise à 0 de divert. L’instruction dnl élimine cette ligne.

H2(Première Section)
H2(Seconde Section)
H2(Conclusion)
```
Ce qui donnera, une fois exécuté, le résultat suivant :

```

<h2>1. Première Section</h2>
<h2>2. Seconde Section</h2>
<h2>3. Conclusion</h2>
```

Les “guillemets” **`** et **’** permettent de supprimer l'expension des macros. Ainsi le code suivant :

```
define(AUTEUR, Nicolas Boileau)dnl

`AUTEUR' : AUTEUR
```

Donnera le résultat suivant :

```
AUTEUR : Nicolas Boileau
```

Il est possible de passer des définitions en ligne de commande en utilisant l'option **-D** : m4 -DAUTEUR="Nicolas Boileau“ -DYEAR=1636 fichier.m4

En ce qui concerne les branchements conditionels, nous avons à notre disposition les instructions **ifdef** et **ifelse**. Ainsi **ifdef(`a’,b)** sortira sur la sortie standard **b** si et seulement si le symbole **a** est défini; **ifdef(`a’,b,c)** sortira **c** si le symbole **a** n'est pas défini.

**ifelse(a,b,c,d)** compare les chaines de caractères **a** et **b**. Si la comparaison est exacte, la macro fournira **c**, sinon ce sera **d**.

Le mécanisme peut aussi être étendu à des structures *else-if* multiples :

```php
ifelse(a,b,c,d,e,f,g)
```

qui correspond à la construction suivante :

```php
ifelse(a,b,c,ifelse(d,e,f,g))
```

Il n'y a pas d'instructions natives pour procéder à des itérations avec M4. On doit pour cela créer des macros récursives pour obtenir le résultat voulu. Par exemple pour les boucles **for** :

```
define(`for',`ifelse($#,0,``$0'',`ifelse(eval($2<=$3),1,

`pushdef(`$1',$2)$4`'popdef(`$1')$0(`$1',incr($2),$3,`$4')')')')

for n = for(`x',1,5,`x,')... # -> for n = 1,2,3,4,5,...

for(`x',1,3,`for(`x',0,4,`eval(5-x)') ')

# -> 54321 54321 54321
```

Les boucles **foreach** :

```
define(`foreach',`ifelse(eval($#>2),1,

`pushdef(`$1',`$3')$2`'popdef(`$1')dnl `'ifelse(eval($#>3),1,`$0(`$1',`$2',shift(shift(shift($@))))')')')

foreach(`X',`Open the X. ',`door',`window')

# -> Open the door. Open the window.
```

Les boucles **while** :
```

define(`while',`ifelse($#,0,``$0'',eval($1+0),1,`$2`'$0($@)')')

define(`POW2',2) while(`POW2<=1000',`define(`POW2',eval(POW2*2))')
POW2 # -> 1024
```

Voilà, ainsi se clôt cette courte présentation de M4, bien trop courte tant les possibilités de M4 sont grandes, je ne saurai que trop vous conseiller d'approfondir le sujet en parcourant les liens ci-dessous. Bonne exploration !

**Liens :**

- [en] [Notes on the M4 Macro Language](http://mbreen.com/m4.html%20) (excellent pour débuter)
- [en] [The m4 Macro Package](http://www.linuxjournal.com/article/5594)
- [en] [m4 for Windows](http://gnuwin32.sourceforge.net/packages/m4.htm)
- [fr] [M4 (langage) - Wikipédia](https://secure.wikimedia.org/wikipedia/fr/wiki/M4_%28langage%29)
- [en] [m4 (computer language) - Wikipedia](https://secure.wikimedia.org/wikipedia/en/wiki/M4_%28computer_language%29)
