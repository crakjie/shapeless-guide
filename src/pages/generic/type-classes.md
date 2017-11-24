## Bref rappel : les type classes {#sec:generic:type-classes}

Avant d'entrer dans les détails de la déduction d'instance,
faisons un bref rappel des aspects importants des *types classes*.

Les type classes sont un pattern de programmation emprunté à *Haskell*
(le mot « classe » n'a rien a voir avec les classes
de la programmation orientée object).
Nous les écrivons en Scala avec des traits et des implicites.
Une *type class* est un trait paramétré représentant une
fonctionalité générale que l'on voudrait appliquer à une grande
variété de types :

```tut:book:silent
// Turn a value of type A into a row of cells in a CSV file:
trait CsvEncoder[A] {
  def encode(value: A): List[String]
}
```
On implémente notre type class avec une *instance*
pour chaque type visé.
Si nous voulons avoir l'instance dans le scope automatiquement,
on peut les placer dans l'objet compagnon de la type class.
Sinon, on peut les placer dans une objet séparé qui fera office de bibliothèque
et que l'utilisateur pourra importer manuellement :

```tut:book:silent
// Custom data type:
case class Employee(name: String, number: Int, manager: Boolean)

// CsvEncoder instance for the custom data type:
implicit val employeeEncoder: CsvEncoder[Employee] =
  new CsvEncoder[Employee] {
    def encode(e: Employee): List[String] =
      List(
        e.name,
        e.number.toString,
        if(e.manager) "yes" else "no"
      )
  }
```
On marque chaque instance avec le mot clé `implicit`,
et on définit une ou plusieurs méthodes qui acceptent le paramètre
implicite du type de notre type class, elles feront office de point d'entrée :

```tut:book:silent
def writeCsv[A](values: List[A])(implicit enc: CsvEncoder[A]): String =
  values.map(value => enc.encode(value).mkString(",")).mkString("\n")
```
Testons `writeCsv` avec quelques données de test :

```tut:book:silent
val employees: List[Employee] = List(
  Employee("Bill", 1, true),
  Employee("Peter", 2, false),
  Employee("Milton", 3, false)
)
```
Quand on appelle `writeCsv`,
le compilateur calcule la valeur du paramètre de type
et cherche un implicite de `CsvEncoder` du type correspondant :


```tut:book
writeCsv(employees)
```
On peut utiliser `writeCsv` avec nimporte quel type de donnée,
du moment que l'on a dans le scope l'implicite de `CsvEncoder` correspondant :

```tut:book:silent
case class IceCream(name: String, numCherries: Int, inCone: Boolean)

implicit val iceCreamEncoder: CsvEncoder[IceCream] =
  new CsvEncoder[IceCream] {
    def encode(i: IceCream): List[String] =
      List(
        i.name,
        i.numCherries.toString,
        if(i.inCone) "yes" else "no"
      )
  }

val iceCreams: List[IceCream] = List(
  IceCream("Sundae", 1, false),
  IceCream("Cornetto", 0, true),
  IceCream("Banana Split", 0, false)
)
```

```tut:book
writeCsv(iceCreams)
```

### Résoudre les instances

Les *types classes* sont très flexibles mais elles nous imposent
de définir une instance pour
chaque type qui nous intéresse.
Heureusement, le compilateur de Scala a plus d'un tour dans son sac,
si on lui donne certaines regles, il est capable de résoudre les instances pour nous.  
Par exemple, on peut ecrire une règle qui créé un `CsvEncoder` pour `(A, B)` pour
un `CsvEncoder` pour `A` et un pour `B` donnés :

```tut:book:silent
implicit def pairEncoder[A, B](
  implicit
  aEncoder: CsvEncoder[A],
  bEncoder: CsvEncoder[B]
): CsvEncoder[(A, B)] =
  new CsvEncoder[(A, B)] {
    def encode(pair: (A, B)): List[String] = {
      val (a, b) = pair
      aEncoder.encode(a) ++ bEncoder.encode(b)
    }
  }
```

Quand tous les paramètres d'un `implicit def`
sont eux meme marquer `implicit`,
alors le compilateur peut l'utiliser comme une règle de résolution
pour crée des instance a partire d'autres instances.
Par exemple, si l'on appel `writeCsv`
et qu'on lui passe une `List[(Employee, IceCream)]`,
le compilateur est capable de combiner
`pairEncoder`, `employeeEncoder` et `iceCreamEncoder`
pour produire le `CsvEncoder[(Employee, IceCream)]` requis:

```tut:book
writeCsv(employees zip iceCreams)
```

À partir d'une liste de règles écrites à partir d'
`implicit val`s et d'`implicit def`s,
le compilateur est capable de *rechercher* les combinaisons
pour retourner l'instance requise.


Cette fonctionalité, connue sous le nom de *résolution d'`implicit`s*,
est ce qui rend le pattern des *types classes* si puissant en Scala.

Même avec cette puissance, le compilateur
ne peut démanteler nos *case classes* et *sealed traits*.
On est tenu de définir à la main les instances de nos *ADT*s.
Les représentations génériques de shapeless changent la donne,
car ils nous permettent de déduire automatiquement les instances de nimporte quel *ADT*.

### Les définitions de type class idiomatique {#sec:generic:idiomatic-style}

Le style généralement accépter pour la définition d'une type classe idiomatique
Il est généralement accépte d'inclure un objet compagon contenant certaines methode standar
lors de la définition d'une type classe idiomatique :

```tut:book:silent
object CsvEncoder {
  // "Summoner" method
  def apply[A](implicit enc: CsvEncoder[A]): CsvEncoder[A] =
    enc

  // "Constructor" method
  def instance[A](func: A => List[String]): CsvEncoder[A] =
    new CsvEncoder[A] {
      def encode(value: A): List[String] =
        func(value)
    }

  // Globally visible type class instances
}
```

La methode `apply` connue sous le nom de *summoner* ou *materializer*,
nous permet d'invoquer une instance de type class selon un type donnée:


```tut:book
CsvEncoder[IceCream]
```
Dans les cas les plus simple le *summoner* fait la meme chose
 que la méhode `implicitly` définie dans `scala.Predef`:

```tut:book
implicitly[CsvEncoder[IceCream]]
```
Cependant, comme nous le verrons dans la Section [@sec:type-level-programming:depfun],
lors que l'on travaille avec shapeless il arrive que
la methode `implicitly` n'infère pas les types correctement.
On peut toujours définir une methode *summoner* pour avoir le bon comportement,
donc cela vaut le coup d'en écrire une pour chaque *type class* que l'on créé.
On peut aussi utiliser une methode de shapeless appelée "`the`"
(nous y reviendrons):

```tut:book:silent
import shapeless._
```

```tut:book
the[CsvEncoder[IceCream]]
```
La methode `instance` parfois appelé `pure`,
fournis une syntaxe simple pour crée de nouvelle instance de type classe,
réduisant au passage le boilerplate pour les classes anoymes:


```tut:book:silent
implicit val booleanEncoder: CsvEncoder[Boolean] =
  new CsvEncoder[Boolean] {
    def encode(b: Boolean): List[String] =
      if(b) List("yes") else List("no")
  }
```

Est réduit en:

```tut:book:invisible
import CsvEncoder.instance
```

```tut:book:silent
implicit val booleanEncoder: CsvEncoder[Boolean] =
  instance(b => if(b) List("yes") else List("no"))
```
Malheureusement,
les limitations imposées par le livre
nous empèchent d'écrire un grand singleton
contenant beaucoup de méthodes et d'instances.
Nous préférons donc décrire les définitons en
dehors de leurs *object* compagnon.
Ceci est à garder a l'esprit lorsque que vous lisez ce livre
mais rappelez-vous que code complet se trouve dans le dépôt mentionné dans la Section [@sec:intro:about-this-book]
