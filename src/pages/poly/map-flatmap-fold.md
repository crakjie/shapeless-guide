## Mapping et flatMapping à l'aide de *Poly*

Shapeless fournit un ensemble d'opérations fonctionnelles
basées sur `Poly`, chacune implémentée par un type classe ops.
Jetons un œil à `map` et `flatMap` en guise d'exemple.
voici `map` :

```tut:book:silent
import shapeless._

object sizeOf extends Poly1 {
  implicit val intCase: Case.Aux[Int, Int] =
    at(identity)

  implicit val stringCase: Case.Aux[String, Int] =
    at(_.length)

  implicit val booleanCase: Case.Aux[Boolean, Int] =
    at(bool => if(bool) 1 else 0)
}
```

```tut:book
(10 :: "hello" :: true :: HNil).map(sizeOf)
```

Notez que les éléments dans la `HList` résultante a des types qui correspondent
au `Case` dans `sizeOf`.
Nous pouvons utiliser `map` avec nimporte quel `Poly` qui contient
un `Case` pour chaque membre de notre `HList`.
Si le compilateur ne peut pas trouver de `Case` pour un membre,
nous obtenons une erreur de compilation :

```tut:book:fail
(1.5 :: HNil).map(sizeOf)
```

Nous pouvons aussi utiliser `flatMap` sur une `HList`,
tant que tout les cas de notre `Poly` retournent une autre `HList` :

```tut:book:silent
object valueAndSizeOf extends Poly1 {
  implicit val intCase: Case.Aux[Int, Int :: Int :: HNil] =
    at(num => num :: num :: HNil)

  implicit val stringCase: Case.Aux[String, String :: Int :: HNil] =
    at(str => str :: str.length :: HNil)

  implicit val booleanCase: Case.Aux[Boolean, Boolean :: Int :: HNil] =
    at(bool => bool :: (if(bool) 1 else 0) :: HNil)
}
```

```tut:book
(10 :: "hello" :: true :: HNil).flatMap(valueAndSizeOf)
```

Encore une fois, nous obtenons une erreur de compilation s'il manque des `Cases`
ou si un `Case` ne retourne pas de `HList` :

```tut:book:fail
// Using the wrong Poly with flatMap:
(10 :: "hello" :: true :: HNil).flatMap(sizeOf)
```

`map` et `flatMap` sont basés sur les *types classes*
`Mapper` et `FlatMapper`. Nous en verrons un exemple
d'utilisation de `Mapper` dans la Section  [@sec:poly:product-mapper].

## Utiliser Fold avec *Poly*

En plus de `map` et `flatMap`, shapeless fournit
`foldLeft` et `foldRight` qui sont basés sur `Poly2` :

```tut:book:silent
import shapeless._

object sum extends Poly2 {
  implicit val intIntCase: Case.Aux[Int, Int, Int] =
    at((a, b) => a + b)

  implicit val intStringCase: Case.Aux[Int, String, Int] =
    at((a, b) => a + b.length)
}
```

```tut:book
(10 :: "hello" :: 100 :: HNil).foldLeft(0)(sum)
```

Nous pouvons aussi utiliser `reduceLeft`, `reduceRight`, `foldMap` et bien d'autres.
Chaque opération est associée à une type classe.
Nous vous laissons le soin d'examiner les autres opérations disponibles.
