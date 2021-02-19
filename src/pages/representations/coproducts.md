## Coproduit générique

Maintenant que nous savons comment Shapeless encode les types produit,
qu'en est-il des coproduits ?
Précédemment, nous avons abordé le `Either`, mais il souffre
des mêmes inconvénients que les *tuples*.
Encore une fois, shapeless fournit son propre encodage, similaire au `HList` :

```tut:book:silent
import shapeless.{Coproduct, :+:, CNil, Inl, Inr}

case class Red()
case class Amber()
case class Green()

type Light = Red :+: Amber :+: Green :+: CNil
```

En général, les coproduits prennent la forme de
`A :+: B :+: C :+: CNil` signifiant « A ou B ou C, »
où `:+:` peut être librement interprété comme `Either`.
Globalement, le type d'un coproduit encode tous les types
possibles d'une disjonction, mais ces instances contiennent
uniquement la valeur de l'une des possibilités.
`:+:` dispose de deux sous types, `Inl` et `Inr`,
qui correspondent vaguement à `Left` et `Right`.
On crée des instances de coproduit en
imbriquant des constructeurs de `Inl` et de `Inr` :

```tut:book
val red: Light = Inl(Red())
val green: Light = Inr(Inr(Inl(Green())))
```
Chaque type de coproduit se termine par  `CNil`,
qui est un type inhabité (sans valeur), similaire à `Nothing`.
On ne peut donc pas instancier `CNil` ou construire un `Coproduct` uniquement
à partir d'instance de `Inr`.
Il y a toujours exactement un `Inl` dans chaque valeur de coproduit.

Encore une fois, il convient de signaler que `Coproduct`s n'a rien de spécial.
La fonctionnalité du dessus peut être obtenue en utilisant `Either` et `Nothing`
à la place de `:+:` et `CNil`.
Utiliser `Nothing` induit des difficultés techniques,
mais on aurait pu utiliser
n'importe quel autre type inhabité ou type singleton à la place de `CNil`.

### Échanger les encodages à l'aide de *Generic*

À première vue, les `Coproduct` sont difficiles à parser.
Cependant, nous voyons comment ils s'intègrent dans le contexte
plus large des écritures génériques.
En plus de comprendre les *case classes* et les *case objects*,
`Generic` de shapeless comprend également les *sealed traits* et les
classes abstraites :

```tut:book:silent
import shapeless.Generic

sealed trait Shape
final case class Rectangle(width: Double, height: Double) extends Shape
final case class Circle(radius: Double) extends Shape
```

```tut:book
val gen = Generic[Shape]
```

Le `Repr` du `Generic` de `Shape` est un coproduit
des sous-types de `Shape`: `Rectangle :+: Circle :+: CNil`.
On peut utiliser les méthodes `to` et `from` de l'instance de `Generic`
pour convertir `Shape` en `gen.Repr` et vice versa :

```tut:book
gen.to(Rectangle(3.0, 4.0))

gen.to(Circle(1.0))
```
