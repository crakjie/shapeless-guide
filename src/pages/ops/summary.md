## Résumé

Dans ce chapitre, nous avons exploré quelques-unes des *types classes* qui
sont fournies par le package `shapeless.ops`.
Nous nous sommes intéressés à deux exemples simples du pattern ops : `Last` et `Init`,
et nous avons construit nos propres *type classes* `Penultimate` et `Migration`
en associant des briques de construction pré-existantes.

De nombreuses type classe ops partagent les mêmes techniques
que celui des ops que nous avons vus jusqu'ici.
La facon la plus simple de les apprendre et
de regarder directement leur code source dans
`shapeless.ops` and `shapeless.syntax`.

Dans le chapitre suivant, nous allons aborder deux ensembles de *type class* `ops`
qui nécessitent davantage de théorie.
Le *Chapitre [@sec:poly]* traite des opérations fonctionnelles comme
`map` et `flatMap` sur les `HList`s, et le Chapitre [@sec:nat] nous explique
comment implémenter des *type classes* qui
nécessitent une représentation au *type level* des nombres.
Ces informations vont nous aider à mieux comprendre
l'ensemble des *type classes* de `shapeless.ops`.
