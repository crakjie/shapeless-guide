## Résumé

Dans ce chapitre, nous avons traité la façon dont shapeless représente les nombre naturels
et comment nous pouvons les utiliser dans les *type classes*.
Nous avons vu qu'il existe déjà des *type classes* ops
qui nous permettent de faire des choses comme
calculer des tailles et accéder à des éléments par leurs index.
Nous avons aussi vu comment créer nos *type classes*
qui utilisent `Nat` pour leurs propres besoins.


Avec `Nat`, `Poly` et les variétés de type que nous avons vues dans le dernier chapitre, nous n'avons abordé qu'une fraction des outils fournit par `shapeless.ops`.
Il existe de nombreuses autres *type classes* qui apportent
des fondations structurées sur lesquelles nous pouvons construire notre propre code.

Cependant, la théorie décrite ici est suffisante pour comprendre la majorité des opérations nécessaires à la déduction de nos propre *type classes*.
Le code source du package `shapeless.ops` devrait vous être désormais suffisamment compréhensible pour que vous y sélectionniez d'autre ops qui vous seront utiles.
