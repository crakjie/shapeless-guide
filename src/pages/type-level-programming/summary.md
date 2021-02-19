## Résumé {#sec:type-level-programming:summary}

Lorsque l'on code avec shapeless,
on cherche souvent à trouver un
type qui dépend de valeurs présentes dans le code.
Cette relation est appelée *Types dépendants*.

Les problèmes impliquant les types dépendants peuvent être aisément exprimés via la recherche d'implicites,
ce qui permet au compilateur de déterminer
les types intermédaires et les types ciblés
d'après un point de départ à l'emplacement d'appel.

Plusieurs étapes sont souvent nécessiares pour
calculer le resultat
(par exemple : utiliser `Generic` pour avoir un `Repr`,
puis utiliser une autre *type class* pour avoir un autre type).
Dans ce cas, on peut suivre quelques règles
pour nous assuer que notre code compile et fonctionne comme prévu :


 1. Il faut placer tous les types intermédiaires dans la liste des paramètres de types.
    De nombreux paramètres de type ne seront pas utilisés dans le résultat mais le compilateur en a besoin pour savoir quel type il doit unifier.

 2. Le compilateur résoud les implicites de gauche à droite
    et effectue du backtracking s'il ne
    peut pas trouver une combinaison qui fonctionne.
    Nous devons donc écrire les implicites dans l'ordre où nous en avons besoin,
    à l'aide d'une ou plusieurs variables de
    types pour les connecter aux implicites précédents.    

 3. Le compilateur ne peut résoudre qu'une contrainte à la fois,
    nous ne devons donc pas sur-contraindre un implicite.

 4. Il faut énoncer le type de retour explicitement,
    spécifier tout paramètre de types et tout membre
    de types qui pourrait être utilisé ailleurs.
    Les membres de types sont souvent importants,
    il faut donc utiliser les types `Aux` pour les préserver
    le cas échéant.
    Si on ne les énonce pas dans le type de retour
    ils ne seront pas disponibles
    pour les recherches d'implicites suivantes.

 5. Le pattern du `type alias` `Aux` reste utile pour maintenir un code propre.
    Il faut faire attention aux alias `Aux` quand on utilise les outils de la toolbox shapeless
    et implémenter les alias `Aux` dans nos propres fonctions à types dépendants.

Quand nous trouvons une chaîne d'opération à type dépendant nous pouvons les regrouper
dans une seule type class.
Ceci est souvent appelé pattern « lemma »
(un terme emprunté aux preuves mathématiques).
Nous verrons un exemple de ce pattern dans la section [@sec:ops:penultimate].
