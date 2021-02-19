## La longueur des représentations génériques

Un des cas d'utilisation de `Nat`consiste à déterminer la taille
des `HList`s et des `Coproduct`s.
Shapeless fournit les *type classes* `shapeless.ops.hlist.Length` et
`shapeless.ops.coproduct.Length` pour cela :

```tut:book:silent
import shapeless._
import shapeless.ops.{hlist, coproduct, nat}
```

```tut:book
val hlistLength = hlist.Length[String :: Int :: Boolean :: HNil]
val coproductLength = coproduct.Length[Double :+: Char :+: CNil]
```

Les instances de `Length` ont un membre de type `Out` qui représente la taille avec un `Nat` :

```tut:book
Nat.toInt[hlistLength.Out]
Nat.toInt[coproductLength.Out]
```

Utilisons un exemple concret.
Nous allons créer une *type class* `SizeOf` qui
compte le nombre de champs d'une case class et 
retourne cette valeur comme un simple `Int` :

```tut:book:silent
trait SizeOf[A] {
  def value: Int
}

def sizeOf[A](implicit size: SizeOf[A]): Int = size.value
```

Pour créer une instance de `SizeOf`, nous avons besoin de trois choses :

1. un `Generic` pour calculer la `HList` correspondant au type ;
2. une `Length` pour calculer la taille de la `HList` pour obtenir un `Nat` ;
3. un `ToInt` pour convertir le `Nat` en `Int`.

Voici une implémentation qui marche, elle est écrite comme décrit dans le Chapitre [@sec:type-level-programming] :

```tut:book:silent
implicit def genericSizeOf[A, L <: HList, N <: Nat](
  implicit
  generic: Generic.Aux[A, L],
  size: hlist.Length.Aux[L, N],
  sizeToInt: nat.ToInt[N]
): SizeOf[A] =
  new SizeOf[A] {
    val value = sizeToInt.apply()
  }
```

Nous pouvons tester notre code de la façon suivante :

```tut:book:silent
case class IceCream(name: String, numCherries: Int, inCone: Boolean)
```

```tut:book
sizeOf[IceCream]
```
