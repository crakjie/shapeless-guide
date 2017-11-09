## Déduire des instances pour les produits {#sec:generic:products}

Dans cette section nous utilisons shapeless pour
déduire des instances de *types classes* pour des types de produits.
(*ie.* *case classes*).

Utilisons ces deux idées :

1. Si l'on a une instance de type *class*
   pour la tête et la queue d'une `HList`,
   on peut en déduire l'instance de toute la `HList`.

2. Si nous avons une case class `A`, un `Generic[A]`
   et une instance de type class pour le `Repr` de ce générique,
   nous pouvons alors les combiner pour obtenir une instance de `A`.

Prenons `CsvEncoder` et `IceCream` comme exemples :

 - `IceCream` a un `Repr` générique de type
   `String :: Int :: Boolean :: HNil`.

 - Le `Repr` est fait d'une
    `String`, d'un `Int`, d'un `Boolean` et d'une `HNil`.
   Si nous avons un  `CsvEncoder` pour ces types alors nous
   disposons d'un encodeur pour le tout.

 - Si nous pouvons déduire un `CsvEncoder` pour le `Repr`,
   nous pouvons en créer un pour `IceCream`.

```tut:book:invisible
// ----------------------------------------------
// Forward definitions

trait CsvEncoder[A] {
  def encode(value: A): List[String]
}

object CsvEncoder {
  def apply[A](implicit enc: CsvEncoder[A]): CsvEncoder[A] =
    enc
}

def writeCsv[A](values: List[A])(implicit encoder: CsvEncoder[A]): String =
  values.map(encoder.encode).map(_.mkString(",")).mkString("\n")

case class IceCream(name: String, numCherries: Int, inCone: Boolean)

val iceCreams: List[IceCream] = List(
  IceCream("Sundae", 1, false),
  IceCream("Cornetto", 0, true),
  IceCream("Banana Split", 0, false)
)

case class Employee(name: String, number: Int, manager: Boolean)

val employees: List[Employee] = List(
  Employee("Bill", 1, true),
  Employee("Peter", 2, false),
  Employee("Milton", 3, false)
)
// ----------------------------------------------
```

### Les instances de *HLists*

Commençons par définir les constructeurs d'instance de `CsvEncoders` pour
  `String`, `Int` et `Boolean`:

```tut:book:silent
def createEncoder[A](func: A => List[String]): CsvEncoder[A] =
  new CsvEncoder[A] {
    def encode(value: A): List[String] = func(value)
  }

implicit val stringEncoder: CsvEncoder[String] =
  createEncoder(str => List(str))

implicit val intEncoder: CsvEncoder[Int] =
  createEncoder(num => List(num.toString))

implicit val booleanEncoder: CsvEncoder[Boolean] =
  createEncoder(bool => List(if(bool) "yes" else "no"))
```

Nous pouvons combiner ces blocs de construction
pour créer un encodeur pour notre `HList`.
Nous utiliserons deux règles :
une pour  `HNil` et une pour `::` comme illustré ci-dessous :

```tut:book:silent
import shapeless.{HList, ::, HNil}

implicit val hnilEncoder: CsvEncoder[HNil] =
  createEncoder(hnil => Nil)

implicit def hlistEncoder[H, T <: HList](
  implicit
  hEncoder: CsvEncoder[H],
  tEncoder: CsvEncoder[T]
): CsvEncoder[H :: T] =
  createEncoder {
    case h :: t =>
      hEncoder.encode(h) ++ tEncoder.encode(t)
  }
```

Prises toutes ensemble, ces cinq instances nous
permettent d'invoquer un `CsvEncoders` pour toute `HList`
impliquant `Strings`, `Ints` et `Booleans`:

```tut:book:silent
val reprEncoder: CsvEncoder[String :: Int :: Boolean :: HNil] =
  implicitly
```

```tut:book
reprEncoder.encode("abc" :: 123 :: true :: HNil)
```

### Des instance pour nos produits {#sec:generic:product-generic}

On peut combiner nos règles de déduction de `HList`s avec
nos instances de `Generic`
pour produire un `CsvEncoder` pour `IceCream`:

```tut:book:silent
import shapeless.Generic

implicit val iceCreamEncoder: CsvEncoder[IceCream] = {
  val gen = Generic[IceCream]
  val enc = CsvEncoder[gen.Repr]
  createEncoder(iceCream => enc.encode(gen.to(iceCream)))
}
```
et l'utiliser comme suit :

```tut:book
writeCsv(iceCreams)
```
Mais cette solution est spécifique à `IceCream`.
Idéalement on voudrait définir une
seule règle pour toutes les *case classes*
qui ont un `Generic` et le `CsvEncoder` correspondant.
Montrons pas à pas comment faire cette déduction.
Voici la première étape :

```scala
implicit def genericEncoder[A](
  implicit
  gen: Generic[A],
  enc: CsvEncoder[???]
): CsvEncoder[A] = createEncoder(a => enc.encode(gen.to(a)))
```
Nous devons choisir le type à placer à la place de `???`.
Mais le problème est que l'on ne peut
pas utiliser le type `Repr` associé avec `gen`,
la solution suivante n'est pas possible :

```tut:book:fail
implicit def genericEncoder[A](
  implicit
  gen: Generic[A],
  enc: CsvEncoder[gen.Repr]
): CsvEncoder[A] =
  createEncoder(a => enc.encode(gen.to(a)))
```

Nous avons ici un problème de scope :
On ne peut faire référence à un membre de type d'un paramètre
à partir d'un autre paramètre du même bloc.
L'astuce pour contourner ce problème
est d'ajouter un nouveau paramètre de type à notre méthode,
et y faire référence dans chacune des valeurs associées :

```tut:book:silent
implicit def genericEncoder[A, R](
  implicit
  gen: Generic[A] { type Repr = R },
  enc: CsvEncoder[R]
): CsvEncoder[A] =
  createEncoder(a => enc.encode(gen.to(a)))
```

Nous traiterons ce style d'écriture dans le chapitre suivant.
Maintenant, cette définiton compile et fonctionne comme attendu avec
n'importe quelle case class.
Intuitivement, la définition nous dit :

> *Avec un `A` donné et une `HList` de type `R`,
> avec un implicite `Generic` qui relie `A` a `R`
> et un `CsvEncoder` pour `R`,
> alors on crée un `CsvEncoder` pour `A`.*

Nous avons maintenant un système complet qui peut gérer n'importe quelle case class.

L'appel suivant

```tut:book:silent
writeCsv(iceCreams)

```

est résolu par le compilateur comme suit :

```tut:book:silent
writeCsv(iceCreams)(
  genericEncoder(
    Generic[IceCream],
    hlistEncoder(stringEncoder,
      hlistEncoder(intEncoder,
        hlistEncoder(booleanEncoder, hnilEncoder)))))
```

et il peut inférer les valeurs correctes pour
un grand nombre de types différents.
Tout comme moi,
je suis sur que vous appréciez ne pas avoir à écrire ce code à la main !

<div class="callout callout-info">
*l'alias de type Aux*

Les types refinements comme  `Generic[A] { type Repr = L }`
sont verbeux et difficiles à lire,
c'est la raison pour laquelle shapeless fournit un alias de type, `Generic.Aux`,
pour reformuler le membre de type en un paramètre de type :

```scala
package shapeless

object Generic {
  type Aux[A, R] = Generic[A] { type Repr = R }
}
```
Avec l'alias de type la définition devient beaucoup plus lisible :

```tut:book:silent
implicit def genericEncoder[A, R](
  implicit
  gen: Generic.Aux[A, R],
  env: CsvEncoder[R]
): CsvEncoder[A] =
  createEncoder(a => env.encode(gen.to(a)))
```

Notez bien que le type `Aux` ne change pas la sémantique,
cela rend juste les chose plus faciles à lire.
Le pattern `Aux` est souvent utilisé dans le code de shapeless.

</div>

### Alors quels en sont les inconvénients ?

Si tout ce que l'on vient de voir semble magique,
permettez-moi de vous ramener à la réalité.
Si quelque-chose tourne mal, le compilateur ne vous sera pas d'une grande aide.

Il existe deux raisons pour laquelle le code précédent pourrait ne pas compiler.
La première est si le compilateur ne peut pas trouver l'instance de `Generic`.
Par exemple, si nous essayons d'appeler `writeCsv` avec une
simple classe :

```tut:book:silent
class Foo(bar: String, baz: Int)
```

```tut:book:fail
writeCsv(List(new Foo("abc", 123)))
```

Dans ce cas le message est relativement simple à comprendre.
Si shapeless ne peut calculer un `Generic` cela veut dire que le type en question n'est pas un *ADT*
(il y a quelque-part dans l'algèbre un type qui n'est pas une case class ou un trait scellé).

L'autre source potentielle d'erreur
survient lorsque le compilateur ne peut calculer un
`CsvEncoder` pour notre `HList`.
Cela arrive normalement car l'on n'a pas
d'encodeur pour un des champs de notre *ADT*.
Par exemple nous n'avons pas encore défini de
`CsvEncoder` pour `java.util.Date`,
donc le code suivant ne fonctionne pas :

```tut:book:silent
import java.util.Date

case class Booking(room: String, date: Date)
```

```tut:book:fail
writeCsv(List(Booking("Lecture hall", new Date())))
```

Le messsage d'erreur ne nous aide pas vraiment.
Tout ce que le compilateur sait,
c'est qu'il a essayé un grand nombre de combinaisons d'implicites
et qu'aucune ne fonctionnait.
Il n'a aucune idée de quelle combinaison était la plus proche de celle attendue,
donc il ne peut nous dire où se trouve la source du problème.

Il n'y a pas de quoi se réjouir ici.
Nous devons trouver nous-même la source
des erreurs par un processus d'élimination.
Nous aborderons les techniques de
debuggage dans la Section [@sec:generic:debugging].
Pour l'instant la seule fonctionalité qui compense c'est que
la résolution d'implicite plantera toujours à la compilation.
Il y a une petite chance que cela finisse
par produire du code qui plante durant l'exécution.
