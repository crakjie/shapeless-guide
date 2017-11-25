## Types littéreaux.

Une valeur de Scala peut avoir plusieurs types.
Par exemple, la string `"hello"` a au moins trois types :
`String`, `AnyRef` et `Any`[^multiple-inheritance] :

```tut:book
"hello" : String
"hello" : AnyRef
"hello" : Any
```

[^multiple-inheritance]:
`String` est également doté de pluisieurs autres types tels que
`Serializable` et `Comparable`, mais ignorons-les pour l'instant.

`"hello"` a aussi un autre type :
un  "type singleton" qui appartient exclusivement à cette valeur.
Le résultat est similaire à ce que l'on obtient
lorsqu'on définit un object compagnon.

```tut:book:silent
object Foo
```

```tut:book
Foo
```

Le type `Foo.type` est le type de `Foo`,
et `Foo` est la seule et unique valeur pour ce type.

Les types singleton appliqués aux valeurs littérales sont appelés *types littéraux*.
Ils existent depuis longtemps dans Scala,
mais il n'y a normalement pas d'interaction avec eux, car le compilateur "élargit" par défaut les littéraux au type non singleton le plus proche.
Par exemple, ces deux expressions sont essentiellement équivalentes :


```tut:book
"hello"

("hello" : String)
```

Shapeless fournit quelques outils pour travailler avec les types littéraux.
Tout d'abord, la macro `narrow` convertit
une expression littérale en une expression littérale typée par un singleton :


```tut:book:silent
import shapeless.syntax.singleton._
```

```tut:book
var x = 42.narrow
```

Notez que le type de `x`: `Int(42)` est un type littéral.
Il s'agit d'un sous-type de `Int` qui contient uniquement la valeur `42`.
Si l'on essaie d'assigner une valeur différente à `x`,
on obtient une erreur de compilation :

```tut:book:fail
x = 43
```

Pourtant, `x` est toujours un sous type normal de `Int`.
Si l'on fait des opérations sur `x`, on obtient bien un type normal pour résultat :

```tut:book
x + 1
```

Nous pouvons utiliser `narrow` avec n'importe quel type littéral Scala :

```tut:book
1.narrow
true.narrow
"hello".narrow
// and so on...
```

Malheureusement, on ne peut pas les utiliser dans une expression composée :

```tut:book:fail
math.sqrt(4).narrow
```

<div class="callout callout-info">
*Types littéraux en Scala*

Jusque récemment, Scala ne disposait pas de syntaxe pour écrire les types littéraux.
Les types étaient là, dans le compilateur, mais nous ne pouvions pas les écrire directement dans le code.
Mais grâce aux versions de Lightbend Scala 2.12.1, Lightbend Scala 2.11.9,
et Typelevel Scala 2.11.8, nous disposons désormais d'un support direct de la syntaxe pour les types littéraux.
Dans ces versions de Scala, nous pouvons écrire les déclarations de la façon suivante :

```scala
val theAnswer: 42 = 42
```
Le type `42` est le même que le type `Int(42)` que nous avons vu précédemment.
Vous verrez toujours `Int(42)` dans les sorties pour des raisons de pérennité,
mais la syntaxe canonique restera `42`.

</div>

## Le type tagging et les types fantômes {#sec:labelled-generic:type-tagging}

Shapeless utilise les types littéraux pour modéliser les noms de champs des *case classes*.
Pour ce faire, il "tag" les types des champs avec le type littéral de leurs noms.
Avant de découvrir comment shapeless réalise cela, faisons-le nous-même pour montrer que la magie n'y est pour rien
(enfin... pas vraiment).
Supposons un nombre :

```tut:book:silent
val number = 42
```
Ce nombre est un `Int` dans deux mondes :
à l'execution où il a réellement une valeur
et des méthodes que l'on peut appeler,
et à la compilation, où le compilateur utilise
le type pour calculer quelle partie
de code fonctionne ensemble
et l'utilise pour rechercher les implicits.

On peut modifier le type du `number` à la compilation
sans modifier ces valeurs
à l'execution en le "taggant" avec un "type fantôme".
Les types fantômes sont des types qui
ne présentent pas de sémantique à l'exécution,
comme ceci :

```tut:book:silent
trait Cherries
```

Nous pouvons tagger le `number` en utilisant `asInstanceOf`.
Nous finissons avec une valeur qui est
à la fois un `Int` et une `Cherries` à la compilation
et uniquement un `Int` à l'execution :

```tut:book
val numCherries = number.asInstanceOf[Int with Cherries]
```

Shapeless utilise cette astuce pour tagger les champs et les sous-types d'un *ADT*
avec le type singleton de leurs noms.
Si l'utilisation de `asInstanceOf`
vous dérange, ne vous inquiétez pas :
Shapeless fournit deux syntaxes de
tagging qui vous éviteront ce désagrément.


La première syntaxe :  `->>`
tag l'expression à droite de la flèche avec
le type singleton de l'expression littérale à droite de la flèche.


```tut:book:silent
import shapeless.labelled.{KeyTag, FieldType}
import shapeless.syntax.singleton._

val someNumber = 123
```

```tut:book
val numCherries = "numCherries" ->> someNumber
```

Voila comment nous taggons `someNumber`
avec le type fantôme suivant :


```scala
KeyTag["numCherries", Int]
```

Le tag détaille à la fois le nom et le type du champ ;
la combinaison des deux est utile lorsque
l'on recherche des éléments dans un `Repr` en
utilisant la résolution d'`implicit`.

La seconde syntaxe considère le tag comme un type plutôt qu'une valeur littérale.
C'est utile lorsque vous connaissez le tag à utiliser
mais que vous n'avez pas la possibilité
d'écrire ce type littéral dans notre code :

```tut:book:silent
import shapeless.labelled.field
```

```tut:book
field[Cherries](123)
```

`FieldType` est un alias de type qui simplifie
l'extraction du tag et du type de base du type tagger :

```scala
type FieldType[K, V] = V with KeyTag[K, V]
```

Comme nous le verrons plus tard,
Shapeless emploie ce mécanisme pour tagger
les champs et les sous-types avec
leurs noms dans notre code source.


Les tags n'existent qu'à la compilation
et n'ont pas de représentation à l'exécution.
Alors, comment pouvons-nous les convertir en valeurs utilisables à l'exécution ?
Shapeless fournit à cette fin une type class appelée `Witness`[^witness].
Si l'on combine `Witness` et `FieldType`,
on obtient quelque chose de vraiment intéressant :
la possibilité d'extraire le nom d'un champ à partir d'un champ tagger :

[^witness] : le terme "witness" est emprunté aux [preuves mathématiques][link-witness]


```tut:book:silent
import shapeless.Witness
```

```tut:book
val numCherries = "numCherries" ->> 123
```

```tut:book:silent
// Get the tag from a tagged value:
def getFieldName[K, V](value: FieldType[K, V])
    (implicit witness: Witness.Aux[K]): K =
  witness.value
```

```tut:book
getFieldName(numCherries)
```

```tut:book:silent
// Get the untagged type of a tagged value:
def getFieldValue[K, V](value: FieldType[K, V]): V =
  value
```

```tut:book
getFieldValue(numCherries)
```

Si nous construisons une `HList` d'éléments taggés,
nous obtenons une structure de données dotées de certaines propriétés d'une `Map`.
Nous pouvons faire référénce à un champ par un tag, le manipuler et le remplacer,
et conserver toutes les information de types et de noms tout du long.
Shapeless appelle ces structures des "records".

### Records et *LabelledGeneric*

Les records sont des `HList`s d'éléments taggés.

```tut:book:silent
import shapeless.{HList, ::, HNil}
```

```tut:book
val garfield = ("cat" ->> "Garfield") :: ("orange" ->> true) :: HNil
```

Pour être clair, le type de `garfield` est le suivant :

```scala
// FieldType["cat",    String]  ::
// FieldType["orange", Boolean] ::
// HNil
```

Nous n'avons pas besoin ici d'approfondir les records :
en bref, les records sont les représentations génériques utilisées par `LabelledGeneric`.
`LabelledGeneric` tag chaque champ ou nom de type d'un *ADT* dans des produits ou des coproduits
(bien que les noms soient représentés par des `Symbols` et non des `Strings`).
Shapeless fournit un ensemble d'operations semblables à celles de `Map` pour manipuler les records,
certaines seront traitées dans la section [@sec:ops:record].
Pour l'instant, contentons-nous de déduire quelquess type class à l'aide de  `LabelledGeneric`.
