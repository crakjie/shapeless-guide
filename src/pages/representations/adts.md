## Rappel: types de données algébriques

*Algebraic data types (*ADT*s)*[^adts]
est un nom raffiné qui cache un concept très simple.
C'est un moyen idiomatique de représenter une donnée
en utilisant les « et » et « ou » logiques. Par exemple :


 - une forme est un rectangle **ou** un cercle
 - un rectangle a une largeur **et** une hauteur
 - un cercle a un rayon

[^adts]: Ne doivent pas être confondus avec les « abstract data types »,
qui sont un outil différent en informatique et qui ont
peu d'influence sur notre sujet.

Dans la terminologie des *ADT*s,
les types « et » comme le rectangle et le cercle sont appelés *products (produits)*,
les types « ou » comme le type forme sont appelés *coproducts (coproduits)*.
En Scala, nous représentons typiquement les produits par des case class
et les coproduits par des sealed traits :


```tut:book:silent
sealed trait Shape
final case class Rectangle(width: Double, height: Double) extends Shape
final case class Circle(radius: Double) extends Shape

val rect: Shape = Rectangle(3.0, 4.0)
val circ: Shape = Circle(1.0)
```

Ce qui est beau avec les *ADT*s, c'est qu'ils sont complètement type safe.
Le compilateur a une connaisence complète de l'algèbre[^algebra]
que nous définissons, il peut donc nous aider à écrire des méthodes
correctement typées avec nos types.

[^algebra]: Définition du mot « algèbre » : les symboles que nous définissons, comme les rectangles et les cercles ainsi que les règles de manipulation de ces symboles, celles-ci formulées sous forme de méthodes.

```tut:book:silent
def area(shape: Shape): Double =
  shape match {
    case Rectangle(w, h) => w * h
    case Circle(r)       => math.Pi * r * r
  }
```

```tut:book
area(rect)
area(circ)
```

### Les écritures alternatives

Les sealed traits et les *case classes* sont sans aucun doute
les encodages les plus pratique des *ADT*s en *Scala*.
Mais ce ne sont pas les *seules* possibilités d'encodage.
Par exemple, la bibliotèque standard de *Scala* fournit
des produits génériques appelés `Tuples`
ainsi qu'un coproduit générique : `Either`.
Nous aurions pu choisir d'écrire notre `Shape` comme suit :

```tut:book:silent
type Rectangle2 = (Double, Double)
type Circle2    = Double
type Shape2     = Either[Rectangle2, Circle2]

val rect2: Shape2 = Left((3.0, 4.0))
val circ2: Shape2 = Right(1.0)
```
Bien que cette écriture soit moins lisible que la précédente avec des case class,
elle dispose de certaines des propriétés requises.
Nous pouvons toujours écrire du code type safe avec `Shape2` :

```tut:book:silent
def area2(shape: Shape2): Double =
  shape match {
    case Left((w, h)) => w * h
    case Right(r)     => math.Pi * r * r
  }
```

```tut:book
area2(rect2)
area2(circ2)
```

Il est important de noter que `Shape2` est une écriture plus *générique* que `Shape`[^generic].
Tout code qui fonctionne avec une paire de `Doubles`
fonctionnera avec `Rectangle2` et vice versa.
En tant que développeur *Scala* nous avons tendance à préférer
les types sémantiques comme `Rectangle` et `Circle`
aux types génériques tels que `Rectangle2` et `Circle2`,
précisément à cause de leur nature spécialisée.
Toutefois, dans certain cas, la généralité est préférable.
Par exemple, si l'on sérialise des données sur un disque,
nous ne nous soucions pas des différences entre une paire de `Doubles`
et un `Rectangle2`. On écrit seulement les nombres et rien d'autre.

Shapeless nous donne le meilleur des deux mondes ;
nous pouvons utiliser les types sémantiques par défaut
et passer aux représentations génériques lorsque le besoin
d'interopérabilité se fait sentir (nous y reviendrons).

En revanche, au lieu d'utiliser `Tuples` et `Either`,
shapeless utilise son propre type de données pour
représenter les produits et coproduits génériques.
Nous présenterons ces types dans la section suivante.

[^generic]: nous utilisons « générique » de façon informelle,
au lieu du sens conventionnel : « un type paramétré ».
