## Record ops {#sec:ops:record}

Nous avons passé un certain temps dans ce chapitre
à travailler avec les type classes du package
`shapeless.ops.hlist` et `shapeless.ops.coproduct`.
Nous ne devons pas passer à côté du troisième package :`shapeless.ops.record`.

« Record ops » de shapeless fournissent des opérations semblables à celles des `Map`
sur les `HList`s d'éléments taggés.
Voici quelques exemples mettant en scène IceCreams :

```tut:book:silent
import shapeless._

case class IceCream(name: String, numCherries: Int, inCone: Boolean)
```

```tut:book
val sundae = LabelledGeneric[IceCream].
  to(IceCream("Sundae", 1, false))
```

Contrairement aux ops de `HList` et de `Coproduct` que nous avons vus précédemment
la syntaxe des record ops nécessite un import explicite de `shapeless.record` :

```tut:book:silent
import shapeless.record._
```

### Sélectionner les champs

La méthode d'extension `get` et sa type classe `Selector`
nous permettent d'extraire un champ par son tag :

```tut:book
sundae.get('name)
```

```tut:book
sundae.get('numCherries)
```

Comme nous nous y attendions, essayer d'accéder à un champ qui n'est pas défini
provoque une erreur de compilation :

```tut:book:fail
sundae.get('nomCherries)
```

### Mettre à jour ou enlever des champs

La méthode `updated` ce type class `Updater` nous permet de modifier un champ via sa clé.
La méthode `remove` ce type class `Remover` nous permet de supprimer un champ via sa clé :

```tut:book
sundae.updated('numCherries, 3)
```

```tut:book
sundae.remove('inCone)
```

La méthode `updateWith` et sa case class `Modifier` nous permettent de modifier un champ
avec une fonction de mise à jour :

```tut:book
sundae.updateWith('name)("MASSIVE " + _)
```

### Convertir en une *Map* conventionnelle

La méthode `toMap` et sa case class `ToMap`
nous permettent de convertir un record en `Map` :

```tut:book
sundae.toMap
```

### Les autres opérations

Il existe d'autres ops pour les records mais nous n'avons pas le temps de les aborder ici.
Nous pouvons renommer un champ, fusionner deux records, utiliser une fonction map sur leurs valeurs et bien plus.
Consultez le code source de `shapeless.ops.record` et `shapeless.syntax.record` pour de plus amples informations.
