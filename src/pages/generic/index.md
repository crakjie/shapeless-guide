# Déduire automatiquement les instances de type class {#sec:generic}

Dans le dernier chapitre, nous avons vu comment le type `Generic`
nous permet de convertir toute instance d'un ADT en un
encodage générique composé de `HList`s et `Coproduct`s.
Dans ce chapitre, nous verrons notre premier véritable cas d'utilisation :
la déduction automatique d'instance de type classes.
