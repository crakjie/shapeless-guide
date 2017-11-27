## Résumé

Dans ce chapitre, nous avons abordé les représentations génériques
que Shapeless fournit pour les types de données algébriques de Scala.
`HList`s pour les types produits et `Coproduct`s pour les types coproduits.
Nous avons aussi présenté la *type class* `Generic` qui fournit mapping bidirectionnel entre
un *ADT* et sa représentation générique.
Nous n'avons pas encore abordé ce qui rend les écritures génériques si attractives.
Le cas d'utilisation que nous avons couvert (conversion entre *ADT*s) est amusant
mais pas particulièrement utile.

La vraie puissance de `HList`s et de `Coproduct`s provient de leur structure récursive.
On peut écrire du code qui traverse leur représentation et calcule des valeurs à partir
des éléments qui les constituent.
Dans le prochain chapitre nous verrons notre premier véritable cas d'utilisation :
la déduction automatique d'instance de *type class*es.
