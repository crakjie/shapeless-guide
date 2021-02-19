## À propos du livre {#sec:intro:about-this-book}

Le livre est divisé en deux parties.

La *Partie I* introduit la déduction de *type class* (*type class derivation*),
notion permettant de créer des instances de *type class*
pour tous les types de données algébriques
à partir de quelques règles génériques uniquement.
La *Partie I* comprend quatre chapitres :

  - Le *Chapitre [@sec:representations]*
    introduit les *generic representations*,
    mais aussi la *type class* `Generic` de shapeless,
    qui permet de produire un encodage générique
    de n'importe quelle *case class* ou *sealed trait*.

  - Le *Chapitre [@sec:generic]* utilise `Generic`
    pour déduire des instances d'une *type class* particulière.
    On créé un exemple de *type class* pour
    encoder des données Scala en
    *Comma Separated Values* (*CSV*),
    mais la technique utilisée
    peut être adaptée à de nombreuses situations.
    Le type `Lazy` de shapeless est également présenté,
    il permet de manipuler les types de données récursives
    comme les listes ou les arbres.

  - Le *Chapitre [@sec:type-level-programming]*
    présente la théorie et les patrons de conception
    nécessaires à la généralisation des techniques du chapitre précédent.
    On se penchera sur les types dépendants,
    les fonctions à type dépendant et la programmation au *type level*.
    C'est l'occasion d'utiliser des applications plus avancées de shapeless.

  - Le *Chapitre [@sec:labelled-generic]* présente `LabelledGeneric`,
    une variante de `Generic` qui divulgue les champs et les noms des types
    dans sa représentation générique.
    D'avantages d'éléments théoriques sont donnés sur
    les *types littéraux*, les *types singletons*, les *types fantômes* et les *type tagging*.
    `LabelledGeneric` est illustré en créant
    un encodeur *JSON* qui préserve les champs et les noms des types dans sa sortie.

La *Partie II* présente les *ops type classes*
fournies dans le package `shapeless.ops`.
Les *ops type classes* comprennent une gamme d'outils
permettant de manipuler les représentations génériques.
Plutôt que d'expliquer en détails chaque `op`,
une synthèse théorique est réalisée en trois chapitres :

  - Le *Chapitre [@sec:ops]* discute
    d'une présentation générale des `ops` *type classes*
    et fournit un exemple
    qui regroupe plusieurs `ops`
    pour former un puissant outil de « migration de case class ».

  - Le *Chapitre [@sec:poly]* présente
    *les fonctions polymorphiques*,
    également connues sous le nom de `Polys`,
    et montre comment les utiliser dans
    les `ops` *type classes* pour *mapper*,
    *flat mapper* et *folder*
    les représentations génériques.

  - Enfin, le Chapitre [@sec:nat] présente
    le type `Nat` que shapeless utilise pour
    représenter les nombres naturels au *type level*.
    Plusieurs `ops` *type classes* associés sont présentés,
    et `Nat` est utilisé pour développer notre propre version
    de `Arbitrary` de *Scalacheck*.
