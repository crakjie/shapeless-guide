## Déduire l'instance d'un produit avec *LabelledGeneric*

Nous allons utiliser un exemple fonctionnel
d'encodage de Json pour illustrer `LabelledGeneric`.
Nous allons définir une type classe `JsonEncoder`
qui va convertir nos valeurs en *AST JSON*.
C'est l'approche adoptée par
*Argonaut*, *Circe*, *Play JSON*, *Spray JSON*
et de nombreuses autres bibliothèques Scala pour le *JSON*.

Tout d'abord, définissons le type de données de notre *JSON* :

```tut:book:silent
sealed trait JsonValue
case class JsonObject(fields: List[(String, JsonValue)]) extends JsonValue
case class JsonArray(items: List[JsonValue]) extends JsonValue
case class JsonString(value: String) extends JsonValue
case class JsonNumber(value: Double) extends JsonValue
case class JsonBoolean(value: Boolean) extends JsonValue
case object JsonNull extends JsonValue
```

puis la *type class* pour encoder les valeurs en *JSON* :

```tut:book:silent
trait JsonEncoder[A] {
  def encode(value: A): JsonValue
}

object JsonEncoder {
  def apply[A](implicit enc: JsonEncoder[A]): JsonEncoder[A] = enc
}
```

puis quelques instances primitives :

```tut:book:silent
def createEncoder[A](func: A => JsonValue): JsonEncoder[A] =
  new JsonEncoder[A] {
    def encode(value: A): JsonValue = func(value)
  }

implicit val stringEncoder: JsonEncoder[String] =
  createEncoder(str => JsonString(str))

implicit val doubleEncoder: JsonEncoder[Double] =
  createEncoder(num => JsonNumber(num))

implicit val intEncoder: JsonEncoder[Int] =
  createEncoder(num => JsonNumber(num))

implicit val booleanEncoder: JsonEncoder[Boolean] =
  createEncoder(bool => JsonBoolean(bool))
```

et quelques combinateurs d'instance :

```tut:book:silent
implicit def listEncoder[A]
    (implicit enc: JsonEncoder[A]): JsonEncoder[List[A]] =
  createEncoder(list => JsonArray(list.map(enc.encode)))

implicit def optionEncoder[A]
    (implicit enc: JsonEncoder[A]): JsonEncoder[Option[A]] =
  createEncoder(opt => opt.map(enc.encode).getOrElse(JsonNull))
```

Idéalement, lorsqu'on encode un *ADT* en *JSON*,
on voudrait utiliser les noms des champs dans la sortie *JSON* :

```tut:book:silent
case class IceCream(name: String, numCherries: Int, inCone: Boolean)

val iceCream = IceCream("Sundae", 1, false)

// Idéalement, on voudrait produire quelque chose comme suit :
val iceCreamJson: JsonValue =
  JsonObject(List(
    "name"        -> JsonString("Sundae"),
    "numCherries" -> JsonNumber(1),
    "inCone"      -> JsonBoolean(false)
  ))
```

C'est à ce moment que `LabelledGeneric` devient utile.
Invoquons une instance de `IceCream` et regardons sa représentation :

```tut:book:silent
import shapeless.LabelledGeneric
```

```tut:book
val gen = LabelledGeneric[IceCream].to(iceCream)
```

Pour plus de clareté voici le type complet de la `HList`:

```scala
// String  with KeyTag[Symbol with Tagged["name"], String]     ::
// Int     with KeyTag[Symbol with Tagged["numCherries"], Int] ::
// Boolean with KeyTag[Symbol with Tagged["inCone"], Boolean]  ::
// HNil
```

Ce type est un peu plus complexe que ce que nous avons vu jusqu'à présent.
Plutôt que représenter les noms des champs avec des types string littéraux,
shapeless les représente avec des symbôles taggés grâce à des types littéraux.
Les détails d'implémentation ne sont pas particulièrement importants :
on peut toujours utiliser `Witness` et `FieldType` pour extraire les tags,
mais ils ressortiront en `Symbols` et non pas en `Strings`[^future-tags].

[^future-tags]: Les futures versions de shapeless pourraient utiliser les `Strings` comme tags.

### Les instances de *HLists*

Définissons un instance de `JsonEncoder` pour `HNil` et `::`.
Nos encodeurs vont générer et manipuler des `JsonObjects`,
nous allons donc créer un nouveau type d'encodeur pour nous faciliter la vie :

```tut:book:silent
trait JsonObjectEncoder[A] extends JsonEncoder[A] {
  def encode(value: A): JsonObject
}

def createObjectEncoder[A](fn: A => JsonObject): JsonObjectEncoder[A] =
  new JsonObjectEncoder[A] {
    def encode(value: A): JsonObject =
      fn(value)
  }
```

La définition de `HNil` devient donc triviale :

```tut:book:silent
import shapeless.{HList, ::, HNil, Lazy}

implicit val hnilEncoder: JsonObjectEncoder[HNil] =
  createObjectEncoder(hnil => JsonObject(Nil))
```

La définitoon de `hlistEncoder` ajoute un peu de complexité,
nous allons donc avancer pas à pas.
Commençons avec la définition à laquelle
on pourrait s'attendre si nous utilisions un `Generic`:

```tut:book:silent
implicit def hlistObjectEncoder[H, T <: HList](
  implicit
  hEncoder: Lazy[JsonEncoder[H]],
  tEncoder: JsonObjectEncoder[T]
): JsonEncoder[H :: T] = ???
```


`LabelledGeneric` nous donne une `HList` qui contient des types taggés,
commençons donc par introduire une nouvelle variable de types pour le type de la clef :

```tut:book:silent
import shapeless.Witness
import shapeless.labelled.FieldType

implicit def hlistObjectEncoder[K, H, T <: HList](
  implicit
  hEncoder: Lazy[JsonEncoder[H]],
  tEncoder: JsonObjectEncoder[T]
): JsonObjectEncoder[FieldType[K, H] :: T] = ???
```

Dans le corps de notre méthode, nous aurons
besoin de la valeur associée à `K`.
Nous allons ajouter un `Witness` implicite qui le fera pour nous :

```tut:book:silent
implicit def hlistObjectEncoder[K, H, T <: HList](
  implicit
  witness: Witness.Aux[K],
  hEncoder: Lazy[JsonEncoder[H]],
  tEncoder: JsonObjectEncoder[T]
): JsonObjectEncoder[FieldType[K, H] :: T] = {
  val fieldName = witness.value
  ???
}
```

On peut accéder à la valeur de `K` en utilisant `witness.value`,
mais le compilateur n'a aucun moyen de savoir quel type de tag nous allons obtenir.
`LabelledGeneric` utilise `Symbols` pour les tags, nous allons donc mettre un type bound a `K`
et utiliser `symbol.name` pour le convertir en `String`:

```tut:book:silent
implicit def hlistObjectEncoder[K <: Symbol, H, T <: HList](
  implicit
  witness: Witness.Aux[K],
  hEncoder: Lazy[JsonEncoder[H]],
  tEncoder: JsonObjectEncoder[T]
): JsonObjectEncoder[FieldType[K, H] :: T] = {
  val fieldName: String = witness.value.name
  ???
}
```

Le reste de la définition utilise les principes
que nous avons abordés dans le chapitre [@sec:generic]:

```tut:book:silent
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

```

### Les instances pour les produits concrets.

Enfin, revenons à notre instance générique.
Elle est identique aux définitions que nous avons vues précédemment,
à l'exception que nous utilisons `LabelledGeneric` à la place de `Generic`:

```tut:book:silent
import shapeless.LabelledGeneric

implicit def genericObjectEncoder[A, H](
  implicit
  generic: LabelledGeneric.Aux[A, H],
  hEncoder: Lazy[JsonObjectEncoder[H]]
): JsonEncoder[A] =
  createObjectEncoder { value =>
    hEncoder.value.encode(generic.to(value))
  }
```

Et voilà tout ce dont nous avons besoin !
Avec ces définitions en place nous pouvons
sérialiser les instances de n'importe quelle case class
et conserver les noms des champs dans le *JSON* obtenu :

```tut:book
JsonEncoder[IceCream].encode(iceCream)
```
