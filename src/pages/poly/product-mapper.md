## Définir une type classe utilisant *Poly* {#sec:poly:product-mapper}

Nous pouvons utiliser `Poly` et les *type classes*
en tant que `Mapper` et un `FlatMapper` comme
brique d'assemblage pour nos propres *type classes*.
Comme exemple, construisons une type classe
pour transformer une case classe en une autre :

```tut:book:silent
trait ProductMapper[A, B, P] {
  def apply(a: A): B
}
```

Nous pouvons créer une instance de `ProductMapper`
en utilisant `Mapper` et une paire de `Generics` :

```tut:book:silent
import shapeless._
import shapeless.ops.hlist

implicit def genericProductMapper[
  A, B,
  P <: Poly,
  ARepr <: HList,
  BRepr <: HList
](
  implicit
  aGen: Generic.Aux[A, ARepr],
  bGen: Generic.Aux[B, BRepr],
  mapper: hlist.Mapper.Aux[P, ARepr, BRepr]
): ProductMapper[A, B, P] =
  new ProductMapper[A, B, P] {
    def apply(a: A): B =
      bGen.from(mapper.apply(aGen.to(a)))
  }
```

Il est intéressant de noter que nous définissons un type `P`
pour notre `Poly`, mais nous ne faisons aucune référence à `P` dans notre code.
La *type class*  `Mapper` utilise la résolution implicite pour trouver nos `Cases`,
le compilateur n'a donc besoin que de connaître le type singleton de `P`
pour retrouver les instances.

Créons une méthode d'extension pour simplifier l'utilisation de `ProductMapper`.
Nous voulons que l'utilisateur n'ait qu'à spécifier le type de `B` au moment de l'appel,
nous utilisons donc une voie détournée pour permettre au compilateur d'inférer le type de `Poly`
à partir du type de la valeur en paramètre :

```tut:book:silent
implicit class ProductMapperOps[A](a: A) {
  class Builder[B] {
    def apply[P <: Poly](poly: P)
        (implicit pm: ProductMapper[A, B, P]): B =
      pm.apply(a)
  }

  def mapTo[B]: Builder[B] = new Builder[B]
}
```
Voici un exemple de l'utilisation de cette méthode :

```tut:book:silent
object conversions extends Poly1 {
  implicit val intCase:  Case.Aux[Int, Boolean]   = at(_ > 0)
  implicit val boolCase: Case.Aux[Boolean, Int]   = at(if(_) 1 else 0)
  implicit val strCase:  Case.Aux[String, String] = at(identity)
}

case class IceCream1(name: String, numCherries: Int, inCone: Boolean)
case class IceCream2(name: String, hasCherries: Boolean, numCones: Int)
```

```tut:book
IceCream1("Sundae", 1, false).mapTo[IceCream2](conversions)
```

La syntaxe de `mapTo` ressemble à un simple appel de méthode,
mais il s'agit en réalité de deux appels :
un appel à `mapTo` pour fixer le paramètre de type `B` et
un appel à `Builder.apply` pour spécifier le `Poly`.
Certaines méthodes d'extension ops de shapeless utilisent la même astuce pour
fournir une syntaxe plus simple à l'utilisateur.
