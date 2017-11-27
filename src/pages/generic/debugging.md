## Débugger les résolutions d'implicites {#sec:generic:debugging}

```tut:book:invisible
import shapeless._

trait CsvEncoder[A] {
  val width: Int
  def encode(value: A): List[String]
}

object CsvEncoder {
  def apply[A](implicit enc: CsvEncoder[A]): CsvEncoder[A] =
    enc
}

def createEncoder[A](w: Int)(func: A => List[String]): CsvEncoder[A] =
  new CsvEncoder[A] {
    val width = w
    def encode(value: A): List[String] =
      func(value)
  }

implicit val stringEncoder: CsvEncoder[String] =
  createEncoder(1)(str => List(str))

implicit val intEncoder: CsvEncoder[Int] =
  createEncoder(1)(num => List(num.toString))

implicit val doubleEncoder: CsvEncoder[Double] =
  createEncoder(1)(num => List(num.toString))

implicit val booleanEncoder: CsvEncoder[Boolean] =
  createEncoder(1)(bool => List(if(bool) "yes" else "no"))

implicit def optionEncoder[A](implicit encoder: CsvEncoder[A]): CsvEncoder[Option[A]] =
  createEncoder(encoder.width)(opt => opt.map(encoder.encode).getOrElse(List.fill(encoder.width)("")))

implicit val hnilEncoder: CsvEncoder[HNil] =
  createEncoder(0)(hnil => Nil)

implicit def hlistEncoder[H, T <: HList](
  implicit
  hEncoder: Lazy[CsvEncoder[H]],
  tEncoder: CsvEncoder[T]
): CsvEncoder[H :: T] =
  createEncoder(hEncoder.value.width + tEncoder.width) {
    case h :: t =>
      hEncoder.value.encode(h) ++ tEncoder.encode(t)
  }

implicit val cnilEncoder: CsvEncoder[CNil] =
  createEncoder(0)(cnil => ???)

implicit def coproductEncoder[H, T <: Coproduct](
  implicit
  hEncoder: Lazy[CsvEncoder[H]],
  tEncoder: CsvEncoder[T]
): CsvEncoder[H :+: T] =
  createEncoder(hEncoder.value.width + tEncoder.width) {
    case Inl(h) => hEncoder.value.encode(h) ++ List.fill(tEncoder.width)("")
    case Inr(t) => List.fill(hEncoder.value.width)("") ++ tEncoder.encode(t)
  }

implicit def genericEncoder[A, R](
  implicit
  gen: Generic.Aux[A, R],
  enc: Lazy[CsvEncoder[R]]
): CsvEncoder[A] =
  createEncoder(enc.value.width)(a => enc.value.encode(gen.to(a)))
```

Les erreurs dans la résolution
d'implicites peuvent être déconcertantes et frustrantes.
Voici quelques techniques à utiliser quand les choses vont mal.

### Débugger en utilisant *implicitly*

Que peut-on faire quand les compilateurs
ne trouvent simplement pas la valeur implicite ?
L'erreur peut être causée par la résolution de n'importe quel implicit.
Par exemple :

```tut:book:silent
case class Foo(bar: Int, baz: Float)
```

```tut:book:fail
CsvEncoder[Foo]
```
Nous avons une erreur car nous navons
pas défini `CsvEncoder` pour `Float`.
Pourtant, cela n'est peut-être pas évident à voir dans le code de l'application.
On peut chercher l'erreur en imaginant comment l'implicite est censé se développer,
insérer des appels à `CsvEncoder.apply` ou à `implicitly`
au-dessus de l'erreur pour voir si cela compile.
Commençons avec la représentation générique de `Foo`:


```tut:book:fail
CsvEncoder[Int :: Float :: HNil]
```
L'erreur ici nous en dit un peu plus, on doit continuer à
chercher plus profondément dans la chaînes d'appel des implicites.
L'étape suivante est de tester les composants de `HList` :

```tut:book:silent
CsvEncoder[Int]
```

```tut:book:fail
CsvEncoder[Float]
```

`Int` fonctionne mais `Float` lève une erreur.
`CsvEncoder[Float]` est une feuille dans le développement de notre abre,
on sait donc que l'on doit commencer par implémenter l'instance manquante.
Si ajouter l'instance ne corrige pas le problème,
on répète le processus pour trouver le prochain point problématique.

### Debugger en utilisant *reify*

La méthode `reify` de `scala.reflect` prend une
expression Scala en paramètre et retourne
un objet AST représentant l'expression sous
forme d'abre avec toutes les annotations de types.

```tut:book:silent
import scala.reflect.runtime.universe._
```

```tut:book
println(reify(CsvEncoder[Int]))
```

Les types inferés pendant la résolution d'implicite
peuvent nous donner un indice sur le problème.
Après la résolution d'implicite,
tous les types existentiels restants comme `A` ou `T`
indiquent que quelque-chose n'a pas bien fonctionné.
Les types "top" et "bottom" comme `Any` and `Nothing`
sont aussi la preuve d'une erreur.
