## Écriture générique pour les produits

Dans la section précédente, nous avons présenté les *tuples*
comme une représentation générique des produits.
Malheureusement, les *tuples* présentent deux inconvénients qui les rendent
inappropriés aux besoins de shapeless.

 1. Chaque taille de tuple est dotée d'un type différent, ces types sont indépendants les uns des autres,
    ce qui rend difficile l'écriture de code qui fait abstraction de la taille des produits.

 2. Il n'existe pas de *tuples* de taille zero,
    qui sont pourtant importants pour représenter les produits sans champ.
    On pourrait utiliser `Unit`,
    mais on souhaite que chaque représentation générique
    ait un supertype commun et tangible.
    Le plus petit ancêtre commun à `Unit` et `Tuple2` est `Any`,
    une combinaison des deux est donc impensable.

C'est pourquoi shapeless utilise un encodage générique différent
pour les produits :  *heterogeneous lists* ou `HList`s[^hlist-name].

[^hlist-name]: `Product` est probablement un meilleur nom pour `HList`,
mais la bibliothèque standard dispose malheureusement déjà d'un `scala.Product`.

La `HList` est soit une liste vide `HNil`,
soit une paire `::[H, T]` où `H` est un type arbitraire
et `T` une autre `HList`.
Parce que chaque `::` a son propre `H` et `T`,
le type de chaque élément est encodé séparément
dans le type de la liste global:

```tut:book:silent
import shapeless.{HList, ::, HNil}

val product: String :: Int :: Boolean :: HNil =
  "Sunday" :: 1 :: false :: HNil
```
Le type et la valeur de la `HList` ci-dessus se reflètent.
Les deux représentent les 3 membres : une `String`, un `Int`, et un `Boolean`.
On peut y retrouver la `tête` et la `queue`,
et le type des éléments y est préservé :

```tut:book
val first = product.head
val second = product.tail.head
val rest = product.tail.tail
```
Le compilateur connaît la taille exacte de chaque `HList`,
cela devient donc une erreur de compilation
de demander la `tête` ou la `queue` d'une liste vide :


```tut:book:fail
product.tail.tail.tail.head
```
En plus de pouvoir inspecter et itérer sur les `HList`s,
on peut les manipuler et les transformer.
Par exemple, on peut ajouter un élément avec la méthode `::`.
Encore une fois, notez comme le type du résultat reflète
le nombre des éléments de la liste.

```tut:book:silent
val newProduct: Long :: String :: Int :: Boolean :: HNil =
  42L :: product
```
Shapeless fournit aussi des outils pour effectuer des opérations plus complexes
comme un mapping, un filtrage ou une concaténation de listes.
On abordera cela plus en détails dans la *Partie II*.

Les propriétés que l'on obtient avec `HList`s ne sont pas magiques.
On aurait pu obtenir ces fonctionalités en utilisant `(A, B)` et `Unit`
comme alternative à  `::` et `HNil`.
Néanmoins, il existe un avantage à garder nos
types génériques séparés de nos types sémantiques
dans les applications.
`HList` fournit cette séparation.

### Changer de représentation en utilisant *Generic*

Shapeless fournit une type class appelée `Generic`
qui permet de convertir des *ADT* concrets en représentation générique
et vice versa.
Il y a quelques macros en coulisses qui permettent d'invoquer des instances
de `Generic` sans code *boilerplate* :

```tut:book:silent
import shapeless.Generic

case class IceCream(name: String, numCherries: Int, inCone: Boolean)
```

```tut:book
val iceCreamGen = Generic[IceCream]
```
Notez que les instance de `Generic` ont un membre de type `Repr`
qui contient le type de sa représentation générique.
Dans notre cas `iceCreamGen.Repr` est `String :: Int :: Boolean :: HNil`.
Les instances de `Generic` disposent de deux méthodes :
Une pour convertir vers le type `Repr` appelé `to`
et la méthode inverse `from` :

```tut:book
val iceCream = IceCream("Sundae", 1, false)

val repr = iceCreamGen.to(iceCream)

val iceCream2 = iceCreamGen.from(repr)
```

On peut convertir de l'un vers l'autre deux *ADT*s qui ont le
même `Repr` en utilisant leurs `Generics` :

```tut:book:silent
case class Employee(name: String, number: Int, manager: Boolean)
```

```tut:book
// Create an employee from an ice cream:
val employee = Generic[Employee].from(Generic[IceCream].to(iceCream))
```

<div class="callout callout-info">
*Les autres types de produits*

Il vaut la peine de souligner que les *tuples Scala* sont des *case classes*,
donc `Generic` fonctionne très bien avec :

```tut:book:silent
val tupleGen = Generic[(String, Int, Boolean)]
```

```tut:book
tupleGen.to(("Hello", 123, true))
tupleGen.from("Hello" :: 123 :: true :: HNil)
```
Cela marche aussi avec les *case classes* de plus de 22 champs:

```tut:book:silent
case class BigData(
  a:Int,b:Int,c:Int,d:Int,e:Int,f:Int,g:Int,h:Int,i:Int,j:Int,
  k:Int,l:Int,m:Int,n:Int,o:Int,p:Int,q:Int,r:Int,s:Int,t:Int,
  u:Int,v:Int,w:Int)
```

```tut:book
Generic[BigData].from(Generic[BigData].to(BigData(
  1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23)))
```
</div>
