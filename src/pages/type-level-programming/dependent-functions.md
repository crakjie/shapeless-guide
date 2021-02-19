## Fonctions à type dépendant {#sec:type-level-programming:depfun}

Shapeless utilise les types dépendants partout :
dans `Generic`, dans `Witness` (que nous traiterons dans le chapitre suivant),
et dans de nombreux *type classes* « ops » que
nous traiterons dans la *Partie II* de ce guide.
Par exemple, shapeless fournit une case class appelée `Last`
qui retourne le dernier élément d'une `HList`.
Voici une version simplifiée de son implémentation :

```scala
package shapeless.ops.hlist

trait Last[L <: HList] {
  type Out
  def apply(in: L): Out
}
```

On peut invoquer des instances de `Last`
pour inspecter les `HList`s dans notre code.
Notez que dans les deux exemples ci-dessous
les types de `Out` sont dépendants des types des `HList` :

```tut:book:silent
import shapeless.{HList, ::, HNil}

import shapeless.ops.hlist.Last
```

```tut:book
val last1 = Last[String :: Int :: HNil]
val last2 = Last[Int :: String :: HNil]
```
Une fois les instances de `Last` invoquées,
et on peut les utiliser au value level
via leurs méthodes `apply` :

```tut:book
last1("foo" :: 123 :: HNil)
last2(321 :: "bar" :: HNil)
```

Nous avons deux formes de protection contre les erreurs.
L'implicite définie pour `Last` nous garantit que
l'on peut construire une instance de `Last` uniquement si
la `HList` en entrée est dotée d'au moins un élément :

```tut:book:fail
Last[HNil]
```

De plus, le paramètre de type de l'instance de `Last`
vérifie qu'on lui passe bien
une instance de `HList` du bon type :

```tut:book:fail
last1(321 :: "bar" :: HNil)
```

Pour faire un autre exemple, implémentons notre propre
type classe, appelé `Second`,
qui retourne le second élément d'une `HList` :

```tut:book:silent
trait Second[L <: HList] {
  type Out
  def apply(value: L): Out
}

object Second {
  type Aux[L <: HList, O] = Second[L] { type Out = O }

  def apply[L <: HList](implicit inst: Second[L]): Aux[L, inst.Out] =
    inst
}
```

Ce code utilise l'agencement idiomatique
décrit dans la section [@sec:generic:idiomatic-style].
On définit un type `Aux` dans l'objet compagnon aux cotés de la
méthode standard `apply`, qui permet d'invoquer des instances.

<div class="callout callout-warning">
*Méthode d'invocation versus "implicitly" versus "the"*

Remarquez que le type de retour d'`apply` est `Aux[L, O]` et non pas `Second[L]`.
C'est important.
Utiliser `Aux` assure que la methode `apply`
n'écrase pas les membres de type de l'instance invoquée.
Si l'on définit `Second[L]` comme type de retour,
le membre de type `Out` sera perdu dans le type retourné et
la *type class* ne fonctionnera plus correctement.

La méthode `implicitly` de `scala.Predef` est dotée de cette propriété.
Comparons le type d'une instance de `Last` invoqué par `implicitly` :


```tut:book
implicitly[Last[String :: Int :: HNil]]
```

au type retourné par  `Last.apply` :

```tut:book
Last[String :: Int :: HNil]
```

Le type invoqué par `implicitly` n'a pas de membre de type `Out`.
C'est pourquoi on doit éviter d'utiliser `implicitly`
lorsque l'on travaille avec des fonctions à type dépendant.
On peut utiliser une méthode d'invocation custom ou directement
la méthode `the` de shapeless :

```tut:book:silent
import shapeless._
```

```tut:book
the[Last[String :: Int :: HNil]]
```
</div>

Nous n'avons besoin que d'une seule instance
définie pour une `HList`s d'au moins deux éléments :


```tut:book:invisible
import Second._
```

```tut:book:silent
implicit def hlistSecond[A, B, Rest <: HList]: Aux[A :: B :: Rest, B] =
  new Second[A :: B :: Rest] {
    type Out = B
    def apply(value: A :: B :: Rest): B =
      value.tail.head
  }
```

On peut invoquer les instances en utilisant `Second.apply` :

```tut:book:invisible
import Second._
```

```tut:book
val second1 = Second[String :: Boolean :: Int :: HNil]
val second2 = Second[String :: Int :: Boolean :: HNil]
```

L'invocation est sujette aux mêmes contraintes que `Last`.
Si l'on essaie d'invoquer une instance pour une `HList` incompatible,
alors la résolution échoue et l'on obtient une erreur de compilation :


```tut:book:fail
Second[String :: HNil]
```
Les instances invoquées avec la méthode `apply` fonctionne
avec les types de `HList` appropriés au niveau des valeurs (value level)

```tut:book
second1("foo" :: true :: 123 :: HNil)
second2("bar" :: 321 :: false :: HNil)
```

```tut:book:fail
second1("baz" :: HNil)
```

## Enchaîner les fonctions dépendantes {#sec:type-level-programming:chaining}
Les fonctions à type dépendant offrent un
moyen de calculer un type à partir d'un autre.
On peut enchaîner les fonctions à type dépendant
pour effectuer un calcul nécessitant plusieurs étapes.
Par exemple, on pourrait utiliser `Generic` pour calculer
le `Repr` d'une case classe et utiliser `Last`
pour calculer le type du dernier élément.
Essayons de coder ceci :


```tut:book:invisible
import shapeless.Generic
```

```tut:book:fail
def lastField[A](input: A)(
  implicit
  gen: Generic[A],
  last: Last[gen.Repr]
): last.Out = last.apply(gen.to(input))
```

Malheureusement, notre code ne compile pas.
Il s'agit du même problème que nous avions rencontré dans la section [@sec:generic:product-generic]
avec notre définition de `genericEncoder`.
Nous avions contourné le problème en remontant la variable
de type non-parametré dans la liste des paramètres de types :

```tut:book:silent
def lastField[A, Repr <: HList](input: A)(
  implicit
  gen: Generic.Aux[A, Repr],
  last: Last[Repr]
): last.Out = last.apply(gen.to(input))
```

```tut:book:invisible
case class Vec(x: Int, y: Int)
case class Rect(origin: Vec, extent: Vec)
```

```tut:book
lastField(Rect(Vec(1, 2), Vec(3, 4)))
```

En règle général, on écrit toujours du code dans ce style.
En paramétrant toutes les variables de type, on permet au compilateur
de les unifier avec les types appropriés.
Cela vaut également pour des contraintes plus subtiles.
Par exemple, imaginons qu'on veuille invoquer un `Generic` pour une case class
qui porte exactement un champ.
On pourrait être tenté d'écrire le code suivant :

```tut:book:silent
def getWrappedValue[A, H](input: A)(
  implicit
  gen: Generic.Aux[A, H :: HNil]
): H = gen.to(input).head
```

Le résultat est plus pernicieux.
La définition de la méthode compilera
mais le compilateur ne trouvera jamais
d'implicits au moment de l'appel :

```tut:book:silent
case class Wrapper(value: Int)
```

```tut:book:fail
getWrappedValue(Wrapper(42))
```

Le message d'erreur fait allusion au problème.
L'apparition du type `H` est en fait notre indice.
C'est le nom d'un des paramètres de type de la méthode,
il ne devrait pas apparaître
dans le type que le compilateur tente d'unifier.
Le problème tient au fait que le paramètre `gen` est sur-contraint :*
le compilateur ne peut pas trouver `Repr` *et* garantir sa taille en même temps.
Le type `Nothing` peut aussi fournir un indice,
il apparaît quand le compilateur ne parvient pas
à unifier les paramètres de types covariants.
La solution au problème ci-dessus est de diviser la résolution d'implicite en plusieurs étapes :

1. trouver un `Generic` avec un `Repr` approprié pour `A` ;
2. s'assurer que `Repr` a un élément en tête de type `H`.

Voici une version revisitée de la méthode
utilisant `=:=` pour contraindre `Repr`:


```tut:book:fail
def getWrappedValue[A, Repr <: HList, Head, Tail <: HList](input: A)(
  implicit
  gen: Generic.Aux[A, Repr],
  ev: (Head :: Tail) =:= Repr
): Head = gen.to(input).head
```

Ça ne compile pas
car la methode `head` dans le corps de la methode getWrappedValue
nécessite un paramètre implicite de type `IsHCons`.
Ce sont des messages d'erreur beaucoup plus simples à corriger,
il nous suffit d'apprendre à utiliser un outil de la toolbox de shapeless.
`IsHCons` est une type classe de shapeless qui coupe une `HList` en une `Head` et une `Tail`.
On peut utiliser `IsHCons` en lieu est place de  `=:=` :

```tut:book:silent
import shapeless.ops.hlist.IsHCons

def getWrappedValue[A, Repr <: HList, Head](in: A)(
  implicit
  gen: Generic.Aux[A, Repr],
  isHCons: IsHCons.Aux[Repr, Head, HNil]
): Head = gen.to(in).head
```

Ça corrige le bug.
Maintenant la definition de la méthode et
l'appel de la méthode compile correctement :


```tut:book
getWrappedValue(Wrapper(42))
```
Le point important n'est pas que
l'on a résolu le problème en utilisant `IsHCons`.
Shapeless fournit de nombreux outils comme celui-ci
(voir Chapitre [@sec:ops] à [@sec:nat]),
et on peut les compléter si nécessaire
avec nos propres *type classes*.
Le point important est la compréhension du processus
que l'on utilise pour parvenir à écrire un code qui compile
et la capacité de trouver les solutions
Cette section se conclue par un
guide pas à pas qui résume
ce que nous avons découvert jusqu'ici.
