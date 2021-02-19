## Résumé

Dans ce chapitre, nous avons abordé `LabelledGeneric`,
une variante de `Generic` qui donne accès aux noms des
champs et aux noms des types dans sa représentation générique.

Les noms accessibles grâce à `LabelledGeneric` sont encodés par des tags au type-level afin de les utiliser durant la résolution d'implicites.
Nous avons commencé ce chapitre en abordant les *types littéraux*
et la façon dont shapeless les utilise dans ces tags.
Nous avons également abordé la *type class* `Witness` utilisée pour
extraire les valeurs des types littéraux.

Enfin, nous avons combiné `LabelledGeneric`, les types littéraux et `Witness`
pour construire une bibliothèque d'encodage *JSON* qui permet d'inclure les noms présents dans les *ADT* dans la sortie *JSON*.

Le plus important dans ce chapitre est que tout ce code n'utilise jamais la réflexion.
Tout est implémenté avec des types, des implicits et le petit ensemble de macros interne à shapeless.
Le code que nous générons est par conséquent très rapide à exécuter.
