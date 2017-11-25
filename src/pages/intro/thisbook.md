## À propos du livre {#sec:intro:about-this-book}

Le livre est divisé en deux parties.

Dans la Partie I, nous introduisons la *déduction de *type class* (type class derivation)*,
qui permet de créer des instances de *type class*
pour tous types de données algébriques
avec quelques règles génériques pour tout matériel.
La Partie I comprend quatre chapitres :

  - Dans le chapitre [@sec:representations],
    nous introduisons les *generic representations*,
    mais aussi la *type class* `Generic` de shapeless,
    qui permet de produire un encodage générique
    de n'importe quelle case class ou famille scellée.

  - Dans le chapitre [@sec:generic] on utilise `Generic`
    pour déduire une instance d'une de vos propres type class.
    On crée un exemple de *type class* pour
    encoder des données Scala en
    Comma Separated Values (CSV),
    mais la technique utilisée
    peut être adaptée à de nombreuses situations.
    Nous présentons également le type `Lazy` de shapeless,
    qui permet de manipuler les types de données récusives
    comme les listes ou les abres.

  - Dans le chapitre [@sec:type-level-programming],
    nous présentons les théories et les patterns de programmation
    nécessaires à la généralisation des techniques du chapitre précédent.
    On se penchera sur les types dépendants,
    les fonctions à type dépendant et la programmation au type level.
    Cela nous permet d'utiliser des applications plus avancées de shapeless.

  - Dans le chapitre [@sec:labelled-generic], nous présentons `LabelledGeneric`,
    une variante de `Generic` qui divulgue les champs et les noms des types
    dans sa repésentation générique.
    Nous présentons également une théorie supplémentaire :
    les types littéraux, les types singleton, les types fantôme et les type tagging.
    Nous illustrons `LabelledGeneric` en créant
    un encodeur *JSON* qui préserve les champs et les noms des types dans sa sortie.

Dans la Partie II, nous présentons les "ops type classes"
fournies dans le package `shapeless.ops`.
Les ops type classes constituent une large bibliotèque d'outils
pour manipuler les représentations génériques.
Plutôt que d'expliquer en détails chaque op,
nous offrons une base théorique en trois chapitres :

  - Dans le chapitre [@sec:ops], nous proposons
    une présentation générale des ops type classes
    et fournissons un exemple
    qui relie plusieurs ops ensemble
    pour former un puissant outil de "migration de case class".

  - Dans le chapitre [@sec:poly], nous présentons
    *les fonctions polymorphique*,
    également connues sous le nom de `Polys`,
    et montrons comment les utiliser dans
    les ops type classes pour « mapper »,
    « flat mappés » et « folder »
    les représentations génériques.

  - Enfin, dans le chapitre [@sec:nat] nous présentons
    le type `Nat` que shapeless utilise pour
    représenter les nombres naturels au type level.
    Nous présentons plusieurs ops type classes associés,
    et utilisons `Nat` pour développer notre propre version
    de `Arbitrary` de Scalacheck.

## Code source et exemples

Ce livre est *open source*.
Vous pouvez trouver [les sources Markdown sur GitHub][link-book-repo].
La communauté met constament à jour ce livre.
Veillez donc à vérifier sur le dépôt GitHub
que vous disposez de la dernière version.

Il y existe aussi des implémentations complètes des
exemples principaux dans un [repo compagnon][link-code-repo].
Consultez le README pour plus de détails sur l'installation.

Les exemples utilisent *shapeless 2.3.2* et
*Typelevel Scala 2.11.8+* ou
*Lightbend Scala 2.11.9+ / 2.12.1+*.
