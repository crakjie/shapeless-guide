## La déduction d'instance pour les type récursif.

```tut:book:invisible
// ----------------------------------------------
// Forward definitions

trait CsvEncoder[A] {
  def encode(value: A): List[String]
}

object CsvEncoder {
  def apply[A](implicit enc: CsvEncoder[A]): CsvEncoder[A] =
    enc
}

def writeCsv[A](values: List[A])(implicit encoder: CsvEncoder[A]): String =
  values.map(encoder.encode).map(_.mkString(",")).mkString("\n")

def createEncoder[A](func: A => List[String]): CsvEncoder[A] =
  new CsvEncoder[A] {
    def encode(value: A): List[String] =
      func(value)
  }

implicit val stringEncoder: CsvEncoder[String] =
  createEncoder(str => List(str))

implicit val intEncoder: CsvEncoder[Int] =
  createEncoder(num => List(num.toString))

implicit val booleanEncoder: CsvEncoder[Boolean] =
  createEncoder(bool => List(if(bool) "cone" else "glass"))

import shapeless.{HList, HNil, ::}

implicit val hnilEncoder: CsvEncoder[HNil] =
  createEncoder(hnil => Nil)

implicit def hlistEncoder[H, T <: HList](
  implicit
  hEncoder: CsvEncoder[H],
  tEncoder: CsvEncoder[T]
): CsvEncoder[H :: T] = createEncoder {
  case h :: t =>
    hEncoder.encode(h) ++ tEncoder.encode(t)
}

import shapeless.Generic

implicit def genericEncoder[A, R](
  implicit
  gen: Generic.Aux[A, R],
  rEncoder: CsvEncoder[R]
): CsvEncoder[A] =
  createEncoder(value => rEncoder.encode(gen.to(value)))

// ----------------------------------------------
```

Essayons quelque chose de plus ambitieu : un arbre binaire :

```tut:book:silent
sealed trait Tree[A]
case class Branch[A](left: Tree[A], right: Tree[A]) extends Tree[A]
case class Leaf[A](value: A) extends Tree[A]
```

En théorie nous devrions deja avoir toutes les définiton
pour instancier un CSV writer pour notre arbre.
Pourtant, appler `writeCsv` provoque une erreur :
However, calls to `writeCsv` fail to compile:

```tut:book:fail
CsvEncoder[Tree[Int]]
````

Le problème est que notre type est qu'il est récusif.
Le compilateur rentre dans une boucle infie
en essayant d'appliquer nos implicits
puis il abandonne.

### Les divergences d'implicits

La résolution d'implicit est un processus de recherche.
Le compilateur utilise des heuristiques pour
s'orienter vers la bonne solution.
Le compilateur effectue ses recherches branche par branche,
si l'une d'elle ne donne pas de résultat favorable,
le compilateur considère que cette branche n'aboutira pas
et continue ses recherches sur une autre branche.

Une des heuristique est conçue spécifiquement pour
éviter les boucles infinies.
Si le compilateur rencontre le type cible
deux fois dans une branche de recherche;
il abandonne cette branche et passe a une autre.
On peut l'observer si on regarde l'expression `CsvEncoder[Tree[Int]]`.
Le processus de résolution d'implicits suit les étapes suivantes :

```scala
CsvEncoder[Tree[Int]]                          // 1
CsvEncoder[Branch[Int] :+: Leaf[Int] :+: CNil] // 2
CsvEncoder[Branch[Int]]                        // 3
CsvEncoder[Tree[Int] :: Tree[Int] :: HNil]     // 4
CsvEncoder[Tree[Int]]                          // 5 uh oh
```

On rencontre `Tree[A]` deux fois lignes 1 et 5,
donc le compilateur abandonne la recherche dans cette branche.
La conséquence est que le compilateur echoue à trouver le bon *implicit*.

En fait, la situation est pire.
Si le compilateur voit le même constructeur de types deux fois
et que la complexité du paramètre de type *augmente* il suppose
que la branche n'aboutira pas.*
C'est un problème avec shapeless car les types comme
`::[H, T]` et `:+:[H, T]` peuvent apparaître plusieurs fois
lorsque le compilateur développe les représentations génériques.
Ce qui pousse le compilateur a abandonner prématurément
même si il aurait pu trouver la solution si il avait persisté
à chercher dans cette voie.

Compte-tenu des types suivants :

```tut:book:silent
case class Bar(baz: Int, qux: String)
case class Foo(bar: Bar)
```

Le déroulement de la recherche pour `Foo`
ressemble à ça :

```scala
CsvEncoder[Foo]                   // 1
CsvEncoder[Bar :: HNil]           // 2
CsvEncoder[Bar]                   // 3
CsvEncoder[Int :: String :: HNil] // 4 uh oh
```

Le compilateur essait de résoudre `CsvEncoder[::[H, T]]` deux fois
dans cette branche, une fois a la ligne 2 et une fois a la ligne 4.
Le paramètre de type `T` est plus complexe à la ligne 4 qu'à la ligne 2,
donc le compilateur pense (de façon erronée dans ce cas)
que la recherche dans cette branche n'aboutira pas.
Il change de branche encore et encore,
il ne trouvera pas de quoi générer la bonne instance.

### *Lazy*

La divergence d'implicit peut marquer un arrêt pour la bibliotèque shapeless.
Heureusement, shapeless fournit un
type appeler `Lazy` pour contourner ce problème.
`Lazy` fait deux choses :

 1. Il supprime la divergence d'implicit à la
    compilation en se protégeant des heuristiques trop défensives.


 2. Il reporte l'évaluation des paramètres implicites au runtime,
    ce qui permet la déduction des implicits récursifs.

On utilise `Lazy` en le paramétrant avec nos paramètres implicites.
En règle générale, il est toujours bon de mettre dans `Lazy`
dans le parametre de "tête" de toute règle de `HList` ou de `Coproduct`
ainsi que n'importe quel paramètre `Repr` d'un `Generic`:

```tut:book:invisible:reset
// Forward definitions -------------------------
import shapeless._

trait CsvEncoder[A] {
  def encode(value: A): List[String]
}

object CsvEncoder {
  def apply[A](implicit enc: CsvEncoder[A]): CsvEncoder[A] =
    enc
}

def createEncoder[A](func: A => List[String]): CsvEncoder[A] =
  new CsvEncoder[A] {
    def encode(value: A): List[String] =
      func(value)
  }

implicit val intEncoder: CsvEncoder[Int] =
  createEncoder(num => List(num.toString))

implicit val hnilEncoder: CsvEncoder[HNil] =
  createEncoder(hnil => Nil)

implicit val cnilEncoder: CsvEncoder[CNil] =
  createEncoder(cnil => throw new Exception("Inconceivable!"))

// ----------------------------------------------
```

```tut:book:silent
implicit def hlistEncoder[H, T <: HList](
  implicit
  hEncoder: Lazy[CsvEncoder[H]], // wrap in Lazy
  tEncoder: CsvEncoder[T]
): CsvEncoder[H :: T] = createEncoder {
  case h :: t =>
    hEncoder.value.encode(h) ++ tEncoder.encode(t)
}
```

```tut:book:silent
implicit def coproductEncoder[H, T <: Coproduct](
  implicit
  hEncoder: Lazy[CsvEncoder[H]], // wrap in Lazy
  tEncoder: CsvEncoder[T]
): CsvEncoder[H :+: T] = createEncoder {
  case Inl(h) => hEncoder.value.encode(h)
  case Inr(t) => tEncoder.encode(t)
}
```

```tut:book:silent
implicit def genericEncoder[A, R](
  implicit
  gen: Generic.Aux[A, R],
  rEncoder: Lazy[CsvEncoder[R]] // wrap in Lazy
): CsvEncoder[A] = createEncoder { value =>
  rEncoder.value.encode(gen.to(value))
}
```

```tut:book:invisible
sealed trait Tree[A]
final case class Branch[A](left: Tree[A], right: Tree[A]) extends Tree[A]
final case class Leaf[A](value: A) extends Tree[A]
```
Ceci permet au compilateur de ne pas jeter l'éponge prématurément
et donc permet aux types complex/récursif comme `Tree` de fonctionner :

```tut:book
CsvEncoder[Tree[Int]]
```
