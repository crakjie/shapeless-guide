# Accéder aux noms durant la déduction d'implicit {#sec:labelled-generic}

Souvent, les instances de *types classes* que l'on définit ont besoin d'accéder à plus de choses que les types uniquement.
Dans ce chapitre nous découvririons une variante de `Generic` appelée `LabelledGeneric` qui donne accès aux noms des champs et aux noms des types.

Pour commencer, abordons un peu la théorie.
`LabelledGeneric` emploie des techniques astucieuses pour exposer les informations sur les noms au niveau des types.
Pour comprendre ces techniques, nous devons traiter les *types littéraux*, *types singleton*, *types fantômes* et le *type tagging*.
