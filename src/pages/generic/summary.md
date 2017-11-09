## En résumé
Dans ce chapitre nous avons expliqué comment utiliser
`Generic`, `HList`s, et `Coproduct`s
pour déduire automatiquement l'instance d'une *type class*.
Nous avons couvert aussi le type `Lazy`
qui est un moyen de manipuler les types complexes/recursifs.
Compte tenu de tout ceci,
on peut écrire le squelette commun suivant
qui permet de déduire des instances de type class.

Premièrement, définissons la type class :

```tut:book:silent
trait MyTC[A]
```

Définissons les instances pour les types primitifs :

```tut:book:silent
implicit def intInstance: MyTC[Int] = ???
implicit def stringInstance: MyTC[String] = ???
implicit def booleanInstance: MyTC[Boolean] = ???
```

Définissons les instances pour `HList`:

```tut:book:silent
import shapeless._

implicit def hnilInstance: MyTC[HNil] = ???

implicit def hlistInstance[H, T <: HList](
  implicit
  hInstance: Lazy[MyTC[H]], // wrap in Lazy
  tInstance: MyTC[T]
): MyTC[H :: T] = ???
```
Si nécessaire, définissons les instances de `Coproduct`:

```tut:book:silent
implicit def cnilInstance: MyTC[CNil] = ???

implicit def coproductInstance[H, T <: Coproduct](
  implicit
  hInstance: Lazy[MyTC[H]], // wrap in Lazy
  tInstance: MyTC[T]
): MyTC[H :+: T] = ???
```

Enfin, définissons les instances pour `Generic`:

```tut:book:silent
implicit def genericInstance[A, R](
  implicit
  generic: Generic.Aux[A, R],
  rInstance: Lazy[MyTC[R]] // wrap in Lazy
): MyTC[A] = ???
```

Dans le chapitre suivant nous verrons
la théorie et des patterns de programmation utiles
pour écrire ce genre de code.
Dans le Chapitre [@sec:labelled-generic],
nous reviendrons sur la déduction de type class
en utilisant une variante de `Generic` qui
permet d'inspecter les champs et les noms des types
dans nos *ADT*s.
