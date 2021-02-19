# Type de données algébriques et représentation générique {#sec:representations}

La programmation générique a pour objectif principal
de résoudre des problèmes pour une grande variété de types
en utilisant un minimum de code générique.
Shapeless fournit deux ensembles d'outils dans ce but :

 1. Un ensemble de types de donnée générique
    qui peuvent être inspectés, itérés et manipulés
    au `type-level`

 2. Le mapping automatique entre *algebraic data types (*ADT*s)*
    (encodé en Scala par les *case classes* et les *sealed traits*)
    et leurs représentations génériques.

Dans ce chapitre, nous commençons par un
récapitulatif sur la théorie des types algébriques
et la raison pour laquelle ils peuvent être familiers pour le développeur Scala.
Puis, nous verrons les représentations génériques utilisées
par shapeless et nous traiterons de la façon dont ils sont
reliés aux *ADT*s concrets.
Enfin, nous présenterons une *type class* appelée `Generic`
qui fournit un mapping automatique bidirectionnel entre
un *ADT* et sa représentation générique.
Enfin, nous utiliserons `Generic` dans quelques exemples
pour convertir des valeurs d'un type vers un autre.
