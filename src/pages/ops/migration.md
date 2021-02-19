## Étude de cas : migration de case class {#sec:ops:migration}

La puissance des *type classes* ops se concrétise
quand on assemble dans notre propre code.
Nous allons conclure ce chapitre avec un exemple probant :
une type classe visant à effectuer des « migrations » (ou « évolutions »)
de *case classes*[^database-migrations].
Par exemple, si la version 1 de notre application contient la case classe suivante :

[^database-migrations]: Le terme est tiré de
« migrations de base de donnée » :
des scripts SQL qui automatise la
mise à jour des schémas de base de données.


```tut:book:silent
case class IceCreamV1(name: String, numCherries: Int, inCone: Boolean)
```

Notre bibliothèque de migration doit pouvoir
faire certaines mises à jour « mécanique » pour nous :


```tut:book:silent
// Remove fields:
case class IceCreamV2a(name: String, inCone: Boolean)

// Reorder fields:
case class IceCreamV2b(name: String, inCone: Boolean, numCherries: Int)

// Insert fields (provided we can determine a default value):
case class IceCreamV2c(
  name: String, inCone: Boolean, numCherries: Int, numWaffles: Int)
```

Idéalement on aimerait être capables d'écrire le code suivant :

```scala
IceCreamV1("Sundae", 1, false).migrateTo[IceCreamV2a]
```

La *type class* doit prendre soin de la migration sans code *boilerplate* supplémentaire.

### La *type class*

La *type class* `Migration` représente la transformation d'une source vers un type destination.
Ils seront tous les deux les paramètres de type pour notre déduction.
Nous n'avons pas besoin d'alias de type `Aux`
car il n'y a pas de membre de type à exposer :

```tut:book:silent
trait Migration[A, B] {
  def apply(a: A): B
}
```
Nous allons également ajouter une méthode d'extension
pour rendre les exemples plus simples à lire :

```tut:book:silent
implicit class MigrationOps[A](a: A) {
  def migrateTo[B](implicit migration: Migration[A, B]): B =
    migration.apply(a)
}
```

### Étape 1. Enlever les champs

Construison notre solution pièce par pièce,
en commençant par la suppression de champs.
Nous pouvons y arriver en plusieurs étapes :

 1. convertir `A` dans sa représentation générique ;
 2. filter la `HList` de l'étape 1, garder uniquement les champs qui sont présents dans `A` et `B` ;
 3. convertir le résultat de l'étape 2 en `B`.

Nous pouvons implémenter l'étape 1 et 3 avec `Generic` ou `LabelledGeneric`,
l'étape 2 peut l'être avec un op appelé `Intersection`.
`LabelledGeneric` semble être un choix intéressant
car nous avons besoin d'identifier le nom des champs :


```tut:book:silent
import shapeless._
import shapeless.ops.hlist

implicit def genericMigration[A, B, ARepr <: HList, BRepr <: HList](
  implicit
  aGen  : LabelledGeneric.Aux[A, ARepr],
  bGen  : LabelledGeneric.Aux[B, BRepr],
  inter : hlist.Intersection.Aux[ARepr, BRepr, BRepr]
): Migration[A, B] = new Migration[A, B] {
  def apply(a: A): B =
    bGen.from(inter.apply(aGen.to(a)))
}
```

Prenez un moment pour localiser [`Intersection`][code-ops-hlist-intersection]
dans la base de code de shapeless.
L'alias de type `Aux` prend en compte trois paramètres :
deux `HList`s qui sont les entrées et une pour le type du résultat de l'intersection.
Dans l'exemple au-dessus nous spécifions `ARepr` et `BRepr` comme type d'entrée et `BRepr` comme type de retour.
Ce qui signifie que la résolution d'implicite ne fonctionnera que si `B` contient l'exact sous-ensemble des champs de `A`,
avec exactement les mêmes noms et dans le même ordre :

```tut:book
IceCreamV1("Sundae", 1, true).migrateTo[IceCreamV2a]
```

Nous obtenons une erreur de compilation si nous
essayons d'utiliser `Migration` avec un type non conforme :

```tut:book:fail
IceCreamV1("Sundae", 1, true).migrateTo[IceCreamV2b]
```

### Etape 2. Réordonner les champs

Nous avons besoin d'une autre ops type class
pour pouvoir faire du réordonnement.
L'op [`Align`][code-ops-hlist-align] nous permet de réordonner les champs d'une `HList`
pour faire correspondre l'ordre d'une autre `HList`.
Nous pouvons redéfinir notre instance en utilisant `Align` de la manière suivante :

```tut:book:silent
implicit def genericMigration[
  A, B,
  ARepr <: HList, BRepr <: HList,
  Unaligned <: HList
](
  implicit
  aGen    : LabelledGeneric.Aux[A, ARepr],
  bGen    : LabelledGeneric.Aux[B, BRepr],
  inter   : hlist.Intersection.Aux[ARepr, BRepr, Unaligned],
  align   : hlist.Align[Unaligned, BRepr]
): Migration[A, B] = new Migration[A, B] {
  def apply(a: A): B =
    bGen.from(align.apply(inter.apply(aGen.to(a))))
}
```
Nous ajoutons un nouveau paramètre de type appelé `Unaligned`
pour représenter l'intersection de `ARepr` et `BRepr` avant l'alignement,
on utilise `Align` pour convertir `Unaligned` en `BRepr`.
Avec cette modification de `Migration` nous pouvons à
la fois retirer mais aussi réordonner les champs :

```tut:book
IceCreamV1("Sundae", 1, true).migrateTo[IceCreamV2a]
IceCreamV1("Sundae", 1, true).migrateTo[IceCreamV2b]

```

Cependant, si on essaie d'ajouter un champ, on obtient encore une erreur :

```tut:book:fail
IceCreamV1("Sundae", 1, true).migrateTo[IceCreamV2c]
```

### Etape 3. Ajouter de nouveaux champs

Pour soutenir l'ajout de champs nous avons
besoin d'un mécanisme pour fournir les valeurs par défaut.
Shapeless ne fournit pas une *type class* pour cette raison,
mais Cats le fait, sous la forme d'un `Monoid`.
En voici une définition simplifiée :

```scala
package cats

trait Monoid[A] {
  def empty: A
  def combine(x: A, y: A): A
}
```

Un `Monoid` dispose de deux opérations :
`empty` pour créer une valeur « zéro »
et `combine` pour « fusionner » deux valeurs en une seule.
Dans notre code nous n'avons besoin que de `empty`
mais il serait trivial de définir également`combine`.

Cats fournit une instance de `Monoid` pour chacun des type
primitf qui nous intéresse (`Int`, `Double`, `Boolean`, and `String`).
Nous pouvons définir une instance pour `HNil`et `::`
en utilisant les techniques du Chapitre [@sec:labelled-generic]:

```tut:book:silent
import cats.Monoid
import cats.instances.all._
import shapeless.labelled.{field, FieldType}

def createMonoid[A](zero: A)(add: (A, A) => A): Monoid[A] =
  new Monoid[A] {
    def empty = zero
    def combine(x: A, y: A): A = add(x, y)
  }

implicit val hnilMonoid: Monoid[HNil] =
  createMonoid[HNil](HNil)((x, y) => HNil)

implicit def emptyHList[K <: Symbol, H, T <: HList](
  implicit
  hMonoid: Lazy[Monoid[H]],
  tMonoid: Monoid[T]
): Monoid[FieldType[K, H] :: T] =
  createMonoid(field[K](hMonoid.value.empty) :: tMonoid.empty) {
    (x, y) =>
      field[K](hMonoid.value.combine(x.head, y.head)) ::
        tMonoid.combine(x.tail, y.tail)
  }
```

Nous devons combiner `Monoid`[^monoid-pun] avec quelques
autre ops pour compléter notre implémentation finale de  `Migration`.
Voici la liste complète des étapes :

 1. utiliser `LabelledGeneric` pour convertir `A` dans sa représentation générique ;
 2. utiliser `Intersection` pour calculer `HList` des champs communs entre `A` et `B` ;
 3. calculer les types qui apparaissent dans `B` mais pas dans `A` ;
 4. utiliser `Monoid` pour calculer une valeur par défaut au type de l'étape 3 ;
 5. ajouter les champs de l'étape 2 aux nouveaux champs de l'étape 4 ;
 6. utiliser `Align` pour réordonner les champs de l'étape 5 dans le même ordre que `B` ;
 7. utiliser `LabelledGeneric` pour convertir le résultat de l'étape 6 vers `B`.

[^monoid-pun]: Le jeu de mots est intentionnel.
(note du traducteur: jeu de mot intraduisible)

Nous avons déja vu comment implémenter les étapes 1, 2, 4, 6 et 7.
Nous pouvons implémenter l'étape 3 en utilisant un op appelé `Diff`
qui est très similaire à `Intersection`,
et nous pouvons implémenter l'étape 5 en utilisant un autre op appélé `Prepend`.
Voici la solution complète :

```tut:book:silent
implicit def genericMigration[
  A, B, ARepr <: HList, BRepr <: HList,
  Common <: HList, Added <: HList, Unaligned <: HList
](
  implicit
  aGen    : LabelledGeneric.Aux[A, ARepr],
  bGen    : LabelledGeneric.Aux[B, BRepr],
  inter   : hlist.Intersection.Aux[ARepr, BRepr, Common],
  diff    : hlist.Diff.Aux[BRepr, Common, Added],
  monoid  : Monoid[Added],
  prepend : hlist.Prepend.Aux[Added, Common, Unaligned],
  align   : hlist.Align[Unaligned, BRepr]
): Migration[A, B] =
  new Migration[A, B] {
    def apply(a: A): B =
      bGen.from(align(prepend(monoid.empty, inter(aGen.to(a)))))
  }
```

Notez que ce code n'évalue pas toutes les *type classes*.
Nous utilisons `Diff` uniquement pour calculer le type de données `Added`,
mais nous n'avons pas besoin d'exécuter `diff.apply`.
Au lieu de cela, nous utilisons notre `Monoid` pour invoquer une instance de `Added`.

Avec cette dernière version de la *type class* nous pouvons utiliser
`Migration` pour tous les cas d'utilisation que nous avions données au
début de cette étude de cas :

```tut:book
IceCreamV1("Sundae", 1, true).migrateTo[IceCreamV2a]
IceCreamV1("Sundae", 1, true).migrateTo[IceCreamV2b]
IceCreamV1("Sundae", 1, true).migrateTo[IceCreamV2c]
```

C'est incroyable tout ce que nous pouvons faire avec les *type class* ops.
`Migration` ne contient qu'une `implicit def` avec une seul ligne d'implémentation
au value-level.
Cela nous permet d'automatiser les migrations entre *n'importe* quelle paire de *case classes*,
avec approximativement la même quantité de code
nécessiare pour gérer une *seule* paire
de types en utilisant une bibliothèque standard.
C'est la grande puissance de shapeless !
