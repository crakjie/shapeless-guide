## Résumé

Dans ce chapitre, nous avons traité des *fonctions polymorphiques*
dont le type de retour varie en fonction du type de paramètre.
Nous avons vu comment le type `Poly` de shapeless est défini
et comment implémenter des opérations fonctionnelles comme `map`, `flatMap`, `foldLeft` et `foldRight`.

Chaque opération est implémentée comme une méthode d'extension sur `HList`,
basée sur une des *type classes* correspondantes :
`Mapper`, `FlatMapper`, `LeftFolder` et les autres.
Nous pouvons utiliser ces *type classes*, `Poly` et les
techniques de la section [@sec:type-level-programming:chaining]
pour créer nos propres *type classes* comportant un enchaînement de transformations sophistiquées.

