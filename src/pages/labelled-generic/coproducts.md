## Déduire les instances de coproduits avec *LabelledGeneric*

```tut:book:invisible:reset
// ----------------------------------------------
// définitions précédentes:

sealed trait JsonValue
final case class JsonObject(fields: List[(String, JsonValue)]) extends JsonValue
final case class JsonArray(items: List[JsonValue]) extends JsonValue
final case class JsonString(value: String) extends JsonValue
final case class JsonNumber(value: Double) extends JsonValue
final case class JsonBoolean(value: Boolean) extends JsonValue
case object JsonNull extends JsonValue

trait JsonEncoder[A] {
  def encode(value: A): JsonValue
}

object JsonEncoder {
  def apply[A](implicit enc: JsonEncoder[A]): JsonEncoder[A] =
    enc
}

def createEncoder[A](func: A => JsonValue): JsonEncoder[A] =
  new JsonEncoder[A] {
    def encode(value: A): JsonValue =
      func(value)
  }

implicit val stringEncoder: JsonEncoder[String] =
  createEncoder(str => JsonString(str))

implicit val doubleEncoder: JsonEncoder[Double] =
  createEncoder(num => JsonNumber(num))

implicit val intEncoder: JsonEncoder[Int] =
  createEncoder(num => JsonNumber(num))

implicit val booleanEncoder: JsonEncoder[Boolean] =
  createEncoder(bool => JsonBoolean(bool))

implicit def listEncoder[A](implicit encoder: JsonEncoder[A]): JsonEncoder[List[A]] =
  createEncoder(list => JsonArray(list.map(encoder.encode)))

implicit def optionEncoder[A](implicit encoder: JsonEncoder[A]): JsonEncoder[Option[A]] =
  createEncoder(opt => opt.map(encoder.encode).getOrElse(JsonNull))

trait JsonObjectEncoder[A] extends JsonEncoder[A] {
  def encode(value: A): JsonObject
}

def createObjectEncoder[A](func: A => JsonObject): JsonObjectEncoder[A] =
  new JsonObjectEncoder[A] {
    def encode(value: A): JsonObject =
      func(value)
  }

import shapeless.{HList, ::, HNil, Witness, Lazy},
       shapeless.labelled.FieldType

implicit val hnilObjectEncoder: JsonObjectEncoder[HNil] =
  createObjectEncoder(hnil => JsonObject(Nil))

implicit def hlistObjectEncoder[K <: Symbol, H, T <: HList](
  implicit
  witness: Witness.Aux[K],
  hEncoder: Lazy[JsonEncoder[H]],
  tEncoder: JsonObjectEncoder[T]
): JsonObjectEncoder[FieldType[K, H] :: T] = {
  val fieldName: String = witness.value.name
  createObjectEncoder { hlist =>
    val head = hEncoder.value.encode(hlist.head)
    val tail = tEncoder.encode(hlist.tail)
    JsonObject((fieldName, head) :: tail.fields)
  }
}

import shapeless.LabelledGeneric

implicit def genericObjectEncoder[A, H](
  implicit
  generic: LabelledGeneric.Aux[A, H],
  hEncoder: Lazy[JsonObjectEncoder[H]]
): JsonObjectEncoder[A] =
  createObjectEncoder(value => hEncoder.value.encode(generic.to(value)))

sealed trait Shape
final case class Rectangle(width: Double, height: Double) extends Shape
final case class Circle(radius: Double) extends Shape

// ----------------------------------------------
```

Appliquer les `LabelledGeneric` aux `Coproduct`s revient à
mélanger des concepts que nous avons déja traités.
Commençons par examiner un type `Coproduct` déduit par `LabelledGeneric`.
Nous allons revisiter notre ADT `Shape` du Chapitre [@sec:generic]:

```tut:book:silent
import shapeless.LabelledGeneric

sealed trait Shape
final case class Rectangle(width: Double, height: Double) extends Shape
final case class Circle(radius: Double) extends Shape
```

```tut:book
LabelledGeneric[Shape].to(Circle(1.0))
```

Voici le type du `Coproduct` dans un format plus lisible :

```scala
// Rectangle with KeyTag[Symbol with Tagged["Rectangle"], Rectangle] :+:
// Circle    with KeyTag[Symbol with Tagged["Circle"],    Circle]    :+:
// CNil
```

Comme vous pouvez le constater, le résultat est un `Coproduit` des sous-types de `Shape`,
chacun taggé avec leur nom.
Nous pouvons utiliser ces informations pour écrire un `JsonEncoders` pour `:+:` et `CNil`:

```tut:book:silent
import shapeless.{Coproduct, :+:, CNil, Inl, Inr, Witness, Lazy}
import shapeless.labelled.FieldType

implicit val cnilObjectEncoder: JsonObjectEncoder[CNil] =
  createObjectEncoder(cnil => throw new Exception("Inconceivable!"))

implicit def coproductObjectEncoder[K <: Symbol, H, T <: Coproduct](
  implicit
  witness: Witness.Aux[K],
  hEncoder: Lazy[JsonEncoder[H]],
  tEncoder: JsonObjectEncoder[T]
): JsonObjectEncoder[FieldType[K, H] :+: T] = {
  val typeName = witness.value.name
  createObjectEncoder {
    case Inl(h) =>
      JsonObject(List(typeName -> hEncoder.value.encode(h)))

    case Inr(t) =>
      tEncoder.encode(t)
  }
}
```

`coproductEncoder` suit les mêmes règles que `hlistEncoder`.
On dispose de trois paramètres de types :
`K` pour le nom du type,
`H` pour la valeur de la tête de la `HList`
et `T` pour la valeur de la queue.
Nous utilisons `FieldType` et `:+:` dans le type résultant
pour garder la relation entre les trois, et nous utilisons `Witness`
pour accéder à la valeur du nom du type pendant l'exécution.
Le résultat est un objet contenant une seule paire de clef/valeur,
la clef étant le nom du type et la valeur le résultat :


```tut:book:silent
val shape: Shape = Circle(1.0)
```

```tut:book
JsonEncoder[Shape].encode(shape)
```

D'autres encodages sont possibles avec un peu plus de travail.
Par exemple, on peut ajouter un champ `"type"` dans la sortie,
ou encore permetre à l'utilisateur de configurer le format.
La code base de Sam Halliday sur [spray-json-shapeless][link-spray-json-shapeless] est un parfait exemple de code qui est accessible tout en offrant beaucoup de flexibilité.
