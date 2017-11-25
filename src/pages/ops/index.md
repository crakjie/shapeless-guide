# Travailler avec *HLists* et *Coproducts* {#sec:ops}

Dans la *Partie I* nous avons abordé des méthodes
visant à déduire des instances de type
class pour des types de données algébriques.
Nous pouvons utiliser la déduction automatique
d'instance pour enrichir presque toutes les type class,
cependant dans les cas les plus compliqués, cela nous amène à rédiger beaucoup de code
pour manipuler les `HList`s et les `Coproduct`s.

Dans la *Partie II*, nous allons regarder de plus près le package `shapeless.ops`,
il fournit un ensemble utile d'outils que
l'on peut utiliser comme brique de construction pour nos programmes.

Chacun des op se présente en deux parties :
une *type class* que nous pouvons utiliser pendant la résolution d'implicite
et des *méthodes d'extension* que nous pouvons appeler sur `HList` et `Coproduct`.

Il existe trois ensembles d'ops,
chacun disponible dans un des trois packages suivants :

  - `shapeless.ops.hlist` définit des type classes pour `HList`s.
    Elles peuvent être utilisées directement sur les `HList` via les
    méthodes d'extension définies dans `shapeless.syntax.hlist`.

  - `shapeless.ops.coproduct` définit des type classes pour `Coproduct`s.
    Elles peuvent être utilisées directement sur les `Coproduct` via les
    méthodes d'extension définies dans `shapeless.syntax.coproduct`.

  - `shapeless.ops.record` définit des type classes pour shapeless records
    (des `HList`s contenant des éléments taggés : Section [@sec:labelled-generic:type-tagging]).
    Elles peuvent être utilisées directement sur les `HList`s importées de `shapeless.record`  via les
    méthodes d'extension définies dans `shapeless.syntax.record`.

Nous n'avons pas le temps dans ce
livre d'aborder tous les ops disponibles.
Par chance, dans la majorité des cas le
code est compréhensible et bien documenté.
Plutôt que d'établir un guide exhaustif,
nous allons aborder les points théoriques et structurels les plus importants
et vous montrer comment obtenir
davantage d'informations de la base de code de shapeless.
