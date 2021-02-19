## Les autres opérations impliquant *Nat*

Shapeless fournit tout un ensemble d'autres opérationss basées sur `Nat`.
La méthode `apply` sur les `HList` ou les `Coproduct` peut accepter un `Nats`
comme paramètre ou comme paramètre de type :

```tut:book:silent
import shapeless._

val hlist = 123 :: "foo" :: true :: 'x' :: HNil
```

```tut:book
hlist.apply[Nat._1]
hlist.apply(Nat._3)
```

Il y a aussi des opérations comme `take`, `drop`, `slice` et `updatedAt` :

```tut:book
hlist.take(Nat._3).drop(Nat._1)
hlist.updatedAt(Nat._1, "bar").updatedAt(Nat._2, "baz")
```

Ces opérations et leurs *type classes* associées
sont utiles pour manipuler individuellement
les éléments d'un produit ou d'un coproduit.
