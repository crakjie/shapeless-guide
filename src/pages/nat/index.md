# Compter avec les types {#sec:nat}

De temps en temps nous avons besoin de compter quelque chose au niveau des types.
Par exemple, nous pourrions avoir besoin de connaître la taille d'une `HList`.
Nous pouvons représenter les nombres facilement avec les valeurs,
mais si nous voulons influencer la résolution d'implicite,
nous avons besoin de les représenter au niveau des types.
Ce chapitre traite des théories qui permettent de compter avec les types,
et nous fournirons quelques cas d'utilisation pour la déduction de *type class*es.

## Représenter des nombres par des types.

Shapeless utilise « le codage de Church » pour représenter
les nombres naturels au niveau des types.
Il introduit le type `Nat` avec deux sous-types :
`_0` représente zero
et `Succ[N]` représente `N+1` :

```tut:book:silent
import shapeless.{Nat, Succ}

type Zero = Nat._0
type One  = Succ[Zero]
type Two  = Succ[One]
// etc...
```

Shapeless fournit des aliases pour les 22 premiers `Nats` via `Nat._N` :

```tut:book:silent
Nat._1
Nat._2
Nat._3
// etc...
```

`Nat` n'a pas de sémantique à l'exécution.
Nous devons utiliser la *type class* `ToInt` pour convertir un `Nat`
en un `Int` à l'exécution :

```tut:book:silent
import shapeless.ops.nat.ToInt

val toInt = ToInt[Two]
```

```tut:book
toInt.apply()
```

La méthode `Nat.toInt` constitue un moyen plus pratique d'appeler `toInt.apply()`.
Elle prend en paramètre implicite une instance de `ToInt` :

```tut:book
Nat.toInt[Nat._3]
```
