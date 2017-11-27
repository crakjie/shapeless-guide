## Exemple d'ops simples

`HList` dispose d'une méthode d'extension `init`
et d'un autre `last` basé sur deux type classes :
`shapeless.ops.hlist.Init` et
`shapeless.ops.hlist.Last`.
`Coproduct` a des méthodes et des type classes similaires.
Ils représentent un exemple parfait du pattern ops.
Voici une définition simplifiée des méthodes d'extension :

```scala
package shapeless
package syntax

implicit class HListOps[L <: HList](l : L) {
  def last(implicit last: Last[L]): last.Out = last.apply(l)
  def init(implicit init: Init[L]): init.Out = init.apply(l)
}
```

Le type de retour de chaque méthode est déterminé par le type dépendant du paramètre implicite.
Ce sont les instances de chaque type class qui fournissent la relation.
Voici le squelette de l'implémentation de `Last` en guise d'exemple :

```scala
trait Last[L <: HList] {
  type Out
  def apply(in: L): Out
}

object Last {
  type Aux[L <: HList, O] = Last[L] { type Out = O }
  implicit def pair[H]: Aux[H :: HNil, H] = ???
  implicit def list[H, T <: HList]
    (implicit last: Last[T]): Aux[H :: T, last.Out] = ???
}
```

Nous pouvons faire quelques observations intéressantes sur cette implémentation.
Premièrement, nous pouvons généralement implémenter les type classes
ops avec un petit nombre d'instances (seulement deux dans cet exemple).
Nous pouvons ainsi rassembler *toutes* les instances requises dans l'objet compagnon de la *type class*,
ce qui nous permet d'appeler les methodes d'extention de `shapeless.ops` qui n'ont aucun rapport :

```tut:book:silent
import shapeless._
```

```tut:book
("Hello" :: 123 :: true :: HNil).last
("Hello" :: 123 :: true :: HNil).init
```

Ensuite, la *type class* n'est définie que pour les `HList`s qui contiennent au moins un élément.
Ce qui nous donne quelques vérifications statiques.
Si on essaie d'appeler `last` sur une `HList` vide,
cela provoque une erreur de compilation :


```tut:book:fail
HNil.last
```
