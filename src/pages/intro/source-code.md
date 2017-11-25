## Code source et exemples {#sec:intro:source-code}

Ce livre est *open source*.
Vous pouvez trouver [les sources Markdown sur GitHub][link-book-repo].
La communauté met constamment à jour ce livre,
veillez donc à vérifier sur le dépôt GitHub
que vous disposez de la dernière version.

Une copie du livre est également maintenue
sur [le site *web* Underscore][link-underscore-book].
En récupérant votre copie sur ce site,
vous pourrez être également prévenus de la sortie d'une mise à jour.

Il y existe aussi des implémentations complètes des
exemples principaux dans un [dépôt annexe][link-code-repo].
Consultez le README pour plus de détails sur l'installation.
Les exemples utilisent *shapeless 2.3.2* et
*Typelevel Scala 2.11.8+* ou
*Lightbend Scala 2.11.9+ / 2.12.1+*.

La plupart des exemples de ce livre
ont été compilés et exécutés avec
la version 2.12.1 du compilateur *Typelevel Scala*.
Entre autres,
cette version de Scala dispose de l'impression de *type infix*,
avec une sortie du *REPL* dans la console plus claire :

```tut:book:invisible
import shapeless._
```

```tut:book
val repr = "Hello" :: 123 :: true :: HNil
```

Si vous utilisez une version antérieure de Scala
vous devrez utiliser l'impression de *type prefix* ainsi :

```scala
val repr = "Hello" :: 123 :: true :: HNil
// repr: shapeless.::[String,shapeless.::[Int,shapeless.::[Boolean,shapeless.HNil]]] = "Hello" :: 123 :: true :: HNil
```

Pas de panique !
Mise à part la manière d'afficher le résultat
(syntaxe *infix* contre *prefix*),
ces types sont les mêmes.
Si vous trouvez les *prefix types* difficiles à lire,
nous vous recommendons de mettre à jour Scala.
Ajoutez simplement ceci au fichier `build.sbt`,
ou remplacez le numéro de version de manière appropriée :

```scala
scalaOrganization := "org.typelevel"
scalaVersion      := "2.12.1"
```

Le paramètre `scalaOrganization`
n'est supporté que dans *sbt 0.13.13* ou supérieur.
Vous pouvez spécifier une version *sbt*
en écrivant ceci dans `project/build.properties`
(crééz le fichier si il n'est pas déjà dans votre projet) :

```
sbt.version=0.13.13
```
