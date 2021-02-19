# Opération fonctionnelle sur les *HLists* {#sec:poly}

Les programmes Scala « classiques » utilisent beaucoup
les opérations fonctionnelles
telles que `map` et `flatMap`.
Une question se pose : peut-on effectuer
des opérations semblables sur les HLists` ?
La réponse est oui, mais nous devons faire les choses
légèrement différemment par rapport au Scala classique.
Nous ne serons pas surpris de constater que le mécanisme utilisé reste basé
sur les *type classes* et un ensemble de *type classes* ops pour nous aider.

Avant de nous pencher directement sur les *type classes*,
nous devons aborder la façon dont shapeless représente les
*fonctions polymorphiques* qui seront capables
de mapper sur une structure de données hétérogènes.

## Motivation : mapper sur une *HList*

Nous allons traiter les fonctions polymorphiques
à l'aide de la méthode `.map`
La Figure [@fig:poly:monomorphic-map] montre un diagramme des types
d'une fonction map classique sur les listes.
Nous commençons avec une `List[A]`,
nous fournissons une fonction `A => B`,
et nous terminons par `List[B]`.

![Mapper sur une liste classique ("monomorphic" map)](src/pages/poly/monomorphic-map.pdf+svg){#fig:poly:monomorphic-map}

Le fait qu'il y ait des éléments hétérogènes dans une `HList`
met ce modèle à mal.
Les fonctions Scala ont des types d'entrée et de sortie fixés,
le résultat de notre map devra donc avoir
le même type pour toutes les positions.

Dans l'idéal, nous voulons que notre fonction `map` corresponde à
la Figure [@fig:poly:polymorphic-map], où la fonction inspecte
le type de chaque entrée pour déterminer le type de chaque sortie.
Ce qui nous donne un environnement clos et
composable qui conserve la nature hétérogène
des `HList`.

![mapper sur des listes hétérogènes ("polymorphic" map)](src/pages/poly/polymorphic-map.pdf+svg){#fig:poly:polymorphic-map}

Malheureusement nous ne pouvons pas utiliser les fonctions Scala pour implémenter
ces opérations. Nous avons besoin d'une nouvelle infrastructure.
