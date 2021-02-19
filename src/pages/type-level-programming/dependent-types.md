## Types dépendants

Dans le dernier chapitre, nous avons passé beaucoup de temps à utiliser `Generic`,
la *type class* qui sert à lier un type *ADT* à sa représentation générique.
Pourtant, nous n'avons pas encore abordé la théorie sous-jacente à `Generic` et à tout shapeless :
*les types dépendants*.


Pour illustrer tout ca, intéressons-nous de plus près à `Generic`.
Voici une version simplifiée de sa définition :

```scala
trait Generic[A] {
  type Repr
  def to(value: A): Repr
  def from(value: Repr): A
}
```
Les instances de `Generic` font référence à deux autres types :
un paramètre de type `A` et un membre de type `Repr`.
Imaginons que l'on implémente une méthode `getRepr` de la façon suivante.
Quel type allons-nous obtenir en retour ?

```tut:book:silent
import shapeless.Generic

def getRepr[A](value: A)(implicit gen: Generic[A]) =
  gen.to(value)
```

La réponse est : tout dépend de l'instance de `gen` que nous avons.
En développant l'appel de `getRepr`,
le compilateur va chercher un `Generic[A]`
et le type sera le `Repr` défini dans l'instance :

```tut:book:silent
case class Vec(x: Int, y: Int)
case class Rect(origin: Vec, size: Vec)
```

```tut:book
getRepr(Vec(1, 2))
getRepr(Rect(Vec(0, 0), Vec(5, 5)))
```

Ce que nous voyons ici est appelé *typage dependent*:
le type du résultat de `getRepr` dépend de la valeur de son
paramètre de type via son membre de type.
Imaginons que nous avons spécifié `Repr` comme paramètre de type
de `Generic` à la place du membre de type:

```tut:book:silent
trait Generic2[A, Repr]

def getRepr2[A, R](value: A)(implicit generic: Generic2[A, R]): R =
  ???
```

Nous aurions dû passer la valeur désirée de `Repr` dans le paramètre de type de `getRepr`,
ce qui finit par rendre `getRepr` inutile.
On peut en déduire que les paramètres de type sont utiles en tant « qu'inputs »
et les membres de type sont utiles en tant « qu'outputs. »
