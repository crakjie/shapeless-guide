## Fonctions polymorphique

Shapeless fournit un type appelé `Poly`
pour représenter les *fonctions polymorphiques*
dont le type de retour dépend des paramètres de types.
Voici une explication simplifiée de leur fonctionnement.
Notez que la section suivante ne contient pas de vrai code shapeless,
nous mettons de coté une partie de la flexibilité et des mécanismes qui servent à simplifier l'utilisation de la vraie bibliothèque de shapeless pour créer une version simplifiée dans le cadre de notre exemple.

### Comment *Poly* fonctionne

Au cœur de `Poly`, on trouve un objet avec une méthode `apply` générique.
En plus de l'habituel paramètre de type `A`,
`Poly` prend un paramètre implicite de type `Case[A]`:

```tut:book:silent
// Ceci n'est pas du code shapeless.
// Ceci n'est qu'une démonstration.

trait Case[P, A] {
  type Result
  def apply(a: A): Result
}

trait Poly {
  def apply[A](arg: A)(implicit cse: Case[this.type, A]): cse.Result =
    cse.apply(arg)
}
```

Lorsque qu'on définit un véritable `Poly`,
on fournit une instance de `Case` pour
chaque paramètre de type dont nous avons besoin.
Ceci permet d'implémenter le vrai corps de la fonction :

```tut:book:silent
// Ceci n'est pas du code shapeless.
// Ceci n'est qu'une démonstration.

object myPoly extends Poly {
  implicit def intCase =
    new Case[this.type, Int] {
      type Result = Double
      def apply(num: Int): Double = num / 2.0
    }

  implicit def stringCase =
    new Case[this.type, String] {
      type Result = Int
      def apply(str: String): Int = str.length
    }
}

```

Lorsqu'on appelle `myPoly.apply`,
le compilateur recherche le `Case` implicite correspondant
et l'intègre comme à l'accoutumée :

```tut:book
myPoly.apply(123)
```

Il y a ici quelques subltilités de scoping, qui permettent
au compilateur de trouver les instances de `Case` sans import supplémentaire.
`Case` a un paramètre de type supplémentaire appelé `P`
qui référence le type singleton de `Poly`.
Le scope de `Case[P, A]` contient comme implicit une référence
à l'objet compagnon de `Case`, `P` et de `A`.
Nous avons assigné  `myPoly.type` à `P` et
l'objet compagnon de  `myPoly.type` est `myPoly` lui-même.
En d'autre termes, `Cases` qui est défini dans le corps de `Poly`
est toujours accesible dans le scope quel que soit l'emplacement de l'appel.

### la syntaxe de *Poly*

Le code ci-dessus n'est pas le véritable code de shapeless.
Heureusement, shapeless permet de définir les `Polys` de facon bien plus simple.
Voici notre fonction `myPoly` réécrite avec la véritable syntaxe :

```tut:book:silent
import shapeless._

object myPoly extends Poly1 {
  implicit val intCase: Case.Aux[Int, Double] =
    at(num => num / 2.0)

  implicit val stringCase: Case.Aux[String, Int] =
    at(str => str.length)
}
```

Il y a quelques différences notables par rapport à notre syntaxe précédente :

 1. Nous étendons le trait appelé `Poly1` au lieu de `Poly`.
    Shapeless a un type `Poly` et un ensemble de sous-types,
    `Poly1` a `Poly22`, qui supporte les différentes variétés de fonctions polymorphiques.

 2. Le type `Case.Aux` ne semble pas faire
    référence au type singleton de `Poly`.
    `Case.Aux` est en fait un alias de type défini dans le corp de `Poly1`.
    Le type singleton est bien présent, il est simplement caché.

 3. Nous utilisons la méthode helper `at` pour définir nos cas.
    Elle agit à la façon d'une méthode constructrice d'instance,
    comme traité dans la section [@sec:generic:idiomatic-style]),
    elle sert principalement à éliminér le boilerplate.

Une fois les différences de syntaxe mises de côté,
la version shapeless de `myPoly` a la même fonctionalité
que notre version de démonstration.
Nous pouvons l'appeler avec un paramètre de type `Int` ou `String` et
obtenir en retour le type correspondant :

```tut:book
myPoly.apply(123)
myPoly.apply("hello")
```

Shapeless supporte aussi les `Polys` avec plus d'un paramètre.
En voici un exemple avec deux paramètres :

```tut:book:silent
object multiply extends Poly2 {
  implicit val intIntCase: Case.Aux[Int, Int, Int] =
    at((a, b) => a * b)

  implicit val intStrCase: Case.Aux[Int, String, String] =
    at((a, b) => b.toString * a)
}
```

```tut:book
multiply(3, 4)
multiply(3, "4")
```

Les `Cases` n'étant que des valeurs implicites,
nous pouvons définir des cases basés sur des type classes
et ainsi appliquer toutes les résolutions implicites sophistiquées que
nous avions traitées dans le chapitre précédent.

Voici un exemple simple qui donne
le total de nombre selon différents contextes :

```tut:book:silent
import scala.math.Numeric

object total extends Poly1 {
  implicit def base[A](implicit num: Numeric[A]):
      Case.Aux[A, Double] =
    at(num.toDouble)

  implicit def option[A](implicit num: Numeric[A]):
      Case.Aux[Option[A], Double] =
    at(opt => opt.map(num.toDouble).getOrElse(0.0))

  implicit def list[A](implicit num: Numeric[A]):
      Case.Aux[List[A], Double] =
    at(list => num.toDouble(list.sum))
}
```

```tut:book
total(10)
total(Option(20.0))
total(List(1L, 2L, 3L))
```

<div class="callout callout-warning">
*Particularité de l'inférence de type*

`Poly` amène l'inférence de type de Scala en dehors de sa zone de confort.
Nous pouvons facilement embrouiller le compilateur en lui demandant trop d'inférences de type d'un seul coup.
Par exemple, le code suivant compile correctement :

```tut:book:silent
val a = myPoly.apply(123)
val b: Double = a
```
Par contre, combiner les deux linges précédentes provoque une erreur :

```tut:book:fail
val a: Double = myPoly.apply(123)
```

Si on ajoute une annotation de type, le code compile à nouveau :

```tut:book
val a: Double = myPoly.apply[Int](123)
```

Ce comportement est ennuyeux et peut porter à confusion.
Malheureusement, il n'existe pas de règles précises pour éviter ces problèmes.
La règle générale est d'essayer de ne pas trop contraindre le compilateur,
de résoudre une seule contrainte à la fois et de lui donner
un coup de pouce quand il se retrouve bloqué.
</div>
