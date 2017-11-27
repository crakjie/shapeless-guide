## Faire son propre op (le patterne "lemma") {#sec:ops:penultimate}

Si l'on trouve un ensemble d'ops utile, on peut les
regrouper sous la forme d'une autre type classe ops.
C'est un exemple du pattern « lemma », un terme que nous avions
introduit dans la section [@sec:type-level-programming:summary].

Prenons comme exercise la création de notre propre op.
Nous allons combiner la puissance de `Last` et de `Init` pour créer
la *type class* `Penultimate` qui retrouve l'avant-dernier élément d'une `HList`.
Voici la définition de la *type class*,
complétée par son `type alias` `Aux` et sa méthode `apply`:


```tut:book:silent
import shapeless._

trait Penultimate[L] {
  type Out
  def apply(l: L): Out
}

object Penultimate {
  type Aux[L, O] = Penultimate[L] { type Out = O }

  def apply[L](implicit p: Penultimate[L]): Aux[L, p.Out] = p
}
```

Notez encore une fois que la méthode `apply`
a comme type de retour `Aux[L, O]` au lieu de `Penultimate[L]`.
Ce qui garanti que le membre de type est visible dans les instances invoquées,
comme souligné dans la section [@sec:type-level-programming:depfun].

Nous n'avons besoin que de définir une seule instance de `Penultimate`,
qui combine `Init` et `Last` de la même façon
que dans la section [@sec:type-level-programming:chaining]:


```tut:book:silent
import shapeless.ops.hlist

implicit def hlistPenultimate[L <: HList, M <: HList, O](
  implicit
  init: hlist.Init.Aux[L, M],
  last: hlist.Last.Aux[M, O]
): Penultimate.Aux[L, O] =
  new Penultimate[L] {
    type Out = O
    def apply(l: L): O =
      last.apply(init.apply(l))
  }
```

Nous pouvons utiliser `Penultimate` de la façon suivante :

```tut:book:silent
type BigList = String :: Int :: Boolean :: Double :: HNil

val bigList: BigList = "foo" :: 123 :: true :: 456.0 :: HNil
```

```tut:book
Penultimate[BigList].apply(bigList)
```


Invoquer une instance de `Penultimate` oblige le compilateur à invoquer des instances pour `Last` et `Init`,
nous bénéficions donc du même niveau de vérification des types sur les `HList`s courtes :

```tut:book:silent
type TinyList = String :: HNil

val tinyList = "bar" :: HNil
```

```tut:book:fail
Penultimate[TinyList].apply(tinyList)
```

Nous pouvons rendre les chose plus simples pour les utilisateurs finaux si nous définissons
une méthode d'extension sur `HList`:

```tut:book:silent
implicit class PenultimateOps[A](a: A) {
  def penultimate(implicit inst: Penultimate[A]): inst.Out =
    inst.apply(a)
}
```

```tut:book
bigList.penultimate
```

Nous pouvons aussi fournir `Penultimate` pour tous les types produit
en fournissant une instance basée sur `Generic` :

```tut:book:silent
implicit def genericPenultimate[A, R, O](
  implicit
  generic: Generic.Aux[A, R],
  penultimate: Penultimate.Aux[R, O]
): Penultimate.Aux[A, O] =
  new Penultimate[A] {
    type Out = O
    def apply(a: A): O =
      penultimate.apply(generic.to(a))
  }

case class IceCream(name: String, numCherries: Int, inCone: Boolean)
```

```tut:book
IceCream("Sundae", 1, false).penultimate
```

Voici le point important à retenir :
en définissant `Penultimate` comme une autre type class,
nous avons créé un outil réutilisable que nous pouvons utiliser ailleurs.
Shapeless fournit de nombreux outils pour de nombreux usages,
mais il est simple d'ajouter le nôtre à la boîte à outils.
