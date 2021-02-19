##Étude de cas: générateur de valeur aléatoire


Les bibliothèques de test axées sur les propriétés comme [ScalaCheck][link-scalacheck]
utilisent des *type classes* dont le but est de générer des valeurs aléatoires pour les tests unitaires.
Par exemple, ScalaCheck fournit la *type class* `Arbitrary` que nous pouvons utiliser de la façon suivante :

```tut:book:silent
import org.scalacheck._
```

```tut:book
for(i <- 1 to 3) println(Arbitrary.arbitrary[Int].sample)
for(i <- 1 to 3) println(Arbitrary.arbitrary[(Boolean, Byte)].sample)
```

ScalaCheck fournit déjà des instances d'`Arbitrary`
pour un grand nombre de types scala standards.
Pourtant, créer des instances d'`Arbitrary` pour les utilisateurs d'*ADT*s
reste un travail manuel et chronophage.
C'est ce qui rend très intéressant l'intégration de shapeless
via des bibliothèques comme [scalacheck-shapeless][link-scalacheck-shapeless].

Dans cette section, nous allons créer une simple *type class* `Random`
pour générer des valeurs aléatoires pour un *ADT* donné.
Nous allons voir comment `Length` et `Nat` font partie intégrante de l'implémentation.
Comme toujours, nous commençons par définir la *type class* elle-même.

```tut:book:invisible
// Assure que nous avons toujours la même sortie :
scala.util.Random.setSeed(0)
```

```tut:book:silent
trait Random[A] {
  def get: A
}

def random[A](implicit r: Random[A]): A = r.get
```

### De simples valeurs aléatoires

Commençons avec une instance basique de `Random` :

```tut:book:silent
// Instance constructor:
def createRandom[A](func: () => A): Random[A] =
  new Random[A] {
    def get = func()
  }

// Random numbers from 0 to 9:
implicit val intRandom: Random[Int] =
  createRandom(() => scala.util.Random.nextInt(10))

// Random characters from A to Z:
implicit val charRandom: Random[Char] =
  createRandom(() => ('A'.toInt + scala.util.Random.nextInt(26)).toChar)

// Random booleans:
implicit val booleanRandom: Random[Boolean] =
  createRandom(() => scala.util.Random.nextBoolean)
```

Nous pouvons utiliser ces générateurs simples via la methode `random` :

```tut:book
for(i <- 1 to 3) println(random[Int])
for(i <- 1 to 3) println(random[Char])
```

### Produits aléatoires

Nous pouvons créer des valeurs aléatoires pour les produits en utilisant `Generic` et `HList` avec
les techniques vues dans le Chapitre [@sec:generic] :

```tut:book:silent
import shapeless._

implicit def genericRandom[A, R](
  implicit
  gen: Generic.Aux[A, R],
  random: Lazy[Random[R]]
): Random[A] =
  createRandom(() => gen.from(random.value.get))

implicit val hnilRandom: Random[HNil] =
  createRandom(() => HNil)

implicit def hlistRandom[H, T <: HList](
  implicit
  hRandom: Lazy[Random[H]],
  tRandom: Random[T]
): Random[H :: T] =
  createRandom(() => hRandom.value.get :: tRandom.get)
```

Ce qui nous conduit à invoquer des instances aléatoires pour les *case classes* :

```tut:book:silent
case class Cell(col: Char, row: Int)
```

```tut:book
for(i <- 1 to 5) println(random[Cell])
```

### Coproduits aléatoires

C'est ici que nous commençons à rencontrer des problèmes.
Générer une valeur aléatoire d'un coproduit implique de choisir aléatoirement un sous-type.
Commençons avec une implémentation naïve :

```tut:book:silent
implicit val cnilRandom: Random[CNil] =
  createRandom(() => throw new Exception("Inconceivable!"))

implicit def coproductRandom[H, T <: Coproduct](
  implicit
  hRandom: Lazy[Random[H]],
  tRandom: Random[T]
): Random[H :+: T] =
  createRandom { () =>
    val chooseH = scala.util.Random.nextDouble < 0.5
    if(chooseH) Inl(hRandom.value.get) else Inr(tRandom.get)
  }
```

Le problème de cette implémentation est qu'elle repose sur un choix binaire dans le calcul de `chooseH`.
Ce qui provoque des inégalités.
Par exemple, étudiez le type suivant :

```tut:book:silent
sealed trait Light
case object Red extends Light
case object Amber extends Light
case object Green extends Light
```

Le `Repr` de `Light` est `Red :+: Amber :+: Green :+: CNil`.
Pour ce type, une instance de `Random` choisira `Red` une fois sur deux et `Amber :+: Green :+: CNil`
une fois sur deux.
Alors qu'une distribution correcte
serait à 33 % `Red` et 67 % `Amber :+: Green :+: CNil`.

Et ce n'est pas tout.
Si nous regardons la distribution de probabilité globale nous constatons quelque-chose d'alarmant :

- `Red` est choisi une fois sur deux
- `Amber` est choisi une fois sur quatre
- `Green` est choisi une fois sur huit
- *`CNil` est choisi une fois sur seize*

Notre instance de coproduit lèvera une exception 6,75 % du temps !

```scala
for(i <- 1 to 100) random[Light]
// java.lang.Exception: Inconceivable!
//   ...
```

Pour résoudre ce problème, nous devons modifier la probabilité de choisir `H` par rapport a `T`
Il faudrait choisir `H` `1/n` du temps, où `n` est la taille du coproduit,
ce qui assurerait une distribution des probabilités égale entre les sous-types du coproduit.
Cela assure aussi de choisir toujours la tête d'un coproduit comportant un seul sous-type,
ce qui assure que `cnilProduct.get` n'est jamais choisi.
Voici notre implémentation mise à jour :

```tut:book:silent
import shapeless.ops.coproduct
import shapeless.ops.nat.ToInt

implicit def coproductRandom[H, T <: Coproduct, L <: Nat](
  implicit
  hRandom: Lazy[Random[H]],
  tRandom: Random[T],
  tLength: coproduct.Length.Aux[T, L],
  tLengthAsInt: ToInt[L]
): Random[H :+: T] = {
  createRandom { () =>
    val length = 1 + tLengthAsInt()
    val chooseH = scala.util.Random.nextDouble < (1.0 / length)
    if(chooseH) Inl(hRandom.value.get) else Inr(tRandom.get)
  }
}

```

Avec ces modifications, nous pouvons générer une valeur aléatoire pour n'importe quel coproduit :

```tut:book
for(i <- 1 to 5) println(random[Light])
```

Générer des données de test pour ScalaCheck nécessite normalement beaucoup de code *boilerplate*.
La génération de valeurs aléatoires est un exemple fascinant de l'utilisation de shapeless où les `Nat` ont une place essentielle.
